---
title: Database: Migrations
description: Learn how to use Laravel migrations for database version control
url: https://laravel.com/docs/13.x/migrations
tags: [data]
---

# Database: Migrations

-   [Introduction](#introduction)
-   [Generating Migrations](#generating-migrations)
    -   [Squashing Migrations](#squashing-migrations)
-   [Migration Structure](#migration-structure)
-   [Running Migrations](#running-migrations)
    -   [Rolling Back Migrations](#rolling-back-migrations)
-   [Tables](#tables)
    -   [Creating Tables](#creating-tables)
    -   [Updating Tables](#updating-tables)
    -   [Renaming / Dropping Tables](#renaming-and-dropping-tables)
-   [Columns](#columns)
    -   [Creating Columns](#creating-columns)
    -   [Available Column Types](#available-column-types)
    -   [Column Modifiers](#column-modifiers)
    -   [Modifying Columns](#modifying-columns)
    -   [Renaming Columns](#renaming-columns)
    -   [Dropping Columns](#dropping-columns)
-   [Indexes](#indexes)
    -   [Creating Indexes](#creating-indexes)
    -   [Renaming Indexes](#renaming-indexes)
    -   [Dropping Indexes](#dropping-indexes)
    -   [Foreign Key Constraints](#foreign-key-constraints)
-   [Events](#events)

## [Introduction](#introduction)

Migrations are like version control for your database, allowing your team to define and share the application's database schema definition. If you have ever had to tell a teammate to manually add a column to their local database schema after pulling in your changes from source control, you've faced the problem that database migrations solve.

The Laravel `Schema` [[03-architecture-concepts/04-facades.md|facade]] provides database agnostic support for creating and manipulating tables across all of Laravel's supported database systems. Typically, migrations will use this facade to create and modify database tables and columns.

## [Generating Migrations](#generating-migrations)

You may use the `make:migration` [[05-digging-deeper/01-artisan-console.md|Artisan command]] to generate a database migration. The new migration will be placed in your `database/migrations` directory. Each migration filename contains a timestamp that allows Laravel to determine the order of the migrations:

```bash
php artisan make:migration create_flights_table
```

Laravel will use the name of the migration to attempt to guess the name of the table and whether or not the migration will be creating a new table. If Laravel is able to determine the table name from the migration name, Laravel will pre-fill the generated migration file with the specified table. Otherwise, you may simply specify the table in the migration file manually.

If you would like to specify a custom path for the generated migration, you may use the `--path` option when executing the `make:migration` command. The given path should be relative to your application's base path.

Migration stubs may be customized using [[05-digging-deeper/01-artisan-console.md#stub-customization|stub publishing]].

### [Squashing Migrations](#squashing-migrations)

As you build your application, you may accumulate more and more migrations over time. This can lead to your `database/migrations` directory becoming bloated with potentially hundreds of migrations. If you would like, you may "squash" your migrations into a single SQL file. To get started, execute the `schema:dump` command:

```bash
php artisan schema:dump

# Dump the current database schema and prune all existing migrations...
php artisan schema:dump --prune
```

When you execute this command, Laravel will write a "schema" file to your application's `database/schema` directory. The schema file's name will correspond to the database connection. Now, when you attempt to migrate your database and no other migrations have been executed, Laravel will first execute the SQL statements in the schema file of the database connection you are using. After executing the schema file's SQL statements, Laravel will execute any remaining migrations that were not part of the schema dump.

If your application's tests use a different database connection than the one you typically use during local development, you should ensure you have dumped a schema file using that database connection so that your tests are able to build your database. You may wish to do this after dumping the database connection you typically use during local development:

```bash
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

You should commit your database schema file to source control so that other new developers on your team may quickly create your application's initial database structure.

Migration squashing is only available for the MariaDB, MySQL, PostgreSQL, and SQLite databases and utilizes the database's command-line client.

## [Migration Structure](#migration-structure)

A migration class contains two methods: `up` and `down`. The `up` method is used to add new tables, columns, or indexes to your database, while the `down` method should reverse the operations performed by the `up` method.

Within both of these methods, you may use the Laravel schema builder to expressively create and modify tables. To learn about all of the methods available on the `Schema` builder, [check out its documentation](#creating-tables). For example, the following migration creates a `flights` table:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

#### [Setting the Migration Connection](#setting-the-migration-connection)

If your migration will be interacting with a database connection other than your application's default database connection, you should set the `$connection` property of your migration:

```php
/**
 * The database connection that should be used by the migration.
 *
 * @var string
 */
protected $connection = 'pgsql';

/**
 * Run the migrations.
 */
public function up(): void
{
    // ...
}
```

#### [Skipping Migrations](#skipping-migrations)

Sometimes a migration might be meant to support a feature that is not yet active and you do not want it to run yet. In this case you may define a `shouldRun` method on the migration. If the `shouldRun` method returns `false`, the migration will be skipped:

```php
use App\Models\Flight;
use Laravel\Pennant\Feature;

/**
 * Determine if this migration should run.
 */
public function shouldRun(): bool
{
    return Feature::active(Flight::class);
}
```

## [Running Migrations](#running-migrations)

To run all of your outstanding migrations, execute the `migrate` Artisan command:

```bash
php artisan migrate
```

If you would like to see which migrations have already run and which are still pending, you may use the `migrate:status` Artisan command:

```bash
php artisan migrate:status
```

If you provide the `--step` option to the `migrate` command, the command will run each migration as its own batch, allowing you to roll back individual migrations later using the `migrate:rollback` command:

```bash
php artisan migrate --step
```

If you would like to see the SQL statements that will be executed by the migrations without actually running them, you may provide the `--pretend` flag to the `migrate` command:

```bash
php artisan migrate --pretend
```

#### [Isolating Migration Execution](#isolating-migration-execution)

If you are deploying your application across multiple servers and running migrations as part of your deployment process, you likely do not want two servers attempting to migrate the database at the same time. To avoid this, you may use the `isolated` option when invoking the `migrate` command.

When the `isolated` option is provided, Laravel will acquire an atomic lock using your application's cache driver before attempting to run your migrations. All other attempts to run the `migrate` command while that lock is held will not execute; however, the command will still exit with a successful exit status code:

```bash
php artisan migrate --isolated
```

To utilize this feature, your application must be using the `memcached`, `redis`, `dynamodb`, `database`, `file`, or `array` cache driver as your application's default cache driver. In addition, all servers must be communicating with the same central cache server.

#### [Forcing Migrations to Run in Production](#forcing-migrations-to-run-in-production)

Some migration operations are destructive, which means they may cause you to lose data. In order to protect you from running these commands against your production database, you will be prompted for confirmation before the commands are executed. To force the commands to run without a prompt, use the `--force` flag:

```bash
php artisan migrate --force
```

### [Rolling Back Migrations](#rolling-back-migrations)

To roll back the latest migration operation, you may use the `rollback` Artisan command. This command rolls back the last "batch" of migrations, which may include multiple migration files:

```bash
php artisan migrate:rollback
```

You may roll back a limited number of migrations by providing the `step` option to the `rollback` command. For example, the following command will roll back the last five migrations:

```bash
php artisan migrate:rollback --step=5
```

You may roll back a specific "batch" of migrations by providing the `batch` option to the `rollback` command, where the `batch` option corresponds to a batch value within your application's `migrations` database table. For example, the following command will roll back all migrations in batch three:

```bash
php artisan migrate:rollback --batch=3
```

If you would like to see the SQL statements that will be executed by the migrations without actually running them, you may provide the `--pretend` flag to the `migrate:rollback` command:

```bash
php artisan migrate:rollback --pretend
```

The `migrate:reset` command will roll back all of your application's migrations:

```bash
php artisan migrate:reset
```

#### [Roll Back and Migrate Using a Single Command](#roll-back-migrate-using-a-single-command)

The `migrate:refresh` command will roll back all of your migrations and then execute the `migrate` command. This command effectively re-creates your entire database:

```bash
php artisan migrate:refresh

# Refresh the database and run all database seeds...
php artisan migrate:refresh --seed
```

You may roll back and re-migrate a limited number of migrations by providing the `step` option to the `refresh` command. For example, the following command will roll back and re-migrate the last five migrations:

```bash
php artisan migrate:refresh --step=5
```

#### [Drop All Tables and Migrate](#drop-all-tables-migrate)

The `migrate:fresh` command will drop all tables from the database and then execute the `migrate` command:

```bash
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

By default, the `migrate:fresh` command only drops tables from the default database connection. However, you may use the `--database` option to specify the database connection that should be migrated. The database connection name should correspond to a connection defined in your application's `database` [[02-getting-started/02-configuration.md|configuration file]]:

```bash
php artisan migrate:fresh --database=admin
```

The `migrate:fresh` command will drop all database tables regardless of their prefix. This command should be used with caution when developing on a database that is shared with other applications.

## [Tables](#tables)

### [Creating Tables](#creating-tables)

To create a new database table, use the `create` method on the `Schema` facade. The `create` method accepts two arguments: the first is the name of the table, while the second is a closure which receives a `Blueprint` object that may be used to define the new table:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email');
    $table->timestamps();
});
```

When creating the table, you may use any of the schema builder's [column methods](#creating-columns) to define the table's columns.

#### [Determining Table / Column Existence](#determining-table-column-existence)

You may determine the existence of a table, column, or index using the `hasTable`, `hasColumn`, and `hasIndex` methods:

```php
if (Schema::hasTable('users')) {
    // The "users" table exists...
}

if (Schema::hasColumn('users', 'email')) {
    // The "users" table exists and has an "email" column...
}

if (Schema::hasIndex('users', ['email'], 'unique')) {
    // The "users" table exists and has a unique index on the "email" column...
}
```

#### [Database Connection and Table Options](#database-connection-table-options)

If you want to perform a schema operation on a database connection that is not your application's default connection, use the `connection` method:

```php
Schema::connection('sqlite')->create('users', function (Blueprint $table) {
    $table->id();
});
```

In addition, a few other properties and methods may be used to define other aspects of the table's creation. The `engine` property may be used to specify the table's storage engine when using MariaDB or MySQL:

```php
Schema::create('users', function (Blueprint $table) {
    $table->engine('InnoDB');

    // ...
});
```

The `charset` and `collation` properties may be used to specify the character set and collation for the created table when using MariaDB or MySQL:

```php
Schema::create('users', function (Blueprint $table) {
    $table->charset('utf8mb4');
    $table->collation('utf8mb4_unicode_ci');

    // ...
});
```

The `temporary` method may be used to indicate that the table should be "temporary". Temporary tables are only visible to the current connection's database session and are dropped automatically when the connection is closed:

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});
```

If you would like to add a "comment" to a database table, you may invoke the `comment` method on the table instance. Table comments are currently only supported by MariaDB, MySQL, and PostgreSQL:

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});
```

### [Updating Tables](#updating-tables)

The `table` method on the `Schema` facade may be used to update existing tables. Like the `create` method, the `table` method accepts two arguments: the name of the table and a closure that receives a `Blueprint` instance you may use to add columns or indexes to the table:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

### [Renaming / Dropping Tables](#renaming-and-dropping-tables)

To rename an existing database table, use the `rename` method:

```php
use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);
```

To drop an existing table, you may use the `drop` or `dropIfExists` methods:

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

#### [Renaming Tables With Foreign Keys](#renaming-tables-with-foreign-keys)

Before renaming a table, you should verify that any foreign key constraints on the table have an explicit name in your migration files instead of letting Laravel assign a convention based name. Otherwise, the foreign key constraint name will refer to the old table name.

## [Columns](#columns)

### [Creating Columns](#creating-columns)

The `table` method on the `Schema` facade may be used to update existing tables. Like the `create` method, the `table` method accepts two arguments: the name of the table and a closure that receives an `Illuminate\Database\Schema\Blueprint` instance you may use to add columns to the table:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

### [Available Column Types](#available-column-types)

The schema builder blueprint offers a variety of methods that correspond to the different types of columns you can add to your database tables. Each of the available methods are listed in the table below:

#### [Boolean Types](#booleans-method-list)

- [boolean](#column-method-boolean)

#### [String & Text Types](#strings-and-texts-method-list)

- [char](#column-method-char)
- [longText](#column-method-longText)
- [mediumText](#column-method-mediumText)
- [string](#column-method-string)
- [text](#column-method-text)
- [tinyText](#column-method-tinyText)

#### [Numeric Types](#numbers--method-list)

- [bigIncrements](#column-method-bigIncrements)
- [bigInteger](#column-method-bigInteger)
- [decimal](#column-method-decimal)
- [double](#column-method-double)
- [float](#column-method-float)
- [id](#column-method-id)
- [increments](#column-method-increments)
- [integer](#column-method-integer)
- [mediumIncrements](#column-method-mediumIncrements)
- [mediumInteger](#column-method-mediumInteger)
- [smallIncrements](#column-method-smallIncrements)
- [smallInteger](#column-method-smallInteger)
- [tinyIncrements](#column-method-tinyIncrements)
- [tinyInteger](#column-method-tinyInteger)
- [unsignedBigInteger](#column-method-unsignedBigInteger)
- [unsignedInteger](#column-method-unsignedInteger)
- [unsignedMediumInteger](#column-method-unsignedMediumInteger)
- [unsignedSmallInteger](#column-method-unsignedSmallInteger)
- [unsignedTinyInteger](#column-method-unsignedTinyInteger)

#### [Date & Time Types](#dates-and-times-method-list)

- [dateTime](#column-method-dateTime)
- [dateTimeTz](#column-method-dateTimeTz)
- [date](#column-method-date)
- [time](#column-method-time)
- [timeTz](#column-method-timeTz)
- [timestamp](#column-method-timestamp)
- [timestamps](#column-method-timestamps)
- [timestampsTz](#column-method-timestampsTz)
- [softDeletes](#column-method-softDeletes)
- [softDeletesTz](#column-method-softDeletesTz)
- [year](#column-method-year)

#### [Binary Types](#binaries-method-list)

- [binary](#column-method-binary)

#### [Object & Json Types](#object-and-jsons-method-list)

- [json](#column-method-json)
- [jsonb](#column-method-jsonb)

#### [UUID & ULID Types](#uuids-and-ulids-method-list)

- [ulid](#column-method-ulid)
- [ulidMorphs](#column-method-ulidMorphs)
- [uuid](#column-method-uuid)
- [uuidMorphs](#column-method-uuidMorphs)
- [nullableUlidMorphs](#column-method-nullableUlidMorphs)
- [nullableUuidMorphs](#column-method-nullableUuidMorphs)

#### [Spatial Types](#spatials-method-list)

- [geography](#column-method-geography)
- [geometry](#column-method-geometry)

#### [Relationship Types](#relationship-method-list)

- [foreignId](#column-method-foreignId)
- [foreignIdFor](#column-method-foreignIdFor)
- [foreignUlid](#column-method-foreignUlid)
- [foreignUuid](#column-method-foreignUuid)
- [morphs](#column-method-morphs)
- [nullableMorphs](#column-method-nullableMorphs)

#### [Specialty Types](#specifics-method-list)

- [enum](#column-method-enum)
- [set](#column-method-set)
- [macAddress](#column-method-macAddress)
- [ipAddress](#column-method-ipAddress)
- [rememberToken](#column-method-rememberToken)
- [vector](#column-method-vector)

#### [`bigIncrements()`](#column-method-bigIncrements)

The `bigIncrements` method creates an auto-incrementing `UNSIGNED BIGINT` (primary key) equivalent column:

```php
$table->bigIncrements('id');
```

#### [`bigInteger()`](#column-method-bigInteger)

The `bigInteger` method creates a `BIGINT` equivalent column:

```php
$table->bigInteger('votes');
```

#### [`binary()`](#column-method-binary)

The `binary` method creates a `BLOB` equivalent column:

```php
$table->binary('photo');
```

When utilizing MySQL, MariaDB, or SQL Server, you may pass `length` and `fixed` arguments to create `VARBINARY` or `BINARY` equivalent column:

```php
$table->binary('data', length: 16); // VARBINARY(16)

$table->binary('data', length: 16, fixed: true); // BINARY(16)
```

#### [`boolean()`](#column-method-boolean)

The `boolean` method creates a `BOOLEAN` equivalent column:

```php
$table->boolean('confirmed');
```

#### [`char()`](#column-method-char)

The `char` method creates a `CHAR` equivalent column with of a given length:

```php
$table->char('name', length: 100);
```

#### [`dateTimeTz()`](#column-method-dateTimeTz)

The `dateTimeTz` method creates a `DATETIME` (with timezone) equivalent column with an optional fractional seconds precision:

```php
$table->dateTimeTz('created_at', precision: 0);
```

#### [`dateTime()`](#column-method-dateTime)

The `dateTime` method creates a `DATETIME` equivalent column with an optional fractional seconds precision:

```php
$table->dateTime('created_at', precision: 0);
```

#### [`date()`](#column-method-date)

The `date` method creates a `DATE` equivalent column:

```php
$table->date('created_at');
```

#### [`decimal()`](#column-method-decimal)

The `decimal` method creates a `DECIMAL` equivalent column with the given precision (total digits) and scale (decimal digits):

```php
$table->decimal('amount', total: 8, places: 2);
```

#### [`double()`](#column-method-double)

The `double` method creates a `DOUBLE` equivalent column:

```php
$table->double('amount');
```

#### [`enum()`](#column-method-enum)

The `enum` method creates a `ENUM` equivalent column with the given valid values:

```php
$table->enum('difficulty', ['easy', 'hard']);
```

Of course, you may use the `Enum::cases()` method instead of manually defining an array of allowed values:

```php
use App\Enums\Difficulty;

$table->enum('difficulty', Difficulty::cases());
```

#### [`float()`](#column-method-float)

The `float` method creates a `FLOAT` equivalent column with the given precision:

```php
$table->float('amount', precision: 53);
```

#### [`foreignId()`](#column-method-foreignId)

The `foreignId` method creates an `UNSIGNED BIGINT` equivalent column:

```php
$table->foreignId('user_id');
```

#### [`foreignIdFor()`](#column-method-foreignIdFor)

The `foreignIdFor` method adds a `{column}_id` equivalent column for a given model class. The column type will be `UNSIGNED BIGINT`, `CHAR(36)`, or `CHAR(26)` depending on the model key type:

```php
$table->foreignIdFor(User::class);
```

#### [`foreignUlid()`](#column-method-foreignUlid)

The `foreignUlid` method creates a `ULID` equivalent column:

```php
$table->foreignUlid('user_id');
```

#### [`foreignUuid()`](#column-method-foreignUuid)

The `foreignUuid` method creates a `UUID` equivalent column:

```php
$table->foreignUuid('user_id');
```

#### [`geography()`](#column-method-geography)

The `geography` method creates a `GEOGRAPHY` equivalent column with the given spatial type and SRID (Spatial Reference System Identifier):

```php
$table->geography('coordinates', subtype: 'point', srid: 4326);
```

Support for spatial types depends on your database driver. Please refer to your database's documentation. If your application is utilizing a PostgreSQL database, you must install the [PostGIS](https://postgis.net) extension before the `geography` method may be used.

#### [`geometry()`](#column-method-geometry)

The `geometry` method creates a `GEOMETRY` equivalent column with the given spatial type and SRID (Spatial Reference System Identifier):

```php
$table->geometry('positions', subtype: 'point', srid: 0);
```

Support for spatial types depends on your database driver. Please refer to your database's documentation. If your application is utilizing a PostgreSQL database, you must install the [PostGIS](https://postgis.net) extension before the `geometry` method may be used.

#### [`id()`](#column-method-id)

The `id` method is an alias of the `bigIncrements` method. By default, the method will create an `id` column; however, you may pass a column name if you would like to assign a different name to the column:

```php
$table->id();
```

#### [`increments()`](#column-method-increments)

The `increments` method creates an auto-incrementing `UNSIGNED INTEGER` equivalent column as a primary key:

```php
$table->increments('id');
```

#### [`integer()`](#column-method-integer)

The `integer` method creates an `INTEGER` equivalent column:

```php
$table->integer('votes');
```

#### [`ipAddress()`](#column-method-ipAddress)

The `ipAddress` method creates a `VARCHAR` equivalent column:

```php
$table->ipAddress('visitor');
```

When using PostgreSQL, an `INET` column will be created.

#### [`json()`](#column-method-json)

The `json` method creates a `JSON` equivalent column:

```php
$table->json('options');
```

When using SQLite, a `TEXT` column will be created.

#### [`jsonb()`](#column-method-jsonb)

The `jsonb` method creates a `JSONB` equivalent column:

```php
$table->jsonb('options');
```

When using SQLite, a `TEXT` column will be created.

#### [`longText()`](#column-method-longText)

The `longText` method creates a `LONGTEXT` equivalent column:

```php
$table->longText('description');
```

When utilizing MySQL or MariaDB, you may apply a `binary` character set to the column in order to create a `LONGBLOB` equivalent column:

```php
$table->longText('data')->charset('binary'); // LONGBLOB
```

#### [`macAddress()`](#column-method-macAddress)

The `macAddress` method creates a column that is intended to hold a MAC address. Some database systems, such as PostgreSQL, have a dedicated column type for this type of data. Other database systems will use a string equivalent column:

```php
$table->macAddress('device');
```

#### [`mediumIncrements()`](#column-method-mediumIncrements)

The `mediumIncrements` method creates an auto-incrementing `UNSIGNED MEDIUMINT` equivalent column as a primary key:

```php
$table->mediumIncrements('id');
```

#### [`mediumInteger()`](#column-method-mediumInteger)

The `mediumInteger` method creates a `MEDIUMINT` equivalent column:

```php
$table->mediumInteger('votes');
```

#### [`mediumText()`](#column-method-mediumText)

The `mediumText` method creates a `MEDIUMTEXT` equivalent column:

```php
$table->mediumText('description');
```

When utilizing MySQL or MariaDB, you may apply a `binary` character set to the column in order to create a `MEDIUMBLOB` equivalent column:

```php
$table->mediumText('data')->charset('binary'); // MEDIUMBLOB
```

#### [`morphs()`](#column-method-morphs)

The `morphs` method is a convenience method that adds a `{column}_id` equivalent column and a `{column}_type` `VARCHAR` equivalent column. The column type for the `{column}_id` will be `UNSIGNED BIGINT`, `CHAR(36)`, or `CHAR(26)` depending on the model key type.

This method is intended to be used when defining the columns necessary for a polymorphic [[08-eloquent-orm/02-eloquent-relationships.md|Eloquent relationship]]. In the following example, `taggable_id` and `taggable_type` columns would be created:

```php
$table->morphs('taggable');
```

#### [`nullableMorphs()`](#column-method-nullableMorphs)

The method is similar to the [morphs](#column-method-morphs) method; however, the columns that are created will be "nullable":

```php
$table->nullableMorphs('taggable');
```

#### [`nullableUlidMorphs()`](#column-method-nullableUlidMorphs)

The method is similar to the [ulidMorphs](#column-method-ulidMorphs) method; however, the columns that are created will be "nullable":

```php
$table->nullableUlidMorphs('taggable');
```

#### [`nullableUuidMorphs()`](#column-method-nullableUuidMorphs)

The method is similar to the [uuidMorphs](#column-method-uuidMorphs) method; however, the columns that are created will be "nullable":

```php
$table->nullableUuidMorphs('taggable');
```

#### [`rememberToken()`](#column-method-rememberToken)

The `rememberToken` method creates a nullable, `VARCHAR(100)` equivalent column that is intended to store the current "remember me" [[06-security/01-authentication.md#remembering-users|authentication token]]:

```php
$table->rememberToken();
```

#### [`set()`](#column-method-set)

The `set` method creates a `SET` equivalent column with the given list of valid values:

```php
$table->set('flavors', ['strawberry', 'vanilla']);
```

#### [`smallIncrements()`](#column-method-smallIncrements)

The `smallIncrements` method creates an auto-incrementing `UNSIGNED SMALLINT` equivalent column as a primary key:

```php
$table->smallIncrements('id');
```

#### [`smallInteger()`](#column-method-smallInteger)

The `smallInteger` method creates a `SMALLINT` equivalent column:

```php
$table->smallInteger('votes');
```

#### [`softDeletesTz()`](#column-method-softDeletesTz)

The `softDeletesTz` method adds a nullable `deleted_at` `TIMESTAMP` (with timezone) equivalent column with an optional fractional seconds precision. This column is intended to store the `deleted_at` timestamp needed for Eloquent's "soft delete" functionality:

```php
$table->softDeletesTz('deleted_at', precision: 0);
```

#### [`softDeletes()`](#column-method-softDeletes)

The `softDeletes` method adds a nullable `deleted_at` `TIMESTAMP` equivalent column with an optional fractional seconds precision. This column is intended to store the `deleted_at` timestamp needed for Eloquent's "soft delete" functionality:

```php
$table->softDeletes('deleted_at', precision: 0);
```

#### [`string()`](#column-method-string)

The `string` method creates a `VARCHAR` equivalent column of the given length:

```php
$table->string('name', length: 100);
```

#### [`text()`](#column-method-text)

The `text` method creates a `TEXT` equivalent column:

```php
$table->text('description');
```

When utilizing MySQL or MariaDB, you may apply a `binary` character set to the column in order to create a `BLOB` equivalent column:

```php
$table->text('data')->charset('binary'); // BLOB
```

#### [`timeTz()`](#column-method-timeTz)

The `timeTz` method creates a `TIME` (with timezone) equivalent column with an optional fractional seconds precision:

```php
$table->timeTz('sunrise', precision: 0);
```

#### [`time()`](#column-method-time)

The `time` method creates a `TIME` equivalent column with an optional fractional seconds precision:

```php
$table->time('sunrise', precision: 0);
```

#### [`timestampTz()`](#column-method-timestampTz)

The `timestampTz` method creates a `TIMESTAMP` (with timezone) equivalent column with an optional fractional seconds precision:

```php
$table->timestampTz('added_at', precision: 0);
```

#### [`timestamp()`](#column-method-timestamp)

The `timestamp` method creates a `TIMESTAMP` equivalent column with an optional fractional seconds precision:

```php
$table->timestamp('added_at', precision: 0);
```

#### [`timestampsTz()`](#column-method-timestampsTz)

The `timestampsTz` method creates `created_at` and `updated_at` `TIMESTAMP` (with timezone) equivalent columns with an optional fractional seconds precision:

```php
$table->timestampsTz(precision: 0);
```

#### [`timestamps()`](#column-method-timestamps)

The `timestamps` method creates `created_at` and `updated_at` `TIMESTAMP` equivalent columns with an optional fractional seconds precision:

```php
$table->timestamps(precision: 0);
```

#### [`tinyIncrements()`](#column-method-tinyIncrements)

The `tinyIncrements` method creates an auto-incrementing `UNSIGNED TINYINT` equivalent column as a primary key:

```php
$table->tinyIncrements('id');
```

#### [`tinyInteger()`](#column-method-tinyInteger)

The `tinyInteger` method creates a `TINYINT` equivalent column:

```php
$table->tinyInteger('votes');
```

#### [`tinyText()`](#column-method-tinyText)

The `tinyText` method creates a `TINYTEXT` equivalent column:

```php
$table->tinyText('notes');
```

When utilizing MySQL or MariaDB, you may apply a `binary` character set to the column in order to create a `TINYBLOB` equivalent column:

```php
$table->tinyText('data')->charset('binary'); // TINYBLOB
```

#### [`unsignedBigInteger()`](#column-method-unsignedBigInteger)

The `unsignedBigInteger` method creates an `UNSIGNED BIGINT` equivalent column:

```php
$table->unsignedBigInteger('votes');
```

#### [`unsignedInteger()`](#column-method-unsignedInteger)

The `unsignedInteger` method creates an `UNSIGNED INTEGER` equivalent column:

```php
$table->unsignedInteger('votes');
```

#### [`unsignedMediumInteger()`](#column-method-unsignedMediumInteger)

The `unsignedMediumInteger` method creates an `UNSIGNED MEDIUMINT` equivalent column:

```php
$table->unsignedMediumInteger('votes');
```

#### [`unsignedSmallInteger()`](#column-method-unsignedSmallInteger)

The `unsignedSmallInteger` method creates an `UNSIGNED SMALLINT` equivalent column:

```php
$table->unsignedSmallInteger('votes');
```

#### [`unsignedTinyInteger()`](#column-method-unsignedTinyInteger)

The `unsignedTinyInteger` method creates an `UNSIGNED TINYINT` equivalent column:

```php
$table->unsignedTinyInteger('votes');
```

#### [`ulidMorphs()`](#column-method-ulidMorphs)

The `ulidMorphs` method is a convenience method that adds a `{column}_id` `CHAR(26)` equivalent column and a `{column}_type` `VARCHAR` equivalent column.

This method is intended to be used when defining the columns necessary for a polymorphic [[08-eloquent-orm/02-eloquent-relationships.md|Eloquent relationship]] that use ULID identifiers. In the following example, `taggable_id` and `taggable_type` columns would be created:

```php
$table->ulidMorphs('taggable');
```

#### [`uuidMorphs()`](#column-method-uuidMorphs)

The `uuidMorphs` method is a convenience method that adds a `{column}_id` `CHAR(36)` equivalent column and a `{column}_type` `VARCHAR` equivalent column.

This method is intended to be used when defining the columns necessary for a [[08-eloquent-orm/02-eloquent-relationships.md#polymorphic-relationships|polymorphic Eloquent relationship]] that use UUID identifiers. In the following example, `taggable_id` and `taggable_type` columns would be created:

```php
$table->uuidMorphs('taggable');
```

#### [`ulid()`](#column-method-ulid)

The `ulid` method creates a `ULID` equivalent column:

```php
$table->ulid('id');
```

#### [`uuid()`](#column-method-uuid)

The `uuid` method creates a `UUID` equivalent column:

```php
$table->uuid('id');
```

#### [`vector()`](#column-method-vector)

The `vector` method creates a `vector` equivalent column:

```php
$table->vector('embedding', dimensions: 100);
```

When utilizing PostgreSQL, the `pgvector` extension must be loaded before `vector` columns can be created:

```php
Schema::ensureVectorExtensionExists();
```

#### [`year()`](#column-method-year)

The `year` method creates a `YEAR` equivalent column:

```php
$table->year('birth_year');
```

### [Column Modifiers](#column-modifiers)

In addition to the column types listed above, there are several column "modifiers" you may use when adding a column to a database table. For example, to make the column "nullable", you may use the `nullable` method:

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

The following table contains all of the available column modifiers. This list does not include [index modifiers](#creating-indexes):

| Modifier | Description |
|----------|-------------|
| `->after('column')` | Place the column "after" another column (MariaDB / MySQL). |
| `->autoIncrement()` | Set `INTEGER` columns as auto-incrementing (primary key). |
| `->charset('utf8mb4')` | Specify a character set for the column (MariaDB / MySQL). |
| `->collation('utf8mb4_unicode_ci')` | Specify a collation for the column. |
| `->comment('my comment')` | Add a comment to a column (MariaDB / MySQL / PostgreSQL). |
| `->default($value)` | Specify a "default" value for the column. |
| `->first()` | Place the column "first" in the table (MariaDB / MySQL). |
| `->from($integer)` | Set the starting value of an auto-incrementing field (MariaDB / MySQL / PostgreSQL). |
| `->instant()` | Add or modify the column using an instant operation (MySQL). |
| `->invisible()` | Make the column "invisible" to `SELECT *` queries (MariaDB / MySQL). |
| `->lock($mode)` | Specify a lock mode for the column operation (MySQL). |
| `->nullable($value = true)` | Allow `NULL` values to be inserted into the column. |
| `->storedAs($expression)` | Create a stored generated column (MariaDB / MySQL / PostgreSQL / SQLite). |
| `->unsigned()` | Set `INTEGER` columns as `UNSIGNED` (MariaDB / MySQL). |
| `->useCurrent()` | Set `TIMESTAMP` columns to use `CURRENT_TIMESTAMP` as default value. |
| `->useCurrentOnUpdate()` | Set `TIMESTAMP` columns to use `CURRENT_TIMESTAMP` when a record is updated (MariaDB / MySQL). |
| `->virtualAs($expression)` | Create a virtual generated column (MariaDB / MySQL / SQLite). |
| `->generatedAs($expression)` | Create an identity column with specified sequence options (PostgreSQL). |
| `->always()` | Defines the precedence of sequence values over input for an identity column (PostgreSQL). |

#### [Default Expressions](#default-expressions)

The `default` modifier accepts a value or an `Illuminate\Database\Query\Expression` instance. Using an `Expression` instance will prevent Laravel from wrapping the value in quotes and allow you to use database specific functions. One situation where this is particularly useful is when you need to assign default values to JSON columns:

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Query\Expression;
use Illuminate\Database\Migrations\Migration;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
            $table->timestamps();
        });
    }
};
```

Support for default expressions depends on your database driver, database version, and the field type. Please refer to your database's documentation.

#### [Column Order](#column-order)

When using the MariaDB or MySQL database, the `after` method may be used to add columns after an existing column in the schema:

```php
$table->after('password', function (Blueprint $table) {
    $table->string('address_line1');
    $table->string('address_line2');
    $table->string('city');
});
```

#### [Instant Column Operations](#instant-column-operations)

When using MySQL, you may chain the `instant` modifier onto a column definition to indicate that the column should be added or modified using MySQL's "instant" algorithm. This algorithm allows certain schema changes to be performed without a full table rebuild, making them nearly instantaneous regardless of table size:

```php
$table->string('name')->nullable()->instant();
```