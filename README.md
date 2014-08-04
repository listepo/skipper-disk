# [<img title="skipper-disk - Local disk adapter for Skipper" src="http://i.imgur.com/P6gptnI.png" width="200px" alt="skipper emblem - face of a ship's captain"/>](https://github.com/balderdashy/skipper-disk) Disk Blob Adapter

[![NPM version](https://badge.fury.io/js/skipper-disk.png)](http://badge.fury.io/js/skipper-disk) &nbsp; &nbsp;
[![Build Status](https://travis-ci.org/balderdashy/skipper-disk.svg?branch=master)](https://travis-ci.org/balderdashy/skipper-disk)

Local filesystem adapter for receiving streams of file streams. Particularly useful for streaming multipart file uploads via [Skipper](github.com/balderdashy/skipper).


========================================

## Installation

```
$ npm install skipper-disk --save
```

Also make sure you have skipper [installed as your body parser](http://beta.sailsjs.org/#/documentation/concepts/Middleware?q=adding-or-overriding-http-middleware).

> Skipper is installed by defaut in [Sails](https://github.com/balderdashy/sails) v0.10.

========================================

## Usage

> This module is bundled as the default file upload adapter in Skipper, so the following usage is slightly simpler than it is with the other Skipper file upload adapters.

In the route(s) / controller action(s) where you want to accept file uploads, do something like:

```javascript
function (req, res) {
  req.file('avatar')
  .upload(function whenDone(err, uploadedFiles) {
    if (err) return res.negotiate(err);
    else return res.json({
      files: uploadedFiles,
      textParams: req.params.all()
    });
  });
}
```


========================================

## Details


```javascript
function (req, res) {

  req.file('avatar')
  .upload({

    // Specify skipper-disk explicitly (you don't need to do this b/c skipper-disk is the default)
    adapter: require('skipper-disk'),
  
    // You can apply a file upload limit (in bytes)
    maxBytes: 1000000
    
  }, function whenDone(err, uploadedFiles) {
    if (err) return res.negotiate(err);
    else return res.json({
      files: uploadedFiles,
      textParams: req.params.all()
    });
  });
}
```


| Option    | Type       | Details |
|-----------|:----------:|---------|
| `dirname`  | ((string)) | The path to the directory on disk where file uploads should be streamed.  May be specified as an absolute path (e.g. `/Users/mikermcneil/foo`) or a relative path from the current working directory.  Defaults to `".tmp/uploads/"`. `dirname` can be used with `saveAs`- the filename from saveAs will be relative to dirname.
| `saveAs()`  | ((function)) -or- ((string)) | Optional.  By default, Skipper decides an "at-rest" filename for your uploaded files (called the `fd`) by generating a UUID and combining it with the file's original file extension when it was uploaded ("e.g. 24d5f444-38b4-4dc3-b9c3-74cb7fbbc932.jpg"). <br/>  If `saveAs` is specified as a string, any uploaded file(s) will be saved to that particular path instead (useful for simple single-file uploads).<br/> If `saveAs` is specified as a function, that function will be called each time a file is received, passing it the raw stream and a callback. When ready, your `saveAs` function should call the callback, passing the appropriate at-rest filename (`fd`) as the second argument to the callback (and passing an error as the first argument if something went wrong). For example: <br/> `function (__newFileStream,cb) { cb(null, 'theUploadedFile.foo'); }`   |


========================================

## Specifying Options

All options may be passed in using any of the following approaches, in ascending priority order (e.g. the 3rd appraoch overrides the 1st)


##### 1. In the blob adapter's factory method:

```js
var blobAdapter = require('skipper-disk')({
  // These options will be applied unless overridden.
});
```

##### 2. In a call to the `.receive()` factory method:

```js
var receiving = blobAdapter.receive({
  // Options will be applied only to this particular receiver.
});
```

##### 3. Directly into the `.upload()` method of the Upstream returned by `req.file()`:

```js
var upstream = req.file('foo').upload({
  // These options will be applied unless overridden.
});
```

========================================

## Low-Level Usage

> **Warning:**
> You probably shouldn't try doing anything in this section unless you've implemented streams before, and in particular _streams2_ (i.e. "suck", not "spew" streams).



#### File adapter instances, receivers, upstreams, and binary streams

First instantiate a blob adapter (`blobAdapter`):

```js
var blobAdapter = require('skipper-disk')();
```

Build a receiver (`receiving`):

```js
var receiving = blobAdapter.receive();
```

Then you can stream file(s) from a particular field (`req.file('foo')`):

```js
req.file('foo').upload(receiving, function (err, filesUploaded) {
  // ...
});
```

<!--

#### `upstream.pipe(receiving)`

As an alternative to the `upload()` method, you can pipe an incoming **upstream** returned from `req.file()` (a Readable stream of Readable binary streams) directly to the **receiver** (a Writable stream for Upstreams.)

```js
req.file('foo').pipe(receiving);
```

There is no performance benefit to using `.pipe()` instead of `.upload()`-- they both use streams2.  The `.pipe()` method is available merely as a matter of flexibility/chainability.  Be aware that `.upload()` handles the `error` and `finish` events for you; if you choose to use `.pipe()`, you will of course need to listen for these events manually:

```js
req.file('foo')
.on('error', function onError() { ... })
.on('finish', function onSuccess() { ... })
.pipe(receiving)
```

-->

========================================

## Contribute

See `CONTRIBUTING.md`.



========================================

### License

**[MIT](./LICENSE)**
&copy; 2013, 2014-

[Mike McNeil](http://michaelmcneil.com), [Balderdash](http://balderdash.co) & contributors

See `LICENSE.md`.

This module is part of the [Sails framework](http://sailsjs.org), and is free and open-source under the [MIT License](http://sails.mit-license.org/).


![image_squidhome@2x.png](http://i.imgur.com/RIvu9.png)


[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/a22d3919de208c90c898986619efaa85 "githalytics.com")](http://githalytics.com/balderdashy/sails.io.js)
