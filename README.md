# Freezing Time: Testing Laravel Temporary Storage URLs

This repository contains the full content of the articles on testing `Storage::temporaryUrl()` in Laravel. These
articles provide practical insights and techniques to test expiring URLs effectively while ensuring your tests remain
isolated and reliable.

## Articles

1. [Testing Temporary URLs in Laravel Storage](https://dev.to/tegos/testing-temporary-urls-in-laravel-storage-20p7)

   - Learn about the challenges of using `Storage::fake` with `temporaryUrl()` and explore solutions like mocking the
     storage driver or using Laravel's built-in mocking capabilities.

2. [Freezing Time: Testing Laravel Temporary Storage URLs](https://dev.to/tegos/freezing-time-testing-laravel-temporary-storage-urls-13n1)

   - Discover how to handle timing issues in tests by freezing time with features like `$this->freezeTime()` and
     `Carbon::setTestNow()`.

Visit the links above for detailed explanations, controller implementations, and test case examples. Whether you're
dealing with mock storage or managing external images, these guides will help you write robust and reliable tests for
your Laravel application.

