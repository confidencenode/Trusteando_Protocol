# Trusteando Quickstart

**Get your node on the graph in an afternoon.**

This guide covers Level 1 and Level 2 of the Trusteando Protocol. Level 1 requires no code — just a text editor and a web server. Level 2 adds a 50-line script that enables cryptographic verification.

For the full specification, see the [whitepaper](trusteando_whitepaper_v021_en.md).
For naming and structure conventions, see the [style guide](trusteando_style_guide.md).

---

## Level 1 — Publish a static node (afternoon)

**What you get:** your entity is on the graph. Other nodes can discover you, reference you with `extern/`, and verify your public facts by reading your URLs directly.

**What you need:** a web server you control, a domain you own, a text editor.

### Step 1 — Create the root folder

```
yourdomain.com/trusteando/
```

### Step 2 — Declare your identity

```
yourdomain.com/trusteando/
└── identity/
    ├── [name "Your Organisation Name"]/
    ├── [founded 2010]/
    └── [language en]/
```

### Step 3 — Declare your state

```
yourdomain.com/trusteando/
└── [state verifiado]/
```

### Step 4 — Add your first members (optional)

```
yourdomain.com/trusteando/
└── professors/
    ├── juan-ruiz/
    │   └── since/2021-09-01/
    └── ana-garcia/
        └── since/2023-03-15/
```

### Minimum viable Level 1 node

```
yourdomain.com/trusteando/
├── identity/
│   └── [name "Your Organisation"]/
├── professors/
│   └── juan-ruiz/
│       └── since/2021-09-01/
└── [state verifiado]/
```

That is a valid Trusteando node. No cryptography, no server code, no registration.

---

## Level 2 — Add the reference server (one day)

**What you get:** cryptographic verification. Other nodes can send you a challenge and you prove, without revealing your key, that a given path belongs to you.

### The four functions

```python
import hmac, hashlib

TRUSTEANDO_GRANT_V1     = b"TRUSTEANDO_GRANT_V1\x00"
TRUSTEANDO_CHALLENGE_V1 = b"TRUSTEANDO_CHALLENGE_V1\x00"

class TrusteandoNode:
    def __init__(self, key: bytes):
        self.key = key

    def grant_key(self, child_path_segment: str) -> bytes:
        msg = TRUSTEANDO_GRANT_V1 + child_path_segment.encode('utf-8')
        return hmac.new(self.key, msg, hashlib.sha256).digest()

    def respond_to_challenge(self, context_elements: list[bytes]) -> bytes:
        result = self.key
        for element in context_elements:
            msg = TRUSTEANDO_CHALLENGE_V1 + element
            result = hmac.new(result, msg, hashlib.sha256).digest()
        return result

    def verify_child_authorship(self, child_path_segment: str,
                                 context_elements: list[bytes], proof: bytes) -> bool:
        child_key = self.grant_key(child_path_segment)
        child_node = TrusteandoNode(child_key)
        return child_node.respond_to_challenge(context_elements) == proof
```

### Generating your root key

```python
import secrets
key = secrets.token_bytes(32)
print(key.hex())  # store this securely — this is your identity
```

### Minimal Flask server (50 lines)

```python
from flask import Flask, request, jsonify
import hmac, hashlib, secrets, time, os

TRUSTEANDO_GRANT_V1     = b"TRUSTEANDO_GRANT_V1\x00"
TRUSTEANDO_CHALLENGE_V1 = b"TRUSTEANDO_CHALLENGE_V1\x00"
app = Flask(__name__)

def load_key():
    return bytes.fromhex(os.environ["TRUSTEANDO_KEY"])

class TrusteandoNode:
    def __init__(self, key):
        self.key = key
    def grant_key(self, child):
        return hmac.new(self.key, TRUSTEANDO_GRANT_V1 + child.encode(), hashlib.sha256).digest()
    def respond_to_challenge(self, ctx):
        r = self.key
        for e in ctx:
            r = hmac.new(r, TRUSTEANDO_CHALLENGE_V1 + e, hashlib.sha256).digest()
        return r
    def verify_child_authorship(self, child, ctx, proof):
        return TrusteandoNode(self.grant_key(child)).respond_to_challenge(ctx) == proof

NODE = TrusteandoNode(load_key())

@app.get("/trusteando/_challenge")
def challenge():
    nonce = secrets.token_hex(16)
    return jsonify({"nonce": nonce, "timestamp": int(time.time()), "expires_in": 300})

@app.post("/trusteando/_respond")
def respond():
    d = request.json
    ctx = [d["verifier_id"].encode(), bytes.fromhex(d["content_hash"]), d["nonce"].encode()]
    return jsonify({"proof": NODE.respond_to_challenge(ctx).hex()})

@app.post("/trusteando/_verify")
def verify():
    d = request.json
    ctx = [d["verifier_id"].encode(), bytes.fromhex(d["content_hash"]), d["nonce"].encode()]
    valid = NODE.verify_child_authorship(d["child_path"], ctx, bytes.fromhex(d["proof"]))
    return jsonify({"valid": valid})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
```

Run with:
```bash
export TRUSTEANDO_KEY="your_hex_key_here"
pip install flask
python server.py
```

### Verification flow end to end

```
Verifier              Juan's node          University node
   |                       |                     |
   |-- GET /_challenge ---->|                     |
   |<- { nonce } ----------|                     |
   |-- POST /_respond ----->|                     |
   |<- { proof } ----------|                     |
   |-- POST /_verify (child_path, proof) -------->|
   |<- { valid: true } ---------------------------| 
```

---

## What's next

- **Level 3** — `fields {}` schemas, `on/` event handlers, `steps/` workflows. See whitepaper §2.14.
- **Style guide** — naming conventions, sector schemas. See `trusteando_style_guide.md`.
- **Cookbook** — practical examples for common domains. See `trusteando_cookbook.md`.
- **Implementation details** — conformance levels, test vectors, verifier MUST list. See `trusteando_implementation_guide.md`.

---

*Trusteando Protocol — confidencenode.org/protocolos/trusteando*
*Licensed under GNU GPL v3*
