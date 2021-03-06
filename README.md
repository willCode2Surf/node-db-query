# node-db-query

Create SQL queries programatically in Node.js. Loosely based on Rails' ActiveRelation.

## Getting Started
Install the module with: `npm install db-query`

This example uses [any-db](https://github.com/grncdr/node-any-db), so you'll need to `npm install any-db` to run it.

## Examples
```javascript
var Query = require('db-query');
var anyDB = require('any-db');
var database = 'postgres://user@host:5432/database_name';

var dbPool = anyDB.createPool(database, {
  min: 5,
  max: 15,
  onConnect: function (conn, done) {
    done(null, conn);
  },
  reset: function (conn, done) {
    done(null);
  }
});

var options = { id: [1,2,3,4] };
// or options = { id: '1,2,3,4' };

var q = new Query(dbPool)
  .select('*')
  .from('users')
  .join('JOIN posts ON posts.user_id = users.id')
  .order('posts.updated_at DESC')
  .limit(10);

// Add some sample optional processing to the query
var food = 'cheese';

if (food) {
  // Get people by food
  q.join('JOIN foods ON foods.id = users.food_id')
  q.param(food);
  q.where('foods.name = ' + q.paramNo()); // This becomes : foods.name = $1
} else {
  // Get people by id(s)
  // q.ids is another form of q.param, but will join arrays and wraps the result in '{ }' braces.
  q.ids([1,2,3,4]); // q.ids('1,2,3,4'); as an alternative
  // This is using Postgres' ANY format rather than id IN blah because it's much more efficient
  q.where('id = ANY(' + q.paramNo() + '::int[])');
  // This becomes : id = ANY($1::int[]) 
}

// Prepare some event handlers and execute the query
q.on('row', function(row) { console.log(row); })
  .on('error', function(err) { console.log(err); })
  .on('end', function(data) { console.log(data); })
  .execute();
```

## Contributing
In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests for any new or changed functionality. Lint and test your code using [Grunt](http://gruntjs.com/).

## Release History
_v0.1.0_

## License
Copyright (c) 2013 Mark Selby  
Licensed under the MIT license.
