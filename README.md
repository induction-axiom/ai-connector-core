# ai-connector-core

The shared OAuth core of three self-hosted AI connectors:

- [ai-connector-for-spliit](https://github.com/induction-axiom/ai-connector-for-spliit)
- [ai-connector-for-your-wealthsimple](https://github.com/induction-axiom/ai-connector-for-your-wealthsimple)
- [ai-connector-for-google-health](https://github.com/induction-axiom/ai-connector-for-google-health)

Each connector is one owner's MCP server on Cloud Run. AI apps (Gemini, Claude,
ChatGPT, Muse and others) connect to it with OAuth: they register themselves,
and the owner approves each one after signing in with Google. This package is
the part of that which the connectors share. It is not a framework for building
other servers, and its API may change with them.

## What's in it

- `ai_connector_core.oauth`: the OAuth provider for the MCP Python SDK. Dynamic
  client registration for public clients, PKCE, owner consent, opaque tokens
  stored only as SHA-256 hashes, and refresh token rotation. A refresh token
  reused within 60 seconds counts as a retry and gets a fresh pair; later reuse
  revokes the grant.
- `ai_connector_core.store`: the record store behind it. `FirestoreStore` keeps
  records in the `oauth_records` collection; `MemoryStore` is for tests.
- `ai_connector_core.testing`: `OAuthSuite`, the OAuth tests every connector
  runs against its own app.

## Using it in a connector

Pin a release in `requirements.txt`:

```
ai-connector-core @ https://github.com/induction-axiom/ai-connector-core/archive/refs/tags/v0.1.0.tar.gz
```

Run the shared tests by mixing `OAuthSuite` into a test case:

```python
import unittest

from ai_connector_core.testing import BASE, OWNER, OAuthSuite
from app import create_app
from config import Config


class OAuthTests(OAuthSuite, unittest.TestCase):
    def make_app(self, store):
        config = Config(BASE, "synthetic-project", OWNER["email"], {})
        return config, create_app(config, store, ...)
```

## Releasing

Bump `version` in `pyproject.toml`, commit, tag `vX.Y.Z` and push the tag. Then
update the pinned URL in each connector, run its tests and deploy it.

## License

AGPL-3.0-only, like the connectors.
