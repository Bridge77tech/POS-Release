# Test fixture

`latest.json` here is read by the POS updater's verification run against real GitHub
infrastructure — DNS, TLS, the CDN, redirects to the release asset store and the byte
stream itself — rather than against a loopback socket.

No installed till ever reads this file. Tills read `../latest.json`.

The asset it names is attached to the `test-fixture-v0` release: a small text file,
not an installer. It exists so the fetch-verify-hash path can be proven end to end
without pretending a placeholder is a build.
