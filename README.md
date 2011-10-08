# It's the Facebook Graph, Bro!

[FBgraph](http://criso.github.com/fbgraph/) is a nodejs module that provides easy access to the facebook graph api


## Oh nooooooesss - MOAR facebook

  I created this because I wanted to access FB's graph from `node`.  
  The libraries I found, felt clunky to me, and I needed an excuse to create a node module.

## Installation via npm
    $ npm install fbgraph

    var graph = require('fbgraph');

## Authentication

If you get an accesstoken via some other Oauth module like [everyauth](https://github.com/bnoguchi/everyauth) , 
[connect-auth](https://github.com/ciaranj/connect-auth) or [node-oauth](https://github.com/ciaranj/node-oauth) you can just set  
the access token directly. Most `get` calls, and pretty much all `post` calls will require an `access_token`

```js
    graph.setAccessToken(access_token);
```

This is how you would get authenticated using only the `fbgraph` module.
More details below on the __express app__ section

```js
    // get authorization url
    var authUrl = graph.getOauthUrl({
      "client_id":       conf.client_id
      , "redirect_uri":  conf.redirect_uri
    });

    // shows dialog
    res.redirect(authUrl);

    // after user click, auth `code` will be set
    // we'll send that and get the access token
    graph.authorize({
        "client_id":        conf.client_id
      , "redirect_uri":   conf.redirect_uri
      , "client_secret":  conf.client_secret
      , "code":           req.query.code
    }, function (err, facebookRes) {
      res.redirect('/loggedIn');
    });
```



## Read data from the Graph Api

```js
graph.get("zuck", function(err, res) {
  console.log(res); // { id: '4', name: 'Mark Zuckerberg'... }
});
```

params in the `url`

```js
graph.get("zuck?fields=picture", function(err, res) {
  console.log(res); // { picture: 'http://profile.ak.fbcdn.net/'... }
});
```

params as an `object`

```js
var params = { fields: picture };

graph.get("zuck", params,  function(err, res) {
  console.log(res); // { picture: "http://profile.ak.fbcdn.net/..." }
});
```

GraphApi calls that redirect directly to an image
will return a json response with relevant fields

```js
graph.get("/zuck/picture", function(err, res) {
  console.log(res); // { image: true, location: "http://profile.ak.fb..." }
});
```

## Search data from the Graph Api

Search for public posts that contain __brogramming__

```js
var searchOptions = {
    q: "brogramming"
  , type: "post"
};

graph.search(searchOptions, function(err, res) {
  console.log(res); // {data: [{id: xxx, from: ...}, {id: xxx, from: ...}]}
});
```

## Publish data to the Graph Api
All publish requests will require an `access token`

only needs to be set once

```js
graph.setAccessToken(accessToken);
```

Post a message on a `friend's` wall

```js
var wallPost = {
  message: "I'm gonna come at you like a spider monkey, chip!"
};

graph.post(userId + "/feed", wallPost, function(err, res) {
  // returns the post id
  console.log(res); // { id: xxxxx}
});
```

## Delete a Graph object

To delete a graph object, provide an `object id` and the 
response will return `true` or `false`

```js
graph.del(postID, function(err, res) {
  console.log(res); // true/false
});
```

## Rockin' it on an Express App

This example assumes that you have a link on the main page `/` that points to `/auth/facebook`.   
The user will click this link and get into the facebook authorization flow ( if the user hasn't already connected)  
After `authorizing` the app the user will be redirected to `/UserHasLoggedIn`  

```js
/**
 * Module dependencies.
 */

var express   = require('express')
  , graph     = require('fbgraph')
  , app       = module.exports = express.createServer();

// this should really be in a config file!
var conf = {
    client_id:      'YOUR FACEBOOK APP ID'
  , client_secret:  'YOU FACEBOOK APP SECRET'
  , scope:          'email, user_about_me, user_birthday, user_location, publish_stream'
  , redirect_uri:   'http://localhost:3000/auth/facebook'
};

// Configuration

app.configure(function(){
  app.set('views', __dirname + '/views');
  app.set('view engine', 'jade');
  app.use(express.bodyParser());
  app.use(express.methodOverride());
  app.use(app.router);
  app.use(express.static(__dirname + '/public'));
});

app.configure('development', function(){
  app.use(express.errorHandler({ dumpExceptions: true, showStack: true })); 
});

app.configure('production', function(){
  app.use(express.errorHandler());
});

// Routes

app.get('/', function(req, res){
  res.render("index", { title: "click link to connect" });
});

app.get('/auth/facebook', function(req, res) {

  // we don't have a code yet
  // so we'll redirect to the oauth dialog
  if (!req.query.code) {
    var authUrl = graph.getOauthUrl({
        "client_id":     conf.client_id
      , "redirect_uri":  conf.redirect_uri
    });

    res.redirect(authUrl);

    return;
  }

  // code is set
  // we'll send that and get the access token
  graph.authorize({
      "client_id":      conf.client_id
    , "redirect_uri":   conf.redirect_uri
    , "client_secret":  conf.client_secret
    , "code":           req.query.code
  }, function (err, facebookRes) {
    res.redirect('/loggedIn');
  });

});


// user gets sent here after being authorized
app.get('/UserHasLoggedIn', function(req, res) {
  res.render("index", { title: "Logged In" });
});


var port = process.env.PORT || 3000;
app.listen(port, function() {
  console.log("Express server listening on port %d", port);
});

```

## Running tests

 Before running the test suite, add your Facebook `appId` and `appSecret` to `tests/config.js`   
 This is needed to create `test users` and to get a test `access_token`

    $ npm install
    $ make test

 _Tests might fail if the Facebook api has an issue._

## License

(The MIT License)

Copyright (c) 2011 Cristiano Oliveira &lt;ocean.cris@gmail.com&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

