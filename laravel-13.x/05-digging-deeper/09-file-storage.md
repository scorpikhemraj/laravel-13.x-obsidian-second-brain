---
title: File Storage
description: Laravel File Storage documentation - filesystem abstraction using Flysystem
url: https://laravel.com/docs/13.x/filesystem
tags: [logic]
---

# File Storage

- [Introduction](#introduction)
- [Configuration](#configuration)
- [Obtaining Disk Instances](#obtaining-disk-instances)
- [Retrieving Files](#retrieving-files)
- [Storing Files](#storing-files)
- [Deleting Files](#deleting-files)
- [Directories](#directories)
- [Testing](#testing)
- [Custom Filesystems](#custom-filesystems)

## Introduction

Laravel provides a powerful filesystem abstraction thanks to the wonderful Flysystem PHP package by Frank de Jonge. The Laravel Flysystem integration provides simple drivers for working with local filesystems, SFTP, and Amazon S3. Even better, it's amazingly simple to switch between these storage options between your local development machine and production server as the API remains the same for each system.

## Configuration

Laravel's filesystem configuration file is located at `config/filesystems.php`. Within this file, you may configure all of your filesystem "disks". Each disk represents a particular storage driver and storage location. Example configurations for each supported driver are included in the configuration file.

The `local` driver interacts with files stored locally on the server running the Laravel application, while the `sftp` storage driver is used for SSH key-based FTP. The `s3` driver is used to write to Amazon's S3 cloud storage service.

You may configure as many disks as you like and may even have multiple disks that use the same driver.

### The Local Driver

When using the `local` driver, all file operations are relative to the `root` directory defined in your `filesystems` configuration file. By default, this value is set to the `storage/app/private` directory:

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', 'Contents');
```

### The Public Disk

The `public` disk included in your application's `filesystems` configuration file is intended for files that are going to be publicly accessible. By default, the `public` disk uses the `local` driver and stores its files in `storage/app/public`.

If your `public` disk uses the `local` driver and you want to make these files accessible from the web, you should create a symbolic link from `storage/app/public` to `public/storage`:

```bash
php artisan storage:link
```

Once a file has been stored and the symbolic link has been created, you can create a URL to the files using the `asset` helper:

```php
echo asset('storage/file.txt');
```

You may configure additional symbolic links in your `filesystems` configuration file. Each of the configured links will be created when you run the `storage:link` command:

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```

The `storage:unlink` command may be used to destroy your configured symbolic links:

```bash
php artisan storage:unlink
```

### Driver Prerequisites

#### S3 Driver Configuration

Before using the S3 driver, you will need to install the Flysystem S3 package via the Composer package manager:

```bash
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

An S3 disk configuration array is located in your `config/filesystems.php` configuration file. Typically, you should configure your S3 information and credentials using environment variables:

```env
AWS_ACCESS_KEY_ID=<your-key-id>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=<your-bucket-name>
AWS_USE_PATH_STYLE_ENDPOINT=false
```

#### FTP Driver Configuration

Before using the FTP driver, you will need to install the Flysystem FTP package:

```bash
composer require league/flysystem-ftp "^3.0"
```

#### SFTP Driver Configuration

Before using the SFTP driver, you will need to install the Flysystem SFTP package:

```bash
composer require league/flysystem-sftp-v3 "^3.0"
```

### Scoped and Read-Only Filesystems

Scoped disks allow you to define a filesystem where all paths are automatically prefixed with a given path prefix. Before creating a scoped filesystem disk, you will need to install an additional Flysystem package:

```bash
composer require league/flysystem-path-prefixing "^3.0"
```

You may create a path scoped instance of any existing filesystem disk:

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

"Read-only" disks allow you to create filesystem disks that do not allow write operations:

```bash
composer require league/flysystem-read-only "^3.0"
```

### Amazon S3 Compatible Filesystems

By default, your application's `filesystems` configuration file contains a disk configuration for the `s3` disk. In addition to using Amazon S3, you may use it to interact with any S3-compatible file storage service such as RustFS, DigitalOcean Spaces, Vultr Object Storage, Cloudflare R2, or Hetzner Cloud Storage.

Typically, after updating the disk's credentials, you only need to update the value of the `endpoint` configuration option:

```php
'endpoint' => env('AWS_ENDPOINT', 'https://rustfs:9000'),
```

## Obtaining Disk Instances

The `Storage` facade may be used to interact with any of your configured disks:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('avatars/1', $content);
```

If your application interacts with multiple disks, you may use the `disk` method:

```php
Storage::disk('s3')->put('avatars/1', $content);
```

### On-Demand Disks

Sometimes you may wish to create a disk at runtime using a given configuration:

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

## Retrieving Files

The `get` method may be used to retrieve the contents of a file:

```php
$contents = Storage::get('file.jpg');
```

If the file you are retrieving contains JSON, you may use the `json` method:

```php
$orders = Storage::json('orders.json');
```

The `exists` method may be used to determine if a file exists on the disk:

```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // ...
}
```

The `missing` method may be used to determine if a file is missing from the disk:

```php
if (Storage::disk('s3')->missing('file.jpg')) {
    // ...
}
```

### Downloading Files

The `download` method may be used to generate a response that forces the user's browser to download the file:

```php
return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);
```

### File URLs

You may use the `url` method to get the URL for a given file:

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```

When using the `local` driver, all files that should be publicly accessible should be placed in the `storage/app/public` directory and you should create a symbolic link.

### Temporary URLs

Using the `temporaryUrl` method, you may create temporary URLs to files stored using the `local` and `s3` drivers:

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::temporaryUrl(
    'file.jpg', now()->plus(minutes: 5)
);
```

#### Enabling Local Temporary URLs

If you started developing your application before support for temporary URLs was introduced to the `local` driver, you may need to enable local temporary URLs:

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app/private'),
    'serve' => true,
    'throw' => false,
],
```

### File Metadata

The `size` method may be used to get the size of a file in bytes:

```php
$size = Storage::size('file.jpg');
```

The `lastModified` method returns the UNIX timestamp of the last time the file was modified:

```php
$time = Storage::lastModified('file.jpg');
```

The MIME type of a given file may be obtained via the `mimeType` method:

```php
$mime = Storage::mimeType('file.jpg');
```

## Storing Files

The `put` method may be used to store file contents on a disk:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```

### Failed Writes

If the `put` method is unable to write the file to disk, `false` will be returned:

```php
if (! Storage::put('file.jpg', $contents)) {
    // The file could not be written to disk...
}
```

### Prepending and Appending To Files

The `prepend` and `append` methods allow you to write to the beginning or end of a file:

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

### Copying and Moving Files

The `copy` method may be used to copy an existing file to a new location on the disk, while the `move` method may be used to rename or move an existing file:

```php
Storage::copy('old/file.jpg', 'new/file.jpg');

Storage::move('old/file.jpg', 'new/file.jpg');
```

### Automatic Streaming

Streaming files to storage offers significantly reduced memory usage:

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// Automatically generate a unique ID for filename...
$path = Storage::putFile('photos', new File('/path/to/photo'));

// Manually specify a filename...
$path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

### File Uploads

In web applications, one of the most common use-cases for storing files is storing user uploaded files:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserAvatarController extends Controller
{
    /**
     * Update the avatar for the user.
     */
    public function update(Request $request): string
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}
```

By default, the `store` method will generate a unique ID to serve as the filename. The file's extension will be determined by examining the file's MIME type.

#### Specifying a File Name

If you do not want a filename to be automatically assigned to your stored file, you may use the `storeAs` method:

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```

#### Specifying a Disk

By default, this uploaded file's `store` method will use your default disk:

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

### File Visibility

In Laravel's Flysystem integration, "visibility" is an abstraction of file permissions across multiple platforms. Files may either be declared `public` or `private`:

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents, 'public');
```

If the file has already been stored, its visibility can be retrieved and set:

```php
$visibility = Storage::getVisibility('file.jpg');

Storage::setVisibility('file.jpg', 'public');
```

## Deleting Files

The `delete` method accepts a single filename or an array of files to delete:

```php
use Illuminate\Support\Facades\Storage;

Storage::delete('file.jpg');

Storage::delete(['file.jpg', 'file2.jpg']);
```

If necessary, you may specify the disk:

```php
Storage::disk('s3')->delete('path/file.jpg');
```

## Directories

### Get All Files Within a Directory

The `files` method returns an array of all files within a given directory:

```php
$files = Storage::files($directory);
```

If you would like to retrieve a list of all files within a given directory including subdirectories, you may use the `allFiles` method:

```php
$files = Storage::allFiles($directory);
```

### Get All Directories Within a Directory

The `directories` method returns an array of all directories within a given directory:

```php
$directories = Storage::directories($directory);
```

If you would like to retrieve a list of all directories including subdirectories:

```php
$directories = Storage::allDirectories($directory);
```

### Create a Directory

The `makeDirectory` method will create the given directory, including any needed subdirectories:

```php
Storage::makeDirectory($directory);
```

### Delete a Directory

The `deleteDirectory` method may be used to remove a directory and all of its files:

```php
Storage::deleteDirectory($directory);
```

## Testing

The `Storage` facade's `fake` method allows you to easily generate a fake disk for testing:

```php
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('albums can be uploaded', function () {
    Storage::fake('photos');

    $response = $this->json('POST', '/photos', [
        UploadedFile::fake()->image('photo1.jpg'),
        UploadedFile::fake()->image('photo2.jpg')
    ]);

    // Assert one or more files were stored...
    Storage::disk('photos')->assertExists('photo1.jpg');
    Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

    // Assert one or more files were not stored...
    Storage::disk('photos')->assertMissing('missing.jpg');
    Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

    // Assert that the number of files in a given directory matches the expected count...
    Storage::disk('photos')->assertCount('/wallpapers', 2);

    // Assert that a given directory is empty...
    Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
});
```

## Custom Filesystems

Laravel's Flysystem integration provides support for several "drivers" out of the box; however, Flysystem is not limited to these and has adapters for many other storage systems.

To get started, you may add a community maintained adapter:

```bash
composer require spatie/flysystem-dropbox
```

Next, you can register the driver within the `boot` method of one of your application's service providers:

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Foundation\Application;
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Storage::extend('dropbox', function (Application $app, array $config) {
            $adapter = new DropboxAdapter(new DropboxClient(
                $config['authorization_token']
            ));

            return new FilesystemAdapter(
                new Filesystem($adapter, $config),
                $adapter,
                $config
            );
        });
    }
}
```

Once you have created and registered the extension, you may use the `dropbox` driver in your `config/filesystems.php` configuration file.