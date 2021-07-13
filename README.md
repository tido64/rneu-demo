# React Native EU 2021 Demos

- [Manage dependencies with `@rnx-kit/dep-check`](#manage-dependencies-with-rnx-kitdep-check)
- [Use `react-native-test-app` in `react-native-webview`](#use-react-native-test-app-in-react-native-webview)

## Manage dependencies with `@rnx-kit/dep-check`

### Introducing the packages

This repository contains a number of packages, each with an issue of some sort
that we're going to try to highlight and fix with dep-check.

#### `nice-lib`

`nice-lib` is a library that supports React Native 0.63 and 0.64. However, it
has a direct dependency on React 17, even though it should also support 16.13.1
(what React Native 0.63 depends on). This can cause issues because both React 16
and 17 will be installed. Which one gets used is a bit of a mystery.

1. Navigate to `packages/nice-lib`
2. Run `npx rnx-dep-check`
3. Observe that dep-check wants to move `react` from `dependencies` to
   `devDependencies`. It also wants to add `react` to `peerDependencies`,
   ensuring to include the versions required by both React Native 0.63 and 0.64.

#### `another-lib`

`another-lib` is a library that needs to use Webview, but we don't know which
library to use.

1. Navigate to `packages/another-lib`
2. Add `"webview"` to the list of capabilities
   ```diff
   diff --git a/packages/another-lib/package.json b/packages/another-lib/package.json
   index f88e543..f61d309 100644
   --- a/packages/another-lib/package.json
   +++ b/packages/another-lib/package.json
   @@ -30,7 +30,8 @@
          "core-android",
          "core-ios",
          "react",
   -      "test-app"
   +      "test-app",
   +      "webview"
        ]
      }
    }
   ```
3. Run `npx rnx-dep-check`
4. Observe that dep-check wants to add `react-native-webview`

#### `one-app`

`one-app` is an app that consumes both `nice-lib` and `another-lib`. Besides
relying on a slightly outdated React Native version, it is unaware that we
recently added `react-native-webview` to `another-lib`.

1. Navigate to `packages/one-app`
2. Run `npx rnx-dep-check`
3. Observe that dep-check wants to bump `react-native` to 0.64.2. It also picked
   up that `another-lib` depends on `react-native-webview`.

#### `test-app`

`test-app` is an app that also uses the slightly outdated React Native version
that `one-app` used, but it isn't configured for dep-check. We'll show you how
you can benefit from dep-check anyway.

1. Navigate to `packages/test-app`
2. Run `npx rnx-dep-check --vigilant 0.64`
3. Observe that dep-check recommends bumping `react-native` to 0.64.2

### Let's fix all the dependencies

Run `npx rnx-dep-check` from the root of the repository to get a recap of all
the issues.

Now, let's run with `--write` to have dep-check fix all the issues:

```sh
npx rnx-dep-check --write
```

Remember that we had one unconfigured package? We can fix that one as well:

```sh
npx rnx-dep-check --vigilant 0.64 --write
```

## Use `react-native-test-app` in `react-native-webview`

- Repo: <https://github.com/react-native-webview/react-native-webview>
- Prepare steps:
  - We need to bump `react-native` to 0.62.3 beforehand to get fixes for Xcode
    12.5

You can find the full PR with all changes here:
<https://github.com/react-native-webview/react-native-webview/pull/2148>

### Initialize a new test app

Branch: <https://github.com/tido64/react-native-webview/tree/demo/yarn-install>

1. First, we need to move the current example app out of the way
   ```sh
   mv example example-prev
   ```
2. Add `react-native-test-app` as a dev dependency
   ```sh
   yarn add react-native-test-app --dev
   ```
3. Initialize a new test app
   ```sh
   yarn init-test-app
   # ✔ What is the name of your test app? … WebviewExample
   # ✔ Which platforms do you need test apps for? › Android, iOS, macOS, Windows
   # ✔ Where should we create the new project? … example
   ```
4. Observe that the test app has minimal native files
   ```sh
   cd example
   ls ios
   ```
5. Copy code from the previous example app
   ```sh
   rm App.js
   cp -R ../example-prev/App.tsx .
   cp -R ../example-prev/assets .
   cp -R ../example-prev/examples .
   ```
6. Add `react-native-webview` to `package.json`
   ```diff
   diff --git a/example/package.json b/example/package.json
   index 8d7c937..4fba714 100644
   --- a/example/package.json
   +++ b/example/package.json
   @@ -19,7 +19,6 @@
        "react": "16.11.0",
        "react-native": "0.62.3",
        "react-native-macos": "^0.62.0",
   +    "react-native-webview": "../",
        "react-native-windows": "^0.62.0"
      },
      "devDependencies": {
   ```
7. Install npm dependencies
   ```sh
   yarn
   ```
8. In a new terminal, start the dev server
   ```sh
   yarn start
   ```
9. Build the test app
   ```sh
   pod install --project-directory=ios
   yarn ios
   ```

### Bump to react-native 0.63

Branch:
<https://github.com/tido64/react-native-webview/tree/demo/bump-react-native>

Prepare steps:

- Stop the dev server
- Clean up build artifacts
  ```sh
  rm -r ios/Podfile.lock ios/Pods
  ```

1. Bump `react-native` and friends to 0.63
   ```diff
   diff --git a/example/package.json b/example/package.json
   index 8d7c937..ce7aad1 100644
   --- a/example/package.json
   +++ b/example/package.json
   @@ -16,11 +16,11 @@
        "windows": "react-native run-windows --sln windows/WebviewExample.sln"
      },
      "dependencies": {
   -    "react": "16.11.0",
   -    "react-native": "0.62.3",
   -    "react-native-macos": "^0.62.0",
   +    "react": "16.13.1",
   +    "react-native": "0.63.4",
   +    "react-native-macos": "^0.63.0",
        "react-native-webview": "../",
   -    "react-native-windows": "^0.62.0"
   +    "react-native-windows": "^0.63.0"
      },
      "devDependencies": {
        "@babel/core": "^7.6.2",
   @@ -29,10 +29,10 @@
        "babel-jest": "^24.9.0",
        "eslint": "^6.5.1",
        "jest": "^24.9.0",
   -    "metro-react-native-babel-preset": "^0.58.0",
   +    "metro-react-native-babel-preset": "^0.59.0",
        "mkdirp": "^1.0.0",
        "react-native-test-app": "^0.7.0",
   -    "react-test-renderer": "16.11.0"
   +    "react-test-renderer": "16.13.0"
      },
      "jest": {
        "preset": "react-native"
   ```
2. Install npm dependencies
   ```sh
   yarn
   ```
3. In a new terminal, start the dev server
   ```sh
   yarn start
   ```
4. Build the test app
   ```sh
   pod install --project-directory=ios
   yarn ios
   ```
