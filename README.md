# POS-Release

Where the ProxyPOS Windows till looks for updates, and where its installers live.

This repository is **public and contains no source code**. That is deliberate: the
application repositories are private, and a private repository's release assets cannot
be downloaded without a credential. A credential baked into an installed till would
have to be rotated, and the day it expired every till in the field would stop
updating — the same failure as a dead address, arriving on a timer.

## Layout

```
v1/pos/windows/latest.json        the pointer every installed till reads
v1/pos/windows/test/latest.json   a fixture used by the updater's tests; no till reads it
```

Installers are attached to GitHub **Releases** in this repository, not committed here.

## How a till finds an update

1. It reads `v1/pos/windows/latest.json`.
2. If the `version` there is newer than its own, it asks the owner, downloads the file
   at `url`, checks it against `sha256`, and runs it.

The address in step 1 is the only one compiled into the app. The address in step 2 is
**data**, so the installers can move to any host at any time by editing one field here
— no release, and no window in which a till is looking somewhere that no longer
answers.

## Publishing a release

`installers/build_installer.ps1` in the POS repository writes `installers/latest.json`
from the installer it has just built, so the version, size and hash describe the file
that actually exists rather than one somebody meant to upload.

```
# 1. Upload the installer first.
gh release create v1.2.0 installers/inventory_pos-1.2.0.exe \
  --repo Bridge77tech/POS-Release --title "1.2.0"

# 2. Then point the tills at it.
cp installers/latest.json v1/pos/windows/latest.json
git commit -am "POS 1.2.0" && git push
```

**Order matters.** Step 2 is what tills act on. Doing it first points every till at a
file that is not there yet: they download, fail, and show their owner a failure for
whatever time passes before step 1.

## Moving this pointer somewhere else

The app reads an ordered list of pointer addresses and uses the first that answers
(`lib/core/update/update_endpoints.dart`). Adding a new primary is a one-line change —
but the old entry **must stay in the same release that adds the new one**, and the old
address must keep answering until every till has reported the new build. Until a till
has taken that build it only knows the old address, and nothing can reach it there once
it stops answering.

Which tills have taken it is visible in the super admin, in the **Till version** column
on the shop list.
