# Database: Seeding

- [Introduction](#introduction)
- [Writing Seeders](#writing-seeders)
    - [Using Model Factories](#using-model-factories)
    - [Calling Additional Seeders](#calling-additional-seeders)
    - [Muting Model Events](#muting-model-events)
- [Running Seeders](#running-seeders)

<a name="introduction"></a>
## Introduction

Laravel bao gồm khả năng seed database của bạn với dữ liệu bằng cách sử dụng các seed classes. Tất cả các seed classes được lưu trữ trong thư mục `database/seeders`. Theo mặc định, một class `DatabaseSeeder` được định nghĩa cho bạn. Từ class này, bạn có thể sử dụng phương thức `call` để chạy các seed classes khác, cho phép bạn kiểm soát thứ tự seeding.

> [!NOTE]
> [Mass assignment protection](/docs/{{version}}/eloquent#mass-assignment) tự động bị tắt trong quá trình database seeding.

<a name="writing-seeders"></a>
## Writing Seeders

Để tạo một seeder, thực thi [Artisan command](/docs/{{version}}/artisan) `make:seeder`. Tất cả các seeders được tạo bởi framework sẽ được đặt trong thư mục `database/seeders`:

```shell
php artisan make:seeder UserSeeder
```

Một seeder class chỉ chứa một phương thức theo mặc định: `run`. Phương thức này được gọi khi [Artisan command](/docs/{{version}}/artisan) `db:seed` được thực thi. Trong phương thức `run`, bạn có thể chèn dữ liệu vào database của mình theo cách bạn muốn. Bạn có thể sử dụng [query builder](/docs/{{version}}/queries) để chèn dữ liệu thủ công hoặc bạn có thể sử dụng [Eloquent model factories](/docs/{{version}}/eloquent-factories).

Ví dụ, hãy sửa đổi class `DatabaseSeeder` mặc định và thêm một câu lệnh insert database vào phương thức `run`:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeders.
     */
    public function run(): void
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

> [!NOTE]
> Bạn có thể type-hint bất kỳ dependencies nào bạn cần trong signature của phương thức `run`. Chúng sẽ tự động được giải quyết thông qua Laravel [service container](/docs/{{version}}/container).

<a name="using-model-factories"></a>
### Using Model Factories

Tất nhiên, chỉ định các thuộc tính cho mỗi model seed một cách thủ công là tốn công. Thay vào đó, bạn có thể sử dụng [model factories](/docs/{{version}}/eloquent-factories) để tạo thuận tiện một lượng lớn các database records. Trước tiên, hãy xem lại [model factory documentation](/docs/{{version}}/eloquent-factories) để tìm hiểu cách định nghĩa các factories của bạn.

Ví dụ, hãy tạo 50 users mà mỗi user có một post liên quan:

```php
use App\Models\User;

/**
 * Run the database seeders.
 */
public function run(): void
{
    User::factory()
        ->count(50)
        ->hasPosts(1)
        ->create();
}
```

<a name="calling-additional-seeders"></a>
### Calling Additional Seeders

Trong class `DatabaseSeeder`, bạn có thể sử dụng phương thức `call` để thực thi các seed classes bổ sung. Sử dụng phương thức `call` cho phép bạn chia nhỏ database seeding thành nhiều files để không có seeder class nào trở nên quá lớn. Phương thức `call` chấp nhận một mảng các seeder classes nên được thực thi:

```php
/**
 * Run the database seeders.
 */
public function run(): void
{
    $this->call([
        UserSeeder::class,
        PostSeeder::class,
        CommentSeeder::class,
    ]);
}
```

<a name="muting-model-events"></a>
### Muting Model Events

Trong khi chạy seeds, bạn có thể muốn ngăn models dispatch các events. Bạn có thể đạt được điều này bằng cách sử dụng trait `WithoutModelEvents`. Khi được sử dụng, trait `WithoutModelEvents` đảm bảo không có model events nào được dispatch, ngay cả khi các seed classes bổ sung được thực thi thông qua phương thức `call`:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents;

    /**
     * Run the database seeders.
     */
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

<a name="running-seeders"></a>
## Running Seeders

Bạn có thể thực thi Artisan command `db:seed` để seed database của bạn. Theo mặc định, command `db:seed` chạy class `Database\Seeders\DatabaseSeeder`, class này có thể lần lượt gọi các seed classes khác. Tuy nhiên, bạn có thể sử dụng tùy chọn `--class` để chỉ định một seeder class cụ thể để chạy riêng lẻ:

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

Bạn cũng có thể seed database của mình bằng cách sử dụng command `migrate:fresh` kết hợp với tùy chọn `--seed`, command này sẽ drop tất cả các bảng và chạy lại tất cả các migrations của bạn. Command này hữu ích để tái tạo hoàn toàn database của bạn. Tùy chọn `--seeder` có thể được sử dụng để chỉ định một seeder cụ thể để chạy:

```shell
php artisan migrate:fresh --seed

php artisan migrate:fresh --seed --seeder=UserSeeder
```

<a name="forcing-seeding-production"></a>
#### Forcing Seeders to Run in Production

Một số thao tác seeding có thể khiến bạn thay đổi hoặc mất dữ liệu. Để bảo vệ bạn khỏi việc chạy các commands seeding đối với database production của bạn, bạn sẽ được nhắc xác nhận trước khi các seeders được thực thi trong môi trường `production`. Để buộc các seeders chạy mà không có prompt, sử dụng flag `--force`:

```shell
php artisan db:seed --force
```
