# Database Testing

- [Introduction](#introduction)
  - [Resetting the Database After Each Test](#resetting-the-database-after-each-test)
- [Model Factories](#model-factories)
- [Running Seeders](#running-seeders)
- [Available Assertions](#available-assertions)

<a name="introduction"></a>

## Introduction

Laravel cung cấp nhiều công cụ và assertion hữu ích để giúp việc test các ứng dụng database-driven dễ dàng hơn. Ngoài ra, Laravel model factories và seeders làm cho việc tạo test database records sử dụng Eloquent models và relationships của ứng dụng trở nên dễ dàng. Chúng ta sẽ thảo luận về tất cả các tính năng mạnh mẽ này trong tài liệu sau.

<a name="resetting-the-database-after-each-test"></a>

### Resetting the Database After Each Test

Trước khi đi xa hơn, hãy thảo luận về cách reset database của bạn sau mỗi test để data từ một test trước đó không can thiệp vào các test tiếp theo. Trait `Illuminate\Foundation\Testing\RefreshDatabase` được tích hợp sẵn của Laravel sẽ lo việc này cho bạn. Chỉ cần sử dụng trait trên test class của bạn:

```php
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('basic example', function () {
    $response = $this->get('/');

    // ...
});
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        $response = $this->get('/');

        // ...
    }
}
```

Trait `Illuminate\Foundation\Testing\RefreshDatabase` không migrate database của bạn nếu schema của bạn đã cập nhật. Thay vào đó, nó sẽ chỉ thực thi test trong một database transaction. Do đó, bất kỳ records nào được thêm vào database bởi test cases không sử dụng trait này có thể vẫn tồn tại trong database.

Nếu bạn muốn hoàn toàn reset database, bạn có thể sử dụng các traits `Illuminate\Foundation\Testing\DatabaseMigrations` hoặc `Illuminate\Foundation\Testing\DatabaseTruncation` thay thế. Tuy nhiên, cả hai tùy chọn này đều chậm hơn đáng kể so với trait `RefreshDatabase`.

<a name="model-factories"></a>

## Model Factories

Khi test, bạn có thể cần chèn một vài records vào database của bạn trước khi thực thi test. Thay vì chỉ định thủ công giá trị của mỗi column khi bạn tạo test data này, Laravel cho phép bạn định nghĩa một tập hợp default attributes cho mỗi [Eloquent models](/docs/{{version}}/eloquent) của bạn sử dụng [model factories](/docs/{{version}}/eloquent-factories).

Để tìm hiểu thêm về việc tạo và sử dụng model factories để tạo models, vui lòng tham khảo tài liệu [model factory documentation](/docs/{{version}}/eloquent-factories) đầy đủ. Sau khi bạn đã định nghĩa một model factory, bạn có thể sử dụng factory trong test của bạn để tạo models:

```php
use App\Models\User;

test('models can be instantiated', function () {
    $user = User::factory()->create();

    // ...
});
```

```php
use App\Models\User;

public function test_models_can_be_instantiated(): void
{
    $user = User::factory()->create();

    // ...
}
```

<a name="running-seeders"></a>

## Running Seeders

Nếu bạn muốn sử dụng [database seeders](/docs/{{version}}/seeding) để populate database của bạn trong một feature test, bạn có thể gọi method `seed`. Theo mặc định, method `seed` sẽ thực thi `DatabaseSeeder`, nên thực thi tất cả các seeders khác của bạn. Ngoài ra, bạn có thể truyền một seeder class name cụ thể cho method `seed`:

```php
<?php

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('orders can be created', function () {
    // Run the DatabaseSeeder...
    $this->seed();

    // Run a specific seeder...
    $this->seed(OrderStatusSeeder::class);

    // ...

    // Run an array of specific seeders...
    $this->seed([
        OrderStatusSeeder::class,
        TransactionStatusSeeder::class,
        // ...
    ]);
});
```

```php
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Test creating a new order.
     */
    public function test_orders_can_be_created(): void
    {
        // Run the DatabaseSeeder...
        $this->seed();

        // Run a specific seeder...
        $this->seed(OrderStatusSeeder::class);

        // ...

        // Run an array of specific seeders...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    }
}
```

Ngoài ra, bạn có thể hướng dẫn Laravel tự động seed database trước mỗi test sử dụng trait `RefreshDatabase`. Bạn có thể thực hiện điều này bằng cách thêm attribute `Seed` vào base test class của bạn:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\Attributes\Seed;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

#[Seed]
abstract class TestCase extends BaseTestCase
{
}
```

Khi attribute `Seed` có mặt, test sẽ chạy class `Database\Seeders\DatabaseSeeder` trước mỗi test sử dụng trait `RefreshDatabase`. Tuy nhiên, bạn có thể chỉ định một seeder cụ thể nên được thực thi bằng cách sử dụng attribute `Seeder` trên test class của bạn:

```php
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Illuminate\Foundation\Testing\Attributes\Seeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

#[Seeder(OrderStatusSeeder::class)]
class OrderTest extends TestCase
{
    use RefreshDatabase;

    // ...
}
```

<a name="available-assertions"></a>

## Available Assertions

Laravel cung cấp nhiều database assertions cho [Pest](https://pestphp.com) hoặc [PHPUnit](https://phpunit.de) feature tests của bạn. Chúng ta sẽ thảo luận về từng assertion này dưới đây.

<a name="assert-database-count"></a>

#### assertDatabaseCount

Xác nhận rằng một table trong database chứa số lượng records đã cho:

```php
$this->assertDatabaseCount('users', 5);
```

<a name="assert-database-empty"></a>

#### assertDatabaseEmpty

Xác nhận rằng một table trong database không chứa bất kỳ records nào:

```php
$this->assertDatabaseEmpty('users');
```

<a name="assert-database-has"></a>

#### assertDatabaseHas

Xác nhận rằng một table trong database chứa records khớp với các key / value query constraints đã cho:

```php
$this->assertDatabaseHas('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-database-missing"></a>

#### assertDatabaseMissing

Xác nhận rằng một table trong database không chứa records khớp với các key / value query constraints đã cho:

```php
$this->assertDatabaseMissing('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-deleted"></a>

#### assertSoftDeleted

Method `assertSoftDeleted` có thể được sử dụng để xác nhận một Eloquent model đã cho đã được "soft deleted":

```php
$this->assertSoftDeleted($user);
```

<a name="assert-not-deleted"></a>

#### assertNotSoftDeleted

Method `assertNotSoftDeleted` có thể được sử dụng để xác nhận một Eloquent model đã cho chưa được "soft deleted":

```php
$this->assertNotSoftDeleted($user);
```

<a name="assert-model-exists"></a>

#### assertModelExists

Xác nhận rằng một model hoặc collection của models đã cho tồn tại trong database:

```php
use App\Models\User;

$user = User::factory()->create();

$this->assertModelExists($user);
```

<a name="assert-model-missing"></a>

#### assertModelMissing

Xác nhận rằng một model hoặc collection của models đã cho không tồn tại trong database:

```php
use App\Models\User;

$user = User::factory()->create();

$user->delete();

$this->assertModelMissing($user);
```

<a name="expects-database-query-count"></a>

#### expectsDatabaseQueryCount

Method `expectsDatabaseQueryCount` có thể được gọi ở đầu test của bạn để chỉ định tổng số database queries mà bạn mong đợi được chạy trong test. Nếu số lượng thực tế của các queries được thực thi không khớp chính xác với mong đợi này, test sẽ thất bại:

```php
$this->expectsDatabaseQueryCount(5);

// Test...
```
