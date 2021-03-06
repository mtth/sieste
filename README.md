Sieste
======

REST, lazily: transform any `(limit, offset)` resource interface into an
asynchronous iterable.


Installation
------------

```bash
$ npm install sieste
```


API
---

### var iter = sieste([opts], fn)

Returns a new iterable for the given fetching function `fn`.

+ `opts` {Object} Pre-fetching configuration. Two keys are available:

  + `lowWaterMark` (number of cached elements when to send a pre-fetch
    request). [default: `2`]
  + `highWaterMark` (maximum number of elements cached at one time in one
    direction). [default: `5`]

+ `fn(limit, offset, params, cb)` {Function} Function used to load the
  iterable. This function takes the following arguments:

  + `limit` {Number} Number of elements to fetch.
  + `offset` {Number} Starting offset of first element to fetch.
  + `params` {Object} Passed through from `reset`.
  + `cb(err, elems)` {Function} If the list of elements returned is shorter
    than the total amount of elements asked for, the resource will be
    considered exhausted.

Note that `sieste` doesn't make any assumptions on how your underlying resource
is served and will simply take care of calling this method appropriately
(handling pre-fetching and caching for you).

#### iter.reset(params, [index], cb)

Reset the iterable, changing the underlying resource.

+ `params` {Object} Parameters used to set the resource used (this will be
  passed to each call to `fn`).
+ `index` {Number} Optional start index. Note that an error will occur if this
  index is set to a value greater than the total amount of elements in the
  resource. [default: `0`]
+ `cb(err, elem)` {Function} Callback to which the element at `index` will be
  passed.

#### iter.next(cb)

Retrieve next element.

+ `cb(err, elem)` {Function} Callback to which the next element will be
  passed (`null` if end of iterable).


#### iter.prev(cb)

Retrieve previous element.

+ `cb(err, elem)` {Function} Callback to which the previous element will be
  passed (`null` if beginning of iterable).



Quickstart
----------

Sample implementations for a standard REST resource.

### jQuery AJAX

```javascript
// Assuming $ and Sieste available on the global object.

var iter = sieste(function (limit, offset, params, cb) {

  $.ajax({
    url: params.protocol + '//' + params.hostname + '/' + params.pathname,
    data: {limit: limit, offset: offset},
    type: 'GET'
  }).done(function (data) { cb(null, data); })
    .fail(function (xhr) { cb(xhr); });

});
```

### node.js

```javascript
var http = require('http'),
    url = require('url'),
    sieste = require('sieste');

var iter = sieste(function (limit, offset, params, cb) {

  var formattedUrl = url.format({
    protocol: 'http',
    hostname: params.hostname,
    port: params.port,
    pathname: params.pathname,
    query: {limit: limit, offset: offset}
  });
  http.get(formattedUrl, function (res) {
    var data = '';
    var obj;
    res
      .on('data', function (chunk) { data += chunk; })
      .on('end', function () {
        try {
          obj = JSON.parse(data);
        } catch (err) {
          cb(err);
          return;
        }
        cb(null, obj);
      });
  });

});
```

These iterables can then be used in the same way, for example:

```javascript
// Assuming params points to the resource's URL.

iter.reset(params, function (err, elem) {

  console.log('Got elem ' + elem + '!');

});
```
