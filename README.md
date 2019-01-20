# Package Type Proposal

This is a [Phase 2 proposal](https://github.com/nodejs/modules/blob/master/doc/plan-for-new-modules-implementation.md#phase-2) for the Node.js [ECMAScript modules project](https://github.com/nodejs/ecmascript-modules).

It builds on and is complementary to Phase 2 work currently done with the [Node import file specifier resolution proposal](https://github.com/GeoffreyBooth/node-import-file-specifier-resolution-proposal) and the [package export maps proposal](https://github.com/jkrems/proposal-pkg-exports).

## Goals

This proposal aims to define the configuration in `package.json` that will be used to:

- Specify the package type, which for the most part will correspond with its intended method of consumption or target environment: CommonJS, ESM, browser, etc.
	- A package can be multiple types.
	- An ESM package type tells Node to treat the package’s `.js` files as ESM.
- Specify the entry point and/or exported paths for each of a package’s types.

## Motivation

### `"exports"` as the ESM Signifier

The [Node import file specifier resolution proposal](https://github.com/GeoffreyBooth/node-import-file-specifier-resolution-proposal) and the [package export maps proposal](https://github.com/jkrems/proposal-pkg-exports) allow `.js` files as ESM whenever the `"exports"` property is used in a `package.json`. `"exports"` therefore acts as a package ESM signifier in the file import specifier proposal.

A `package.json` of simply `{}` will cause `node file.js` inside the same folder to load `file.js` as CommonJS.

However, if one then sets that `package.json` to be `{ "exports": "./file.js" }`, then `node file.js` will now load `file.js` as an ES module, while also locking down the package subpaths.

Users should be able to trigger `.js`-as-ESM without necessarily needing to also enable `"exports"`’ encapsulation, or needing to type the verbose `{ "exports": { "./": "./" } }`.

## Proposal

### `"type"` Field

We propose the creation of a `package.json` `type` field that takes a string or array of strings, for example:

```json
{
  "type": "esm"
}
```

```json
{
  "type": ["esm", "commonjs", "browser"]
}
```

Node will initially only support the types `"esm"` and `"commonjs"`. Other types that users may specify can be used by build tools or loaders. For example, a loader could be written to enable support for a `"browser"` type by emulating a DOM environment and supplying `window` and other browser globals.

A `package.json` without a `type` field is equivalent to `"type": "commonjs"`. This corresponds with Node’s current behavior for loading packages.

A `package.json` with `"type": "esm"` will cause Node to treat the package as ESM: not only will `.js` files within the package be loaded as ESM, but the CommonJS globals such as `__filename` etc. will not be available.

### Exports

The `"exports"` key from the [package export maps proposal](https://github.com/jkrems/proposal-pkg-exports) behaves as that proposal describes, however a package with multiple types can have objects as values:

```json
{
  "type": ["esm", "commonjs"],
  "exports": {
    "./": {
      "esm": "./src/",
      "commonjs": "./dist/"
    }
  }
}
```

If a path is given only one value, such as `"./": "./"`, it is applied for all of the package’s types.

Note that if a package has only one type, there is no need for the object form as shown here; the simpler `".": "./src/index.js"` is enough.

### Entry points

`"main"` remains reserved exclusively for defining the CommonJS entry point. We cannot change its signature, for example to accept an object of multiple entry points, without breaking backward compatibility.

Therefore, entry points must be defined within `"exports"`:

```json
{
  "type": "esm",
  "exports": "./index.mjs"
}
```
```json
{
  "type": ["typescript", "esm", "commonjs"],
  "exports": {
    ".": {
      "typescript": "./src/index.ts",
      "esm": "./esm/index.js",
      "commonjs": "./cjs/index.js"
    }
  }
}
```

If the user wants to additionally support older versions of Node, `"main"` must also be specified. For newer versions of Node that support both `"exports"` and `"main"`, the entry point specified in `"exports"` shall take precedence.

## Specification

TBD.

### Draft Implementation

TBD.