# Console Tests

- [Introduction](#introduction)
- [Success / Failure Expectations](#success-failure-expectations)
- [Input / Output Expectations](#input-output-expectations)
- [Console Events](#console-events)

<a name="introduction"></a>

## Introduction

Ngoài việc đơn giản hóa HTTP testing, Laravel cung cấp một API đơn giản để test [custom console commands](/docs/{{version}}/artisan) của ứng dụng.

<a name="success-failure-expectations"></a>

## Success / Failure Expectations

Để bắt đầu, hãy khám phá cách tạo assertion về exit code của một Artisan command. Để thực hiện điều này, chúng ta sẽ sử dụng method `artisan` để gọi một Artisan command từ test của chúng ta. Sau đó, chúng ta sẽ sử dụng method `assertExitCode` để xác nhận rằng command hoàn thành với một exit code đã cho:

```php
test('console command', function () {
    $this->artisan('inspire')->assertExitCode(0);
});
```

```php
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('inspire')->assertExitCode(0);
}
```

Bạn có thể sử dụng method `assertNotExitCode` để xác nhận rằng command không exit với một exit code đã cho:

```php
$this->artisan('inspire')->assertNotExitCode(1);
```

Tất nhiên, tất cả các terminal commands thường exit với status code là `0` khi chúng thành công và một non-zero exit code khi chúng không thành công. Do đó, để thuận tiện, bạn có thể sử dụng các assertions `assertSuccessful` và `assertFailed` để xác nhận rằng một command đã cho exit với một successful exit code hoặc không:

```php
$this->artisan('inspire')->assertSuccessful();

$this->artisan('inspire')->assertFailed();
```

<a name="input-output-expectations"></a>

## Input / Output Expectations

Laravel cho phép bạn dễ dàng "mock" user input cho các console commands của bạn sử dụng method `expectsQuestion`. Ngoài ra, bạn có thể chỉ định exit code và text mà bạn mong đợi được output bởi console command sử dụng các method `assertExitCode` và `expectsOutput`. Ví dụ, hãy xem console command sau:

```php
Artisan::command('question', function () {
    $name = $this->ask('What is your name?');

    $language = $this->choice('Which language do you prefer?', [
        'PHP',
        'Ruby',
        'Python',
    ]);

    $this->line('Your name is '.$name.' and you prefer '.$language.'.');
});
```

Bạn có thể test command này với test sau:

```php
test('console command', function () {
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
});
```

```php
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
}
```

Nếu bạn đang sử dụng các functions `search` hoặc `multisearch` được cung cấp bởi [Laravel Prompts](/docs/{{version}}/prompts), bạn có thể sử dụng assertion `expectsSearch` để mock user input, search results, và selection:

```php
test('console command', function () {
    $this->artisan('example')
        ->expectsSearch('What is your name?', search: 'Tay', answers: [
            'Taylor Otwell',
            'Taylor Swift',
            'Darian Taylor'
        ], answer: 'Taylor Otwell')
        ->assertExitCode(0);
});
```

```php
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsSearch('What is your name?', search: 'Tay', answers: [
            'Taylor Otwell',
            'Taylor Swift',
            'Darian Taylor'
        ], answer: 'Taylor Otwell')
        ->assertExitCode(0);
}
```

Bạn cũng có thể xác nhận rằng một console command không tạo ra bất kỳ output nào sử dụng method `doesntExpectOutput`:

```php
test('console command', function () {
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
});
```

```php
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
}
```

Các method `expectsOutputToContain` và `doesntExpectOutputToContain` có thể được sử dụng để tạo assertion đối với một phần của output:

```php
test('console command', function () {
    $this->artisan('example')
        ->expectsOutputToContain('Taylor')
        ->assertExitCode(0);
});
```

```php
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsOutputToContain('Taylor')
        ->assertExitCode(0);
}
```

<a name="confirmation-expectations"></a>

#### Confirmation Expectations

Khi viết một command mong đợi confirmation dưới dạng câu trả lời "yes" hoặc "no", bạn có thể sử dụng method `expectsConfirmation`:

```php
$this->artisan('module:import')
    ->expectsConfirmation('Do you really wish to run this command?', 'no')
    ->assertExitCode(1);
```

<a name="table-expectations"></a>

#### Table Expectations

Nếu command của bạn hiển thị một table của thông tin sử dụng method `table` của Artisan, việc viết output expectations cho toàn bộ table có thể rất phức tạp. Thay vào đó, bạn có thể sử dụng method `expectsTable`. Method này chấp nhận headers của table như argument đầu tiên và data của table như argument thứ hai:

```php
$this->artisan('users:all')
    ->expectsTable([
        'ID',
        'Email',
    ], [
        [1, 'taylor@example.com'],
        [2, 'abigail@example.com'],
    ]);
```

<a name="console-events"></a>

## Console Events

Theo mặc định, các events `Illuminate\Console\Events\CommandStarting` và `Illuminate\Console\Events\CommandFinished` không được dispatch khi chạy tests của ứng dụng. Tuy nhiên, bạn có thể enable các events này cho một test class đã cho bằng cách thêm trait `Illuminate\Foundation\Testing\WithConsoleEvents` vào class:

```php
<?php

use Illuminate\Foundation\Testing\WithConsoleEvents;

pest()->use(WithConsoleEvents::class);

// ...
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithConsoleEvents;
use Tests\TestCase;

class ConsoleEventTest extends TestCase
{
    use WithConsoleEvents;

    // ...
}
```
