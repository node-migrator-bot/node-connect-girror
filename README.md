# connect-girror

[![Build Status](https://secure.travis-ci.org/eladb/node-connect-girror.png?branch=master)](http://travis-ci.org/eladb/node-connect-girror)

Tiny little connect middleware to mount and auto-deploy apps from a git repository. Pretty cool.

```bash
$ npm install connect-girror
```

The sample below "mounts" the github repository https://github.com/eladb/foo into the `/foo`
route on the connect/express server. When the program is started, [girror](https://github.com/eladb/node-girror) is used to 
checkout a local copy of this repository and [spinner](https://github.com/eladb/node-spinner) is used to spawn `app.js` from 
this repository and keep it alive. HTTP requests into this route are proxied 
using [http-proxy](https://github.com/nodejitsu/node-http-proxy) into the child app.

```js
var connect = require('connect');
var girror = require('connect-girror');

var app = connect.createServer();
app.use('/foo', girror('https://github.com/eladb/foo'));
app.get('/', function(req, res) {
  res.end('Nothing to see here. Try /foo\n');
});
app.listen(5000);
```

A working example is available under `samples` (start with `node server`).

## Redeploy hook

`connect-girror` implements a simple POST endpoint which triggers redeployment when invoked.
This is commonly used to serve git post-receive hook such as [github's](http://help.github.com/post-receive-hooks).

To set up the hook, just pass the `{ hook: '/path/to/secret/endpoint' }` option when calling the middleware:

```js
app.use('/foo', girror('https://github.com/eladb/foo', { hook: '/bhmn489dkjh8m' }));
```

Then, any HTTP POST request sent to `/foo/bhmn489dkjh8m` will trigger a redeployment.

## Post-deploy actions

`connect-girror` supports arbitrary post-deployment handlers via the `build` option.
The default handler will execute `npm install` if `package.json` exists in the root of the
fetched repository (and `npm` is installed).

```js
var exec = require('child_process').exec;
var foo = girror('https://github.com/eladb/foo', { 
  hook: '/bhmn489dkjh8m' 
  build: function(dir, callback) {
    console.log('foo was just updated! horray!');
    return callback(); // first argument can be an error which will fail the deployment
  }
});
```

## API

### `girror(remote, [options])`

`remote` is any legit git URL (e.g. `https://user:password@github.com/account/repo`, `git@github.com:account/repo.git`, 
`file:///repo/file/system/path`). Any credentials must be supplied in case they are not implicitly installed on the machine.

`options.main` is the app's file name. Defaults to `app.js`.

`options.hook` is the relative URL of the deployment hook endpoint. Any HTTP POST requests sent to this hook will cause a re-deployment. Use some form of obscure key for security.

`options.build` is a `function(callback)` which is called after deployment. `callback` is a `function(err)` that should be called when the build process is over. The default is to run `npm install` in the working directory in case there is a `package.json` file and `npm` is installed.

`options.verbose` can be set to `true` for maximum log verbosity. Otherwise, only errors and warnings will be emitted.

## The MIT License

Copyright (c) 2012 Elad Ben-Israel

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
