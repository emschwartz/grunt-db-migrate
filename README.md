grunt-db-migrate
================

Migrate database schema and data with [db-migrate](https://github.com/kunklejr/node-db-migrate)

## Getting started

Install with npm:

````
npm i grunt-db-migrate --save-dev
````

You may want to start with simple configuration:

````
   migrate: {
      options: {
        env: {
          DATABASE_URL: databaseUrl
        },
        verbose: true
      }
    }
````

where `databaseUrl` may be exctracted from your environment config like this:

````
var CONF = require('config'),
 databaseUrl = 'postgres://' + CONF.db.user + ':' + CONF.db.password + '@' + CONF.db.host + ':5432/' + CONF.db.name;
````

Create migration with command line command:

````
$ grunt migrate:create:migrate_name
````
It generates a new file based on template in your 'migrations' folder (see configuration section). For example, edit this file, using `async` library:

````
/* global exports, require */

var async = require('async');
exports.up = function (db, callback) {
  async.series([
    db.createTable.bind(db, 'users', {
      id: {
        type: 'int',
        primaryKey: true,
        autoIncrement: true
      },
      email: {
        type: 'string',
        unique: true,
        notNull: true
      },
      password: 'string',
      created_at: 'datetime',
      updated_at: 'datetime'
    }),
    db.createTable.bind(db, 'items', {
      id: {
        type: 'int',
        primaryKey: true,
        autoIncrement: true
      },
      user_id: 'int',
      published: 'boolean',
      title: 'string'
    })
  ], callback);
};

exports.down = function (db, callback) {
  async.series([
    db.dropTable.bind(db, 'users', {
      ifExists: true
    }),
    db.dropTable.bind(db, 'items', {
      ifExists: true
    })
  ], callback);
};
````

And then run the migration step up with `grunt migration` command.

## Targets

`migration:up` - run all available migrations up (or just `migration`). For example, to update database to specified migration run `grunt migration:up 

`migration:down` - run one migration down

`migration:create %name%` - create migration javascript file from template

## Configuration

### `env`

Environment variables for running `db-migrate`.

### `dir`

Default migrations folder is `migrations`. But you can choose another:

````
migrate : {
    options: {
      dir: 'db/schema-migrations'
  }
}
````

### `verbose`

Verbose output migration process.

## CleanDB

````
 grunt.registerTask('cleandb', 'Clean db and re-apply all migrations', function () {
    var fs = require('fs');
    var files = fs.readdirSync('./migrations');
    for (var i = 0; i < files.length; i++) {
      grunt.task.run('migrate:down');
    }
    grunt.task.run('migrate:up');
  });
````