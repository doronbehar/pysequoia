# PySequoia

Note: This is a work in progress. The API is **not** stable!

Building:

```bash
set -euxo pipefail
python3 -m venv .env
source .env/bin/activate
pip install maturin
maturin develop
```

Now open the console with `python` and import the library:

```python
from pysequoia import Cert, Context
```

## Available functions

### encrypt

Signs and encrypts a string to one or more recipients:

```python
s = Cert.from_file("signing-key.asc")
r = Cert.from_bytes(open("wiktor.asc", "rb").read())
encrypted = Context.standard().encrypt(s, r, "content to encrypt")
print(f"Encrypted data: {encrypted}")
```

### merge

Merges data from old certificate with new packets:

```python
old = Cert.from_file("wiktor.asc")
new = Cert.from_file("wiktor-fresh.asc")
merged = old.merge(new)
print(f"Merged, updated cert: {merged}")
```

### minimize

Discards expired subkeys and User IDs:

```python
cert = Cert.from_file("wiktor.asc")
minimized = Context.standard().minimize(cert)
print(f"Minimized cert: {minimized}")
```

### generate

Creates new general purpose key with given User ID:

```python
alice = Cert.generate("alice@example.com")
fpr = alice.fingerprint
print(f"Generated cert with fingerprint {fpr}:\n{alice}")
```

Newly generated certificates are usable in both encryption and signing
contexts:

```python
alice = Cert.generate("alice@example.com")
bob = Cert.generate("bob@example.com")

encrypted = Context.standard().encrypt(alice, bob, "content to encrypt")
print(f"Encrypted data: {encrypted}")
```

### WKD

Fetching certificates via Web Key Directory:

```python
from pysequoia import wkd
import asyncio

async def fetch_and_display():
    cert = await wkd("test-wkd@metacode.biz")
    print(f"Cert found via WKD: {cert}")

asyncio.run(fetch_and_display())
```

### Key server

Fetching certificates via HKPS protocol:

```python
from pysequoia import KeyServer
import asyncio

async def fetch_and_display():
    ks = KeyServer("hkps://keys.openpgp.org")
    cert = await ks.get("653909a2f0e37c106f5faf546c8857e0d8e8f074")
    print(f"Cert found via HKPS: {cert}")

asyncio.run(fetch_and_display())
```
