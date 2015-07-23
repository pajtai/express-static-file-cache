# express-static-file-cache

```
npm install --save express-static-file-cache
```

Middleware to cache your Jade templates as static html. The api gives you the ability to wholesale clear the cache as
needed. As a side effect, also gives you the ability to pod organize your jade templates with the middlewares that show them.

Streaming to return content to the user is faster than compiling the entire response first. Static files are streamed
by default in express, but Jade templates do not support streaming. So, even if you cache all database interactions,
a static html page will be faster than templating that doesn't support streaming.

## Api

`configure` and `clearCache` are available directly from the npm.

### require('express-static-file-cache').configure({ ... })

Configure return a middleware function to use in your app.

#### options

* `app`      : required - the express app being used
* `cacheDir` : required - path to root directory of cache
* `express`  : required - express
* `verbose`  : optional - if truthy will console log some extra info
* `dev`      : options - defaults to false - true will send the response but not cache it to a file

```javascript
var express = require('express'),
    app = express();
    
app.use(require('express-static-file-cache').configure({
    app         : app,
    express     : express,
    cacheDir    : cacheDir
}));
```

Now in your downstream middleware:

```javascript
var path = require('path');

function(req, res, next) {
    res.cache(require.resolve('./view.jade', data);
}
```

### require('express-static-file-cache').clearCache()

Calling this method will delete all static cache files.
Returns a promise that is resolved once all files are cleared.

clearCache is available both directly from the npm and on the response object:

```javascript
var staticCache = require('express-static-file-cache')

db.on('content-changed', function() {
    staticCache.clearCache();
});
```

or

```javascript
function(req, res, next) {
    
    if (timeToClearCache()) {
        res
            .clearCache()
            .then(function() {
                next();
            });
    } else {
        next();
    }
}
```

### middleware

`res.cache` and `res.clearCache` are available as methods on the response object downstream from the addition of the
middleware. 

To add the caching middleware, `use` the method returned after configuration:

#### res.cache(templatePath, templatingData)

`res.cache` takes the path to a jade template and the data to be used. It will send the response and write a static
html file with that repsonse to `path.join(req.url, 'index.html');

```javascript
function(req, res, next) {
    res.cache(require.resolve('./view.jade'), {
        name : 'jane'
    });
```    

#### res.clearCache()

Deletes all static cache fiels and returns a promise that is resolved once done.

## Directory Structure

Since `res.cache` takes a path to the Jade file to be used, you can modularize your jade templates.

For example if you always call `res.cache(require.resolve('./view.jade'), data)`, you can organize your "pages" as
follows:

    pages
        home
            index.js
            view.jade
        notFound
            index.js
            view.jade
            
