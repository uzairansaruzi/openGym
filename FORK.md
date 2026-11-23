# Working in this fork

This checkout is Uzair's fork of openGym. `origin` is the fork (`uzairansaruzi/openGym`);
`upstream` is the project (`DuarteSantos8/openGym`). This file and its pointer in CLAUDE.md
live only on `mine`.

## Branches

- `main`: an exact copy of `upstream/main`. Only ever fast-forwarded, never committed to.
- `mine`: `upstream/main` plus Uzair's changes. It is the build that goes on his phone.
- Feature branches (`feat/...`): one change each, branched off `upstream/main`, so they can
  become upstream PRs without dragging the fork-only commits along.

## Where a change goes

1. A change any openGym user would want (a fix, device support): branch off `upstream/main`,
   commit there, then `git switch mine && git merge --ff-only <branch>` (or rebase it onto `mine`).
   Opening the upstream PR is public: ask Uzair first.
2. A change only for this fork: commit it on `mine` with "Fork-only" in the message body.

Fork-only commits on `mine` today: "Personal signing" (team `6GYD9C9N6R`, bundle ID
`com.uzairansar.opengym`, App Group `group.com.uzairansar.opengym` in the Xcode project,
`App.entitlements`, `Info.plist` `BAAppGroupID` and `WorkoutActivity.swift` `OpenGymGroup.id`)
and the commit adding this file. Neither may reach an upstream PR.

## Syncing with upstream

```bash
git fetch upstream
git switch main && git merge --ff-only upstream/main && git push origin main
git switch mine && git rebase upstream/main
```

Done when `git log upstream/main..mine` lists only Uzair's commits. When upstream merges one of
them as a squash, the rebase keeps the originals and may conflict: skip each with
`git rebase --skip`, since upstream's squash already holds it.

## Building onto the iPhone Duo

The `push-to-iphone` skill's script is for Hermex only. For openGym, from `mine`, with the
phone ("Uzair's iPhone Duo") paired to this Mac:

```bash
cd frontend && npm run build:mobile
cd ios/App && xcodebuild -workspace App.xcworkspace -scheme App -configuration Debug \
  -destination 'id=<udid>' -allowProvisioningUpdates -allowProvisioningDeviceRegistration build
xcrun devicectl device install app --device <udid> <DerivedData>/Build/Products/Debug-iphoneos/App.app
xcrun devicectl device process launch --terminate-existing --device <udid> com.uzairansar.opengym
```

`xcrun devicectl list devices` gives the UDID. Done when devicectl prints "App installed" and
"Launched application". It installs beside the App Store openGym, with its own data.

## Gotchas

- `cap sync` (inside `build:mobile`) rewrites `frontend/ios/App/Podfile` and two
  `frontend/android/` gradle files. Restore them with `git checkout` before committing.
  `Podfile.lock` and `App.xcworkspace/contents.xcworkspacedata` stay untracked.
- The iPhone Duo simulator refuses the normal simulator build: the MLKit barcode pod has no
  arm64 simulator slice, so the app comes out x86_64 only. For a simulator test, drop the
  `CapacitorMlkitBarcodeScanning` line from the Podfile, `pod install`, build with
  `-destination 'generic/platform=iOS Simulator' ARCHS=arm64 ONLY_ACTIVE_ARCH=NO
  CODE_SIGNING_ALLOWED=NO`, refresh the page with `VITE_MOBILE=1 npx vite build && node
  ../scripts/catalogue/stage-media.mjs dist && npx cap copy ios`, then restore the Podfile.
- Screenshots of exercise screens carry the licensed media (CLAUDE.md, "Exercise media"): check
  layouts on screens without exercise pictures.
