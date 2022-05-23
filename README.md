# vue-ifdef-loader

## 前言

- 首先感谢您的查阅与使用，该 loader 是基于`Antonino Porcino <nino.porcino@gmail.com>`的`ifdef-loader(v2.3.2)`进行的改造
- 原 loader 仅支持`// #if xxx` 或 `/// #if xxx`的写法，现可支持 vue 单文件中`template`的`<!-- #if xxx -->`这种写法。
- `ifdef-fill-with-blanks`默认值改为`true`，默认行为是删除非当前条件环境下的代码，`false`为使用`/`代替非当前环境的代码

### 安装

`npm i vue-ifdef-loader -D`

### 快速上手

```js
const opts = {
  DEBUG: true,
  version: 3,
  age: 15,
  platform: 'wechat'
  // 以上参数是你像在项目中使用的环境变量，可以任意设置，配合环境变量更好用

  // 是否打印日志
  "ifdef-verbose": true,
  // 为true是三斜杠作为条件编译标识
  // false 是双斜杠作为条件编译标识
  "ifdef-triple-slash": true,
  // add this to uncomment code starting with "// #code "
  "ifdef-uncomment-prefix": "// #code ",
};
module.exports = {
  chainWebpack: config => {
    // ...
    config.module
      .rule("vue")
      .use("ifdef-loader")
      .loader("ifdef-loader")
      .tap(() => {
        return opts;
      })
      .end();
    // ...
    return config;
  },
};
```

### 用法 demo

```vue
<template>
  <div id="app">
    <div id="nav">
      <!-- /// #if age == 16 -->
      我15岁
      <!-- /// #else -->
      我不是15岁
      <!-- /// #endif -->
    </div>
  </div>
</template>

<script>
/// #if age == 15
console.log("年龄等于15");
/// #else
console.log("年龄不等于15");
/// #endif
</script>

<style lang="scss">
/// #if age <= 15
body {
  background: red;
}
/// #else
body {
  background: green;
}
/// #endif
</style>
```

## loader 用法介绍

Webpack loader that allows JavaScript or TypeScript conditional compilation (`#if ... #elif ... #else ... #endif`)
directly from Webpack.

Conditional compilation directives are written inside `///` triple slash comment so
that they don't effect normal JavaScript or TypeScript parsing.

Example:

```js
/// #if DEBUG
console.log("there's a bug!");
/// #endif
```

The `DEBUG` or any other variable can be specified when configuring the Webpack loader (see below).

The directive `#if` accepts any valid JavaScript expression:

```js
/// #if PRODUCTION && version.charAt(0)=='X'
console.log("Ho!");
/// #endif
```

If the expression is `true` the block of code between `#if` and `#endif` is included, otherwise is excluded by commenting it out.

Additionally, `#elif` and `#else` clauses can be added to an `#if` clause:

```js
/// #if env == 'PRODUCTION'
console.log("Production!");
/// #elif env == 'DEBUG'
console.log("Debug!");
/// #else
console.log("Something else!");
/// #endif
```

The `#if` clauses can also be nested:

```js
/// #if PRODUCTION
/// #if OS=="android"
android_code();
/// #elif OS=="ios"
ios_code();
/// #endif
/// #endif
```

## Installation

In webpack build directory:

```
npm install ifdef-loader --save-dev
```

## Configuration

Example of use with TypeScript files, enabling the `DEBUG` and `version` variables:

In `webpack.config.json` put `ifdef-loader` after `ts-loader` so that files are processed
before going into TypeScript compiler:

```js
// define preprocessor variables
const opts = {
   DEBUG: true,
   version: 3,
   "ifdef-verbose": true,                 // add this for verbose output
   "ifdef-triple-slash": false,           // add this to use double slash comment instead of default triple slash
   "ifdef-fill-with-blanks": true,         // add this to remove code with empty spaces instead of "//" comments
   "ifdef-uncomment-prefix": "// #code "  // add this to uncomment code starting with "// #code "
};

/* ... */ {
   test: /\.tsx?$/,
   exclude: /node_modules/,
   use: [
      { loader: "ts-loader" },
      { loader: "ifdef-loader", options: opts }
   ]
}

// alternatively, options can be passed via query string:
const q = require('querystring').encode(opts);
/* ... */ {
   test: /\.tsx?$/,
   exclude: /node_modules/,
   loaders: [ "ts-loader", `ifdef-loader?${q}` ]
}

```

in `example.ts`:

```ts
/// #if DEBUG
/* code to be included if DEBUG is defined */
///   #if version <2
/* code to be included if DEBUG is defined and version < 2*/
///   #endif
/// #endif
```

## Code in comments

Often times writing `#if` ... `#else` ... `#endif` results in code that is not syntactically valid
or does not pass the LINT check. A possible workaround is to hide such code in comments
and let `ifdef-loader` uncomment it if it's part of the block that has to be included in the output.

Example:

The following code is invalid because the linter sees a double declaration of the `a` variable.

```
// #if DEBUG
let a=1;
// #else
let a=2;
// #endif
```

Using code in comments:

```
// #if DEBUG
let a=1;
// #else
// #code let a=2;
// #endif
```

The code is now under comment so it's ignored by the linter; but it's uncommented
by `ifdef-loader` if the else branch has to be included in the output (that is when `DEBUG==false`).

The `// #code ` string prefix can be changed and has to be explictly specified
in the options object:

```
const opts = {
   // ...
   "ifdef-uncomment-prefix": "// #code ",
   // ...
};
```

## License

MIT

## Contributions

Contributions in the form of issues or pull requests are welcome.

## Changes

- v2.3.0 added option `uncomment-prefix` to write code in comments allowing it to pass through linters and syntax checking

- v2.2.0 added option `fill-with-blanks` for removing code with blank spaces instead of `//` comments

- v2.1.0 added support for `#elif` clause.

- v2.0.0 BREAKING CHANGE: options are now passed using the
  standard Webpack API (`loader-utils`). See below for the upgrade.

- v1.0.0 changed to triple slash comment syntax. Double slash syntax
  deprecated and available by turning off the `ifdef-triple-slash` option.

- v1.0.3 fixed bug occurring with short lines. Improved handling of line
  termination (CRLF vs LF) in order to preserve source maps.

- v1.1.0 added support for `#else` clauses.

## Upgrading from v1 to v2

In v2 options are passed differently than v1, so you need to update your `webpack.config.js`.
Just do the following simple changes:

```js
/* from */ const q = require("querystring").encode({
  json: JSON.stringify(opts),
});
/* to   */ const q = require("querystring").encode(opts);
/* you can keep the  ... `ifdef-loader?${q}` ... syntax    */
/* but it's better to pass options directly (see the docs) */
```
