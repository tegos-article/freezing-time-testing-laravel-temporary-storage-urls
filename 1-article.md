
In a previous article, [Testing Temporary URLs in Laravel Storage](https://dev.to/tegos/testing-temporary-urls-in-laravel-storage-20p7), we explored two techniques to test Laravel's `Storage::temporaryUrl()` method. While `Storage::fake` doesn't natively support `temporaryUrl`, we demonstrated how to use mocking to simulate the behavior effectively. If you haven’t read it yet, consider starting there to understand the foundation of testing temporary URLs in Laravel.

In this continuation, we'll delve into how freezing time can make testing temporary URLs more reliable, especially for time-sensitive functionality. We'll leverage Laravel's built-in test helpers and Carbon's time manipulation features to solve potential inconsistencies in tests.

### Why Freezing Time Matters

Temporary URLs often include expiration timestamps, making them time-sensitive. In a test environment, a small delay in execution can cause mismatched expiration times, leading to test failures. For example:

```
Failed asserting that two strings are equal.
Expected :'http://localhost/test/image?expiration=1737729799'
Actual   :'http://localhost/test/image?expiration=1737729800'
```

This happens when the expiration timestamp generated during test execution slightly differs due to the passage of time. Freezing time ensures that all time-related operations return consistent values, eliminating such discrepancies.

### Laravel's Time Freezing Helpers

Laravel offers several methods for freezing and manipulating time in tests:

- **`$this->freezeTime()`**: Freezes time to the current moment. Any subsequent calls to time-based methods will use this frozen time.
- **`$this->travelTo(Carbon::now())`**: Simulates moving to a specific point in time.
- **`Carbon::setTestNow(Carbon::now())`**: Directly sets the current time for all Carbon operations.

These methods allow you to control the flow of time and ensure consistency across your tests.

For more details, refer to these resources:
- [Freezing Time in Laravel Tests](https://laraveldaily.com/tip/freezing-time-in-laravel-tests)
- [Freezing Time in Tests](https://laravel-code.tips/you-can-freeze-time-in-tests/)


### Practical Example: External Image Fetching with Temporary URLs

#### Controller
Here’s a controller method that fetches an image from an external source, stores it in the local storage, and generates a temporary URL for redirection:

```php
final class ExternalImageTMController extends Controller
{
    /**
     * @throws ItemNotFoundException
     */
    public function show(string $path): RedirectResponse
    {
        $path = Str::replace('-', '/', trim($path));

        if (!Storage::disk(StorageDiskName::DO_S3_TEHNOMIR_PRODUCT_IMAGE->value)->exists($path)) {
            $externalUrl = implode('/', [config('services.tehnomir.s3_image_url'), $path]);
            $response = Http::get($externalUrl);

            if (!$response->successful()) {
                throw new ItemNotFoundException('External image', [$path]);
            }

            Storage::disk(StorageDiskName::DO_S3_TEHNOMIR_PRODUCT_IMAGE->value)->put($path, $response->body());
        }

        return Redirect::away(
            Storage::disk(StorageDiskName::DO_S3_TEHNOMIR_PRODUCT_IMAGE->value)->temporaryUrl($path, Carbon::now()->addHour())
        );
    }
}
```

This method ensures the image is fetched and stored locally if it doesn’t already exist, then redirects the user to a temporary URL for the image.


### Testing the Controller

Here’s how you can test the above functionality with time freezing:

```php
public function test_image_is_fetched_externally_stored_and_redirected(): void
{
    // Arrange
    $user = $this->getDefaultUser();
    $this->actingAsFrontendUser($user);

    $this->freezeTime();

    $path = 'test/image';
    $disk = Storage::fake(StorageDiskName::DO_S3_TEHNOMIR_PRODUCT_IMAGE->value);
    $externalUrl = config('services.tehnomir.s3_image_url') . '/' . $path;

    Http::fake([
        $externalUrl => Http::response('external-image-content'),
    ]);

    $disk->assertMissing($path);

    // Act
    $response = $this->getJson(route('api-v2:external.images.tm.show', ['path' => 'test-image']));

    // Assert
    $disk->assertExists($path);
    $this->assertEquals('external-image-content', $disk->get($path));

    $temporaryUrl = $disk->temporaryUrl($path, Carbon::now()->addHour());
    $response->assertRedirect($temporaryUrl);
}
```

#### Key Points:
1. **`$this->freezeTime()`**: Ensures that all time-based operations in the test use the same frozen time.
2. **Storage Assertions**:
    - `assertMissing`: Verifies the file doesn’t exist before the operation.
    - `assertExists`: Confirms the file is stored after the operation.
3. **HTTP Fake**: Simulates the external API call to fetch the image.
4. **Temporary URL Validation**: Compares the expected and actual temporary URLs, which remain consistent due to time freezing.

Without freezing time, this test could fail due to mismatched timestamps in the temporary URL.


### Conclusion

Freezing time is a simple yet powerful technique to ensure the reliability of your time-sensitive tests. By combining Laravel’s test helpers (`$this->freezeTime`) and Carbon’s time manipulation methods (`Carbon::setTestNow`), you can eliminate inconsistencies caused by execution delays.

By adopting these practices, you’ll have more predictable and robust tests for temporary URLs and other time-sensitive functionality.

