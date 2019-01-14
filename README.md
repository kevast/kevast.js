# kevast.js
[![Build Status](https://img.shields.io/travis/kevast/kevast.js.svg?style=flat-square)](https://travis-ci.org/kevast/kevast.js)
[![Coverage Status](https://img.shields.io/coveralls/github/kevast/kevast.js.svg?style=flat-square)](https://coveralls.io/github/kevast/kevast.js?branch=master)
[![Maintainability](https://img.shields.io/codeclimate/maintainability/kevast/kevast.js.svg?style=flat-square)](https://codeclimate.com/github/kevast/kevast.js)
[![Dependencies](https://img.shields.io/david/kevast/kevast.js.svg?style=flat-square)](https://david-dm.org/kevast/kevast.js)
[![Dev Dependencies](https://img.shields.io/david/dev/kevast/kevast.js.svg?style=flat-square)](https://david-dm.org/kevast/kevast.js?type=dev)
[![Package Version](https://img.shields.io/npm/v/kevast.svg?style=flat-square)](https://www.npmjs.com/package/kevast)
[![Contributions Welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg?style=flat-square)](https://github.com/kevast/kevast.js/issues)
[![Open Issues](https://img.shields.io/github/issues-raw/kevast/kevast.js.svg?style=flat-square)](https://github.com/kevast/kevast.js/issues)
![Node Version](https://img.shields.io/node/v/kevast.svg?style=flat-square)
[![MIT License](https://img.shields.io/npm/l/kevast.svg?style=flat-square)](https://github.com/kevast/kevast.js/blob/master/LICENSE)

![logo](./docs/assets/logo.png)

Kevast is a dependency-free **key-value storage interface**, allowing you to **access key-value based data wherever you want**, memory, file, gist, redis, google drive, etc.

Kevast.js is Javascript version of kevast for both Node.js and browser.

## Installation
### Node.js
Using yarn
```bash
yarn add kevast
```

Using npm
```bash
npm install kevast
```

### Browser
Latest version
```html
<script src="https://cdn.jsdelivr.net/npm/kevast/dist/browser/kevast.min.js"></script>
```
Specific version
```html
<script src="https://cdn.jsdelivr.net/npm/kevast@<version>/dist/browser/kevast.min.js"></script>
```

## Hello Kevast
```javascript
const { Kevast } = require('kevast');
// Install these package with yarn or npm
const { KevastFile } = require('kevast-file');
const { KevastGist } = require('kevast-gist');
const { KevastEncrypt } = require('kevast-encrypt');

const fileStore = new KevastFile('./storage.json');
const gistStore = new KevastGist('YOUR GITHUB ACCESS TOKEN');

// Kevast stores data in all storages
const kevast = new Kevast(fileStore, gistStore);

// Use encryption as a middleware
const password = KevastEncrypt.randomString();
kevast.use(new KevastEncrypt(password));

(async () => {
  // Save key-value data
  await kevast.set('key', Math.random().toString());

  // According to configuration,
  // data will be saved in both file and gist
  // Of course, after encryption

  // Read data from memory
  // and decrypt
  const value = await kevast.get('key');
  console.log(value);
})();
```

## Documentation
### Usage
#### Create an instance
Kevast requires at least one storage to read and store data.
```javascript
const { Kevast } = require('kevast');
const { KevastMemory } = require('kevast-memory');
const { KevastFile } = require('kevast-file');
const { KevastGist } = require('kevast-gist');

const memoryStore = new KevastMemory();
const fileStore = new KevastFile('./storage.json');
const gistStore = new KevastGist('YOUR GITHUB ACCESS TOKEN');

// Error: There should be at least one storage
new Kevast();

const kevast = new Kevast(memoryStore, fileStore, gistStore);
```

#### Basic function
- `.constructor(master: Storage, redundancies?: Storage[])`: Instantiates kevast.
- `.set(key: string, value: string): Promise<void>`: Sets the value for the key.
- `.get(key: string, defaultValue?: string): Promise<string | undefined>`: Returns the value associated to the key, or `defaultValue` or `undefined` if there is none.
- `.remove(key: string): Promise<void>`: Removes a key-value pair
- `.clear(): Promise<void>`: Removes all key-value pairs
- `.use(middleware: DuplexMiddleware): Kevast`: Adds a middleware that works when both `Set` and `Get`.
- `.afterGet.use(middleware: SimplexMiddleware)`: Adds a middleware that works after `Get`.
- `.beforeSet.use(middleware: SimplexMiddleware)`: Adds a middleware that works before `Set`.

#### Fallback getting
While performing `get`, kevast will find the value corresponding to the key from all the stores sequentially.

```javascript
const { Kevast } = require('kevast');
const { KevastMemory } = require('kevast-memory');

(async () => {
  // Currently, both memoryStore1 and memoryStore2 are empty
  const memoryStore1 = new KevastMemory();
  const memoryStore2 = new KevastMemory();

  // Now, memoryStore1 is still empty,
  // but memoryStore2 is { 'key' => 'value' }
  const kevast1 = new Kevast(memoryStore2);
  await kevast1.set('key', 'value');

  const kevast2 = new Kevast(memoryStore1, memoryStore2);
  // Outputs 'value'
  console.log(await kevast2.get('key'));

  // kevast2 initially tried to get value from memoryStore1 but failed,
  // then it got the value from memoryStore2.
})();
```

#### Use a middleware
Kevast has three kinds of middleware:

1. `afterGet` middleware works after `Get` operation.
2. `beforeSet` middleware works before `Set` operation.
3. `DuplexMiddleware` is a combination of the two above.

```javascript
const kevast = await Kevast.create(...storages);
kevast.afterGet.use((pair) => {
  console.log(pair.key, pair.value);
  if (pair.value) {
    pair.value += '***';
  }
});

kevast.beforeSet.use((pair) => {
  console.log(pair.key, pair.value);
  pair.value += '***';
});

kevast.use({
  afterGet(pair) {/* ... */},
  beforeSet(pair) {/* ... */},
});
```

### Development
- [Storage](./docs/storage.md)
- [Middleware](./docs/middleware.md)

## Compatibility
Kevast requires ![Node.js v8.0.0](https://img.shields.io/node/v/kevast.svg?style=flat-square)

Browser support:

|![chrome](https://github.com/alrra/browser-logos/raw/master/src/chrome/chrome_64x64.png)|![firefox](https://github.com/alrra/browser-logos/raw/master/src/firefox/firefox_64x64.png)|![safari](https://github.com/alrra/browser-logos/raw/master/src/safari/safari_64x64.png)|![opera](https://github.com/alrra/browser-logos/raw/master/src/opera/opera_64x64.png)|![edge](https://github.com/alrra/browser-logos/raw/master/src/edge/edge_64x64.png)|![ie11](https://github.com/alrra/browser-logos/raw/master/src/archive/internet-explorer_9-11/internet-explorer_9-11_64x64.png)|![ios](https://github.com/alrra/browser-logos/raw/master/src/safari-ios/safari-ios_64x64.png)|![android](https://github.com/alrra/browser-logos/raw/master/src/archive/android/android_64x64.png)|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|70|63|≤11 12|55|17|9 10 11|≤10 11 12|≤3 4|
|:white_check_mark:|:white_check_mark:|:white_check_mark:|:white_check_mark:|:white_check_mark:|:white_check_mark:|:white_check_mark:|:white_check_mark:|

## Build
```bash
yarn build
```

Both Node.js and browser version will be built. Locate at `kevast.js/dist`.

## Running Tests
```bash
yarn test
```

Both Node.js and browser version will be tested. Note that it will finally open your browser to finish the test.

### Coverage
```bash
yarn coverage
```

## LICENSE
[![MIT License](https://img.shields.io/npm/l/kevast.svg?style=flat-square)](https://github.com/kevast/kevast.js/blob/master/LICENSE)

## Why the funky name?
Kevast stands for ke(y) va(lue) st(orage).
