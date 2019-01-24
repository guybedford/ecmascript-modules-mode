# Package Type Proposal

This is a [Phase 2 proposal](https://github.com/nodejs/modules/blob/master/doc/plan-for-new-modules-implementation.md#phase-2) for the Node.js [ECMAScript modules project](https://github.com/nodejs/ecmascript-modules).

It builds on and is complementary to Phase 2 work currently done with the [Node import file specifier resolution proposal](https://github.com/GeoffreyBooth/node-import-file-specifier-resolution-proposal) and the [package export maps proposal](https://github.com/jkrems/proposal-pkg-exports).

## Problem Statement

To provide a simple mechanism for supporting `.js` ES Modules in Node.js.

## Current Status

### Example of `"exports"` as the ESM Signifier

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

### `"exports"` ESM Signifier as a Conflation of Concerns

This is a conflation of three separate concerns:

1. A user wants to load `.js` files within a package as ESM.
2. A user wants to set the ESM package entry point.
3. A user wants package export maps or encapsulation.

If a user wants to achieve just one of the above, they are immediately tied into the others.

Instead we should try to break up these cases into the simplest orthogonal primitives so that they are easy to understand and don't unnecessarily bundle up concerns for users.

## Proposal

We propose the creation of a `package.json` `type` field that takes a string, to describe the “type” of the package the same way that a file extension describes the _type_ of a file. This `type` field is explicitly _descriptive,_ like the current `package.json` `name` or `version` fields, rather than a place for configuration like a `babel` block.

Adding `"type": "esm"` to `package.json` tells Node to treat `.js` files in this package as ESM.

_package.json_

```json
{}
```

will load `file.js` in the same folder as CommonJS.

_packge.json_

```json
{
  "type": "esm"
}
```

will now load `file.js` as an ES module.

Thus a user can now achieve (1), `.js` as ESM, without necessarily opting in to the other features.

### ESM entry point

When it comes to setting the entry point, `"type": "esm"` is actually fully compatible with the existing `"main"` property in the `package.json`.

So instead of `{ "exports": "./file.js" }` a user can write:

```json
{
  "type": "esm",
  "main": "file.js"
}
```

to have a `.js` main ES module be supported.

If they wrote `{ "main": "file.mjs" }` then they can still have ESM support fine without setting a package type.

Thus (2), defining the ESM package entry point, is now fully separated as well.

### Compatibility with the Package Exports Proposal

In terms of bringing back the `"exports"` proposal here, this can be done in a compatible way to provide package export maps and encapsulation, and possibly even dual-type (CommonJS and ESM) package support too.

There are some implementation questions that would need to be worked out further:

* Should `"exports"` automatically act as if `"type": "esm"` is present?
* If there is both a `"main"` and an `"exports"` property does `"exports"` always win?

### Multiple Types

Currently this proposal envisions packages as only ever being a single type: CommonJS or ESM. If dual- or multiple-type packages are supported by Node in the future, this proposal will need to be amended so as to provide a way for users to specify separate entry points for each type, as `"main"` is already historically defined to only take a single string as its value.

The package exports proposal would also need to be amended accordingly, if the `"exports"` key would not be limited to just ESM. We are deciding to keep the dual-/multiple-type configuration beyond the scope of this proposal for now, to potentially be added at a later date.

## Specification

The specification for this feature is defined at _TBD_.

This is a diff on top of the current import file specifier resolution proposal spec is available at _TBD_.

All the defined behaviours remain, except for:

1. Delegating the type to the `"type": "esm"` signifier.
2. Supporting the `package.json` `"main"` as the main entry point in ESM packages.

### Draft Implementation

A draft implementation of the approach is available at _TBD_.

This does not yet provide the package export maps proposal support as the exact behaviours of this interaction still need to be worked out.
