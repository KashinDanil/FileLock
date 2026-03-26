# FileLock

A lightweight PHP library for file-based exclusive locking using `flock()`.

## Requirements

- PHP 8.1 or higher

## Installation

```bash
composer require danil-kashin/file-lock
```

## Usage

### Basic acquire and release

```php
use DanilKashin\FileLock\FileLock;

$lock = new FileLock('/tmp/my-process.lock');

$lock->acquire(); // blocks until the lock is obtained
// ... critical section ...
$lock->release();
```

### Clean up the lock file after use

```php
$lock = new FileLock('/tmp/my-process.lock');

$lock->acquire();
// ... critical section ...
$lock->release();
$lock->deleteFile();
```

### Error handling

```php
use DanilKashin\FileLock\Exceptions\FileLockException;

$lock = new FileLock('/tmp/my-process.lock');

try {
    $lock->acquire();
    // ... critical section ...
} catch (FileLockException $e) {
    // lock file could not be opened or the lock could not be acquired
    echo $e->getMessage();
} finally {
    $lock->release();
}
```

## API

### `FileLock::__construct(string $lockFile)`

Creates a new `FileLock` instance. The `$lockFile` path is the file used as the lock.
The file is created automatically on the first call to `acquire()` if it does not exist.

### `acquire(): void`

Acquires an exclusive lock (`LOCK_EX`). Blocks until the lock becomes available.
Calling `acquire()` multiple times on the same instance is safe — the file handle is reused and `flock` is re-entrant on the same process.

Throws `FileLockException` if the lock file cannot be opened.

### `release(): void`

Releases the exclusive lock (`LOCK_UN`). The lock file itself is kept on disk.

### `deleteFile(): void`

Deletes the lock file from disk. Call this after `release()` when the file is no longer needed.

## How it works

`FileLock` opens the target file in `cb` mode (create if missing, binary) and delegates all locking to the operating system via PHP's `flock()`. This makes it safe to use across multiple processes on the same machine.

## Running tests

```bash
composer install
vendor/bin/phpunit
```