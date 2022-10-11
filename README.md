# jest-async-storage-mock error

Repository to reproduce the following error: https://github.com/react-native-async-storage/async-storage/issues/849

## Steps to reproduce

1. `yarn install`
2. `yarn test`

Expected error for every test:

```
  ● Test suite failed to run                                                                                                                                                                                                                                                          

    ReferenceError: C:\temp\jest-demo\JestDemo\test\setup.ts: The module factory of `jest.mock()` is not allowed to reference any out-of-scope variables.
    Invalid variable access: async_storage_mock_1
    Allowed objects: AbortController, AbortSignal, AggregateError, Array, ArrayBuffer, Atomics, BigInt, BigInt64Array, BigUint64Array, Boolean, Buffer, DataView, Date, Error, EvalError, Event, EventTarget, FinalizationRegistry, Float32Array, Float64Array, Function, Generator, GeneratorFunction, Infinity, Int16Array, Int32Array, Int8Array, InternalError, Intl, JSON, Map, Math, MessageChannel, MessageEvent, MessagePort, NaN, Number, Object, Promise, Proxy, RangeError, ReferenceError, Reflect, RegExp, Set, SharedArrayBuffer, String, Symbol, SyntaxError, TextDecoder, TextEncoder, TypeError, URIError, URL, URLSearchParams, Uint16Array, Uint32Array, Uint8Array, Uint8ClampedArray, WeakMap, WeakRef, WeakSet, WebAssembly, __dirname, __filename, arguments, atob, btoa, clearImmediate, clearInterval, clearTimeout, console, decodeURI, decodeURIComponent, encodeURI, encodeURIComponent, escape, eval, expect, exports, global, globalThis, isFinite, isNaN, jest, module, parseFloat, parseInt, performance, process, queueMicrotask, require, setImmediate, setInterval, setTimeout, undefined, unescape.
    Note: This is a precaution to guard against uninitialized mock variables. If it is ensured that the mock is required lazily, variable names prefixed with `mock` (case insensitive) are permitted.

      27 | };
      28 | Object.defineProperty(exports, "__esModule", { value: true });
    > 29 | jest.mock("@react-native-async-storage/async-storage", () => async_storage_mock_1.default);
         |                                                              ^^^^^^^^^^^^^^^^^^^^
      30 | jest.mock("i18n-js", () => ({
      31 |     currentLocale: () => "en",
      32 |     t: (key, params) => {

      at File.buildCodeFrameError (node_modules/@babel/core/lib/transformation/file/file.js:249:12)
          at transformFile.next (<anonymous>)
          at run.next (<anonymous>)
          at transform.next (<anonymous>)
```

## Recreate this repository from scratch

1. `npx ignite-cli new JestDemo \
   --bundle=test.jest \
   --git \
   --install-deps \
   --packager=yarn \
   --target-path=C:\temp\jest-demo\JestDemo \
   --remove-demo`
2. Remove `android` and `ios` folders.
3. Pin all dependencies to a fixed version.
4. Run `yarn upgrade-interactive --latest -E` and upgrade all dependencies to the latest version.
5. Add missing `jest-environment-jsdom`: `yarn add --dev jest-environment-jsdom`
6. Fix "private method error" by adding `@babel/plugin-proposal-private-methods` to `babel.config.js`
  ```js
  const plugins = [
    [
        "@babel/plugin-proposal-decorators",
        {
            legacy: true,
        },
    ],
    [
        "@babel/plugin-proposal-private-methods",
        {
            loose: true
        }
    ],
    ["@babel/plugin-proposal-optional-catch-binding"],
    "react-native-reanimated/plugin", // NOTE: this must be last in the plugins
  ]
  
  const vanillaConfig = {
      presets: ["module:metro-react-native-babel-preset"],
      env: {
          production: {},
      },
      plugins,
  }
  
  const expoConfig = {
      presets: ["babel-preset-expo"],
      env: {
          production: {},
      },
      plugins,
  }
  
  let isExpo = false
  try {
      const Constants = require("expo-constants")
      // True if the app is running in an `expo build` app or if it's running in Expo Go.
      isExpo =
          Constants.executionEnvironment === "standalone" ||
          Constants.executionEnvironment === "storeClient"
  } catch {
  }
  
  const babelConfig = isExpo ? expoConfig : vanillaConfig
  
  module.exports = babelConfig
  ```
7. Run `yarn test` and see the tests fail
