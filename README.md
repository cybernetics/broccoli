# Broccoli

[![Build Status](https://travis-ci.org/joliss/broccoli.png?branch=master)](https://travis-ci.org/joliss/broccoli)

A fast, reliable asset pipeline, supporting constant-time rebuilds and compact
build definitions. Comparable to the Rails asset pipeline in scope, though it
runs on Node and is backend-agnostic.

For the command line interface, see
[broccoli-cli](https://github.com/joliss/broccoli-cli).

**This is 0.x beta software.**

Windows is not yet supported.

## Installation

```bash
npm install --save broccoli
npm install --global broccoli-cli
```

## Getting Started

Check out
[broccoli-sample-app](https://github.com/joliss/broccoli-sample-app).

## Brocfile.js

A `Brocfile.js` file in the project root contains the build specification. It
has the following format:

```js
module.exports = function (broccoli) {
};
```

The function may use `broccoli.makeTree(path)` to create trees from paths on
the file system. It may use plugins to create new, generated trees.

The function must return a tree, or an array of trees.

## Plugins

* [broccoli-coffee](https://github.com/joliss/broccoli-coffee)
* [broccoli-template](https://github.com/joliss/broccoli-template)
* [broccoli-static-compiler](https://github.com/joliss/broccoli-static-compiler)
* [broccoli-uglify-js](https://github.com/joliss/broccoli-uglify-js)
* [broccoli-es6-concatenator](https://github.com/joliss/broccoli-es6-concatenator)
* [broccoli-sass](https://github.com/joliss/broccoli-sass) (incomplete)

### Helpers

Shared code for writing plugins.

* [broccoli-filter](https://github.com/joliss/broccoli-filter)
* [broccoli-transform](https://github.com/joliss/broccoli-transform)
* [broccoli-env](https://github.com/joliss/broccoli-env)
* [node-quick-temp](https://github.com/joliss/node-quick-temp)

## Plugin API Specification

Broccoli defines a single plugin API: a tree. A tree object represents a tree
(directory hierarchy) of files that can be regenerated on each build.

By convention, plugins will export a function that takes one or more input
trees, and returns an output tree object.

A tree object must supply two methods that will be called by Broccoli:

### `tree.read(readTree)`

The `.read` method must return a path or a promise for a path, containing the
tree contents.

It receives a `readTree` function argument from Broccoli. If `.read` needs to
read other trees, it must not call `otherTree.read` directly. Instead, it must
call `readTree(otherTree)`, which returns a promise for the path containing
`otherTree`'s contents. It must not call `readTree` again until the promise
has resolved; that is, it cannot call `readTree` on multiple trees in
parallel.

Broccoli will call the `.read` method repeatedly to rebuild the tree, but at
most once per rebuild; that is, if a tree is used multiple times in a build
definition, Broccoli will reuse the path returned instead of calling `.read`
again.

The `.read` method is responsible for creating a new temporary directory to
store the tree contents in. Subsequent invocations of `.read` should remove
temporary directories created in previous invocations.

### `tree.cleanup()`

For every tree whose `.read` method was called one or more times, the
`.cleanup` method will be called exactly once. No further `.read` calls will
follow `.cleanup`. The `.cleanup` method should remove all temporary
directories created by `.read`.

## Security

* Currently Broccoli binds to `0.0.0.0`, exposing your app to the world,
  unless you use a firewall. This is what Rails does, but it still seems like
  bad practice.

* Do not run `broccoli serve` on a production server. While this is
  theoretically safe, it exposes a needlessly large amount of attack surface
  just for serving static assets. Instead, use `broccoli build` to precompile
  your assets, and serve the static files from a web server of your choice.

## License

Broccoli was originally written by [Jo Liss](http://www.solitr.com/) and is
licensed under the [MIT license](LICENSE.md).
