# Adding Hashes to Filenames

Webpack provides placeholders that can be used to access different types of hashes and entry name as we saw before. The most useful ones are:

* `[path]` - Returns entry path.
* `[name]` - Returns entry name.
* `[hash]` - Returns build hash.
* `[chunkhash]` - Returns a chunk specific hash.

Using these placeholders you could end up with filenames, such as:

```bash
app.d587bbd6e38337f5accd.js
vendor.dc746a5db4ed650296e1.js
```

If the file contents are different, the hash will change as well, thus invalidating the cache, or more accurately the browser will send a new request for the new file. This means if only `app` bundle gets updated, only that file needs to be requested again.

An alternative way to achieve the same would be to generate static filenames and invalidate the cache through a querystring (i.e., `app.js?d587bbd6e38337f5accd`). The part behind the question mark will invalidate the cache. This method is not recommended. According to [Steve Souders](http://www.stevesouders.com/blog/2008/08/23/revving-filenames-dont-use-querystring/), attaching the hash to the filename is a more performant way to go.

## Setting Up Hashing

We have already done half of the work needed for a hashing setup to work. In the previous chapter we extracted a manifest to make Webpack build results reliable. We are missing just one thing, hashes. To generate the hashes, we need to tweak the `output` configuration slightly:

**webpack.config.js**

```javascript
...

// Detect how npm is run and branch based on that
switch(process.env.npm_lifecycle_event) {
  case 'build':
    config = merge(
      common,
      {
        devtool: 'source-map',
leanpub-start-insert
        output: {
          path: PATHS.build,
          filename: '[name].[chunkhash].js',
          // This is used for require.ensure. The setup
          // will work without but this is useful to set.
          chunkFilename: '[chunkhash].js'
        }
leanpub-end-insert
      },
      ...
    );
    break;
  default:
    ...
}

module.exports = validate(config);
```

If you execute `npm run build` now, you should see output like this.

```bash
[webpack-validator] Config is valid.
Hash: 5ee4340ea7d9c82ecba8
Version: webpack 1.13.0
Time: 2332ms
                           Asset       Size  Chunks             Chunk Names
     app.61a2f8164146f5d06b9f.js    3.91 kB    0, 2  [emitted]  app
  vendor.0328dbecd1e0c3534145.js    20.7 kB    1, 2  [emitted]  vendor
manifest.6d22dc49ed30205a5468.js  763 bytes       2  [emitted]  manifest
                      index.html  288 bytes          [emitted]
   [0] ./app/index.js 124 bytes {0} [built]
   [0] multi vendor 28 bytes {1} [built]
  [35] ./app/component.js 136 bytes {0} [built]
    + 34 hidden modules
Child html-webpack-plugin for "index.html":
        + 3 hidden modules
```

Our files have neat hashes now. To prove that it works, you could try altering *app/index.js* and include a `console.log` there. After you build, only `app` and `manifest` related bundles should change.

One more way to improve the build further would be to load popular dependencies, such as React, through a CDN. That would decrease the size of the vendor bundle even further while adding an external dependency on the project. The idea is that if the user has hit the CDN earlier, caching can kick in just like here.

## Conclusion

Even though our project has neat caching behavior now, adding hashes to our filenames brings a new problem. If a hash changes, we still have possible older files within our output directory. To eliminate this problem, we can set up a little plugin to clean it up for us.
