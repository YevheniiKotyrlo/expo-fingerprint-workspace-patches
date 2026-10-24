# `@expo/fingerprint` ignores patches applied from a workspace root

A bun workspace whose root `package.json` patches `react-native` through `patchedDependencies`, with a blank Expo app (`npx create-expo-app@latest --template blank`) at `app/`.

## Steps

```sh
bun install
bunx @expo/fingerprint fingerprint:generate app > first.json

# a second revision of the same patch, changing its native hunk
cp revisions/react-native@0.88.0.revision-2.patch patches/react-native@0.88.0.patch
bun install --force   # re-applies the changed patch to node_modules
bunx @expo/fingerprint fingerprint:generate app > second.json
```

Compare the `hash` of `first.json` and `second.json`.

- **Expected:** two different hashes, because the patched native source of react-native changed.
- **Actual:** the same hash. Neither fingerprint has a source for the root `patches/` directory, and react-native is a `package` source keyed by its name and version.

## Control

The fingerprint does read a `patches/` directory inside the app:

```sh
mkdir app/patches
cp patches/react-native@0.88.0.patch app/patches/
bunx @expo/fingerprint fingerprint:generate app > third.json
```

`third.json` has a different hash and a `patchPackage` source for `patches`.
