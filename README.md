# vite

The goal of this project is to provide and easier and faster way
to use Selenium for browser based testing of components.  In 
particular it allows us to test certain things that enzyme can't:
- components which rely on browser behavior, e.g. keyboard navigation
- image regression testing

The system allows you to write tests in a similar what to how you'd
write an enzyme test, but instead of using `jsdom` or shallow rendering
it uses `Selenium` to render components in a real browser.  The
benefit of this is that when you simulate keyboard events, the browser
interprets them as real events which triggers other events normally, e.g.
pressing `TAB` normally causes the focus to be updated which triggers
various `blur` and `focus` events.  This doesn't occur when simulating
a `TAB` keypress event with JavaScript which is what `enzyme` does.

## Quick Start

- yarn
- cd packages/babel-plugin-vite
- yarn build
- cd ../jest-environment-vite
- yarn build
- cd ../vite-demo
- yarn test

## Configuration

Example config:
```
  "jest": {
    "testRegex": "/test/.+\\.js$",
    "testEnvironment": "@khanacademy/jest-environment-vite",
    "testEnvironmentOptions": {
      "capabilities": {
        "browserName": "chrome",
        "chromeOptions": {
          "args": [
            "headless",
            "disable-gpu"
          ]
        }
      },
      "collectCoverage": true
    },
    "globalSetup": "@khanacademy/jest-environment-vite/dist/global-setup.js",
    "globalTeardown": "@khanacademy/jest-environment-vite/dist/global-teardown.js",
    "setupTestFrameworkScriptFile": "@khanacademy/jest-environment-vite/dist/setup.js",
    "verbose": false,
    "moduleNameMapper": {
      "^@khanacademy/(.*)$": "<rootDir>/node_modules/@khanacademy/$1/dist/index.js"
    }
  }
```

You'lla

**Notes**:
- `collectCoverage` must be specified within `testEnvironmentOptions`.  This
  is because the way `jest-environment-vite` collects coverage is incompatible
  with the default jest test runner.
- `testEnvironmentOptions` are passed through to `jest-environment-selenium`
  with the `capabilities` section being passed through to `webdriver.Builder().withCapabilities()`.

## Architecture

The system is comprised of two main parts:
- a server that loads npm modules as well as source .js files
- a babel plugin that rewrites the first argument to `render`, passing
  it an object containg the original source of the first argument as 
  well as the necessary imports to use it.  See `vite-demo/util/util.js`
  for more details.

The server allows us to load any source file and its dependencies in
the browser.  This avoids having to precompile large bundles ahead of
with all of the tests and dependencies.  It also means that we don't 
have to create test pages for each component we want to test using
`Selenium`.  Instead, we can construct scripts at runtime to render
the components we want.

The server is able to serve npm modules by wrapping commonjs modules in
an ES6 module wrapper.  Since the browser can `import("react")` all 
modules are rewritten as paths, e.g. `"/node_modules/react.js"`.  The
file extension is necessary b/c Chrome won't load modules using dynamic
import statements without it.

## TODO

- [x] automate path rewriting within tests
- [x] coverage
- [ ] run tests in different browsers
- [x] extract utils from vite-demo into jest-environment-vite
- [x] add config options to jest-environment-vite to toggle debug output from vite-server
- [ ] run tests inside docker
- [ ] add ability to take screenshots
- [ ] comparison new screenshots against existing snapshots
- [x] replace the original `vite`
- [x] cache compiled npm modules
