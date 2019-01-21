# Package Mode Proposal

This is a [Phase 2 proposal](https://github.com/nodejs/modules/blob/master/doc/plan-for-new-modules-implementation.md#phase-2) for the Node.js [ECMAScript modules project](https://github.com/nodejs/ecmascript-modules).

It builds on and is complementary to Phase 2 work currently done with the [Node import file specifier resolution proposal](https://github.com/GeoffreyBooth/node-import-file-specifier-resolution-proposal) and the [package export maps proposal](https://github.com/jkrems/proposal-pkg-exports).

## Problem Statement

To provide a simple mechanism for supporting `.js` ES Modules in Node.js.

## Current Status

### Example of `"exports"` as the Mode Boundary

The [Node import file specifier resolution proposal](https://github.com/GeoffreyBooth/node-import-file-specifier-resolution-proposal) and the [package export maps proposal](https://github.com/jkrems/proposal-pkg-exports) allow `.js` files as ESM whenever the `"exports"` property is used in a package.

`"exports"` therefore acts as a package ESM signifier in the file import specifier proposal:

_package.json_

```json
{}
```

will cause `node file.js` inside the same folder to load `file.js` as CommonJS.

If I then set:

_package.json_

```json
{
  "exports": "./file.js"
}
```

then `node file.js` will now load `file.js` as an ES module, while also locking down the package subpaths.

### `"exports"` Mode Boundary as a Conflation of Concerns

This is a conflation of three separate concerns:

1. A user wants to load `.js` files within a package as ESM.
2. A user wants to set the ESM package entry point.
3. A user wants package export maps or encapsulation.

If a user wants to achieve just one of the above, they are immediately tied into the others.

Instead we should try to break up these cases into the simplest orthogonal primitives so that they are easy to understand and don't unnecessarily bundle up concerns for users.

## Proposal

The proposal, as presented before one year ago to this group (in 5 mins!), is to introduce the `"mode": "esm"` package boundary indicator
to know when `.js` files should be treated as ESM.

_package.json_

```json
{}
```

will load `file.js` in the same folder as CommonJS.

_packge.json_

```json
{
  "mode": "esm"
}
```

will now load `file.js` as an ES module.

Thus a user can now achieve (1), `.js` as ESM, without necessarily opting in to the other features.

### ESM entry point

When it comes to setting the entry point, `"mode": "esm"` is actually fully compatible with the existing `"main"` property in the `package.json`.

So instead of `{ "exports": "./file.js" }` a user can write:

```json
{
  "mode": "esm",
  "main": "file.js"
}
```

to have a `.js` main ES module be supported.

If they wrote `{ "main": "file.mjs" }` then they can still have ESM support fine without setting a mode boundary.

Thus (2), defining the ESM package entry point, is now fully separated as well.

### Compatibility with the Package Exports Proposal

In terms of bringing back the `"exports"` proposal here, this can be done in a compatible way to provide package export maps and encapsulation, and possibly even dual mode package support too.

There are some implementation questions that would need to be worked out further:

* Should `"exports"` automatically act as if `"mode": "esm"` is present?
* If there is both a `"main"` and an `"exports"` property does `"exports"` always win?

## Specification

The specification for this feature is defined here - https://github.com/guybedford/ecmascript-modules/commit/25b493c369cb430b4eac3a69ecdabe7e6cdc41c3.

This is a diff on top of the current import file specifier resolution proposal spec is available at https://github.com/nodejs/ecmascript-modules/pull/19.

All the defined behaviours remain, except for:

1. Delegating the mode to the `"mode": "esm"` signifier.
2. Supporting the package.json `"main"` as the main entry point in ESM packages.

### Draft Implementation

A draft implementation of the approach is available at https://github.com/guybedford/node/tree/irp-mode-implementation.

This does not yet provide the package export maps proposal support as the exact behaviours of this interaction still need to be worked out.
