# Processes

- [Introduction](#introduction)
- [Invoking Processes](#invoking-processes)
  - [Process Options](#process-options)
  - [Process Output](#process-output)
  - [Pipelines](#process-pipelines)
- [Asynchronous Processes](#asynchronous-processes)
  - [Process IDs and Signals](#process-ids-and-signals)
  - [Asynchronous Process Output](#asynchronous-process-output)
  - [Asynchronous Process Timeouts](#asynchronous-process-timeouts)
- [Concurrent Processes](#concurrent-processes)
  - [Naming Pool Processes](#naming-pool-processes)
  - [Pool Process IDs and Signals](#pool-process-ids-and-signals)
- [Testing](#testing)
  - [Faking Processes](#faking-processes)
  - [Faking Specific Processes](#faking-specific-processes)
  - [Faking Process Sequences](#faking-process-sequences)
  - [Faking Asynchronous Process Lifecycles](#faking-asynchronous-process-lifecycles)
  - [Available Assertions](#available-assertions)
  - [Preventing Stray Processes](#preventing-stray-processes)

<a name="introduction"></a>

## Introduction

Laravel cung cấp một API biểu đạt, tối thiểu xung quanh [component Process của Symfony](https://symfony.com/doc/current/components/process.html), cho phép bạn thuận tiện gọi các processes bên ngoài từ ứng dụng Laravel của bạn. Các tính năng process của Laravel tập trung vào các trường hợp sử dụng phổ biến nhất và trải nghiệm nhà phát triển tuyệt vời.

<a name="invoking-processes"></a>

## Invoking Processes

Để gọi một process, bạn có thể sử dụng các phương thức `run` và `start` được cung cấp bởi facade `Process`. Phương thức `run` sẽ gọi một process và chờ process hoàn thành thực thi, trong khi phương thức `start` được sử dụng cho thực thi process bất đồng bộ. Chúng tôi sẽ xem xét cả hai cách tiếp cận trong tài liệu này. Trước hết, hãy xem cách gọi một process đồng bộ cơ bản và kiểm tra kết quả của nó:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

Tất nhiên, instance `Illuminate\Contracts\Process\ProcessResult` được trả về bởi phương thức `run` cung cấp nhiều phương thức hữu ích có thể được sử dụng để kiểm tra kết quả process:

```php
$result = Process::run('ls -la');

$result->command();
$result->successful();
$result->failed();
$result->output();
$result->errorOutput();
$result->exitCode();
```

<a name="throwing-exceptions"></a>

#### Throwing Exceptions

Nếu bạn có một kết quả process và muốn ném một instance của `Illuminate\Process\Exceptions\ProcessFailedException` nếu mã thoát lớn hơn không (do đó chỉ ra thất bại), bạn có thể sử dụng các phương thức `throw` và `throwIf`. Nếu process không thất bại, instance `ProcessResult` sẽ được trả về:

```php
$result = Process::run('ls -la')->throw();

$result = Process::run('ls -la')->throwIf($condition);
```

<a name="process-options"></a>

### Process Options

Tất nhiên, bạn có thể cần tùy chỉnh hành vi của một process trước khi gọi nó. May mắn thay, Laravel cho phép bạn tinh chỉnh nhiều tính năng process, chẳng hạn như thư mục làm việc, timeout, và các biến môi trường.

<a name="working-directory-path"></a>

#### Working Directory Path

Bạn có thể sử dụng phương thức `path` để chỉ định thư mục làm việc của process. Nếu phương thức này không được gọi, process sẽ kế thừa thư mục làm việc của script PHP hiện đang thực thi:

```php
$result = Process::path(__DIR__)->run('ls -la');
```

<a name="input"></a>

#### Input

Bạn có thể cung cấp input thông qua "standard input" của process bằng cách sử dụng phương thức `input`:

```php
$result = Process::input('Hello World')->run('cat');
```

<a name="timeouts"></a>

#### Timeouts

Theo mặc định, các processes sẽ ném một instance của `Illuminate\Process\Exceptions\ProcessTimedOutException` sau khi thực thi hơn 60 giây. Tuy nhiên, bạn có thể tùy chỉnh hành vi này thông qua phương thức `timeout`:

```php
$result = Process::timeout(120)->run('bash import.sh');
```

Các phương thức `timeout` và `idleTimeout` cũng chấp nhận các instance `CarbonInterval`:

```php
use function Illuminate\Support\minutes;

$result = Process::timeout(minutes(2))->run('bash import.sh');
```

Hoặc, nếu bạn muốn vô hiệu hóa timeout process hoàn toàn, bạn có thể gọi phương thức `forever`:

```php
$result = Process::forever()->run('bash import.sh');
```

Phương thức `idleTimeout` có thể được sử dụng để chỉ định số giây tối đa process có thể chạy mà không trả về bất kỳ output nào:

```php
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

<a name="environment-variables"></a>

#### Environment Variables

Các biến môi trường có thể được cung cấp cho process thông qua phương thức `env`. Process được gọi cũng sẽ kế thừa tất cả các biến môi trường được định nghĩa bởi hệ thống của bạn:

```php
$result = Process::forever()
    ->env(['IMPORT_PATH' => __DIR__])
    ->run('bash import.sh');
```

Nếu bạn muốn xóa một biến môi trường được kế thừa khỏi process được gọi, bạn có thể cung cấp biến môi trường đó với giá trị `false`:

```php
$result = Process::forever()
    ->env(['LOAD_PATH' => false])
    ->run('bash import.sh');
```

<a name="tty-mode"></a>

#### TTY Mode

Phương thức `tty` có thể được sử dụng để bật chế độ TTY cho process của bạn. Chế độ TTY kết nối input và output của process với input và output của chương trình của bạn, cho phép process của bạn mở một editor như Vim hoặc Nano như một process:

```php
Process::forever()->tty()->run('vim');
```

> [!WARNING]
> Chế độ TTY không được hỗ trợ trên Windows.

<a name="process-output"></a>

### Process Output

Như đã thảo luận trước đó, output process có thể được truy cập bằng cách sử dụng các phương thức `output` (stdout) và `errorOutput` (stderr) trên kết quả process:

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

echo $result->output();
echo $result->errorOutput();
```

Tuy nhiên, output cũng có thể được thu thập theo thời gian thực bằng cách chuyển một closure làm đối số thứ hai cho phương thức `run`. Closure sẽ nhận hai đối số: "type" của output (`stdout` hoặc `stderr`) và chuỗi output chính nó:

```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```

Laravel cũng cung cấp các phương thức `seeInOutput` và `seeInErrorOutput`, cung cấp một cách thuận tiện để xác định xem một chuỗi nhất định có được chứa trong output của process hay không:

```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

<a name="disabling-process-output"></a>

#### Disabling Process Output

Nếu process của bạn đang viết một lượng lớn output mà bạn không quan tâm, bạn có thể tiết kiệm bộ nhớ bằng cách vô hiệu hóa việc truy xuất output hoàn toàn. Để thực hiện điều này, gọi phương thức `quietly` khi xây dựng process:

```php
use Illuminate\Support\Facades\Process;

$result = Process::quietly()->run('bash import.sh');
```

<a name="process-pipelines"></a>

### Pipelines

Đôi khi bạn muốn làm cho output của một process trở thành input của một process khác. Điều này thường được gọi là "piping" output của một process vào một process khác. Phương thức `pipe` được cung cấp bởi các facades `Process` làm cho việc này dễ dàng thực hiện. Phương thức `pipe` sẽ thực thi các processes được pipe đồng bộ và trả về kết quả process cho process cuối cùng trong pipeline:

```php
use Illuminate\Process\Pipe;
use Illuminate\Support\Facades\Process;

$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
});

if ($result->successful()) {
    // ...
}
```

Nếu bạn không cần tùy chỉnh các processes riêng lẻ tạo nên pipeline, bạn có thể chỉ chuyển một array các chuỗi lệnh cho phương thức `pipe`:

```php
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```

Output process có thể được thu thập theo thời gian thực bằng cách chuyển một closure làm đối số thứ hai cho phương thức `pipe`. Closure sẽ nhận hai đối số: "type" của output (`stdout` hoặc `stderr`) và chuỗi output chính nó:

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
}, function (string $type, string $output) {
    echo $output;
});
```

Laravel cũng cho phép bạn gán các key chuỗi cho mỗi process trong một pipeline thông qua phương thức `as`. Key này cũng sẽ được chuyển cho closure output được cung cấp cho phương thức `pipe`, cho phép bạn xác định process nào output thuộc về:

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->as('first')->command('cat example.txt');
    $pipe->as('second')->command('grep -i "laravel"');
}, function (string $type, string $output, string $key) {
    // ...
});
```

<a name="asynchronous-processes"></a>

## Asynchronous Processes

Trong khi phương thức `run` gọi các processes đồng bộ, phương thức `start` có thể được sử dụng để gọi một process bất đồng bộ. Điều này cho phép ứng dụng của bạn tiếp tục thực hiện các tác vụ khác trong khi process chạy trong nền. Sau khi process đã được gọi, bạn có thể sử dụng phương thức `running` để xác định xem process có vẫn đang chạy hay không:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // ...
}

$result = $process->wait();
```

Như bạn có thể nhận thấy, bạn có thể gọi phương thức `wait` để chờ cho đến khi process hoàn thành thực thi và truy xuất instance `ProcessResult`:

```php
$process = Process::timeout(120)->start('bash import.sh');

// ...

$result = $process->wait();
```

<a name="process-ids-and-signals"></a>

### Process IDs and Signals

Phương thức `id` có thể được sử dụng để truy xuất ID process được gán bởi hệ điều hành của process đang chạy:

```php
$process = Process::start('bash import.sh');

return $process->id();
```

Bạn có thể sử dụng phương thức `signal` để gửi một "signal" đến process đang chạy. Một danh sách các hằng số signal được định trước có thể được tìm thấy trong [tài liệu PHP](https://www.php.net/manual/en/pcntl.constants.php):

```php
$process->signal(SIGUSR2);
```

<a name="asynchronous-process-output"></a>

### Asynchronous Process Output

Trong khi một process bất đồng bộ đang chạy, bạn có thể truy cập toàn bộ output hiện tại của nó bằng cách sử dụng các phương thức `output` và `errorOutput`; tuy nhiên, bạn có thể sử dụng `latestOutput` và `latestErrorOutput` để truy xuất output từ process đã xảy ra kể từ khi output được truy xuất lần cuối:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    echo $process->latestOutput();
    echo $process->latestErrorOutput();

    sleep(1);
}
```

Như phương thức `run`, output cũng có thể được thu thập theo thời gian thực từ các processes bất đồng bộ bằng cách chuyển một closure làm đối số thứ hai cho phương thức `start`. Closure sẽ nhận hai đối số: "type" của output (`stdout` hoặc `stderr`) và chuỗi output chính nó:

```php
$process = Process::start('bash import.sh', function (string $type, string $output) {
    echo $output;
});

$result = $process->wait();
```

Thay vì chờ cho đến khi process hoàn thành, bạn có thể sử dụng phương thức `waitUntil` để ngừng chờ dựa trên output của process. Laravel sẽ ngừng chờ process hoàn thành khi closure được đưa cho phương thức `waitUntil` trả về `true`:

```php
$process = Process::start('bash import.sh');

$process->waitUntil(function (string $type, string $output) {
    return $output === 'Ready...';
});
```

<a name="asynchronous-process-timeouts"></a>

### Asynchronous Process Timeouts

Trong khi một process bất đồng bộ đang chạy, bạn có thể xác minh rằng process không bị timeout bằng cách sử dụng phương thức `ensureNotTimedOut`. Phương thức này sẽ ném một [ngoại lệ timeout](#timeouts) nếu process đã bị timeout:

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    $process->ensureNotTimedOut();

    // ...

    sleep(1);
}
```

<a name="concurrent-processes"></a>

## Concurrent Processes

Laravel cũng làm cho việc quản lý một pool các processes bất đồng bộ, đồng thời trở nên dễ dàng, cho phép bạn dễ dàng thực thi nhiều tác vụ đồng thời. Để bắt đầu, gọi phương thức `pool`, chấp nhận một closure nhận một instance của `Illuminate\Process\Pool`.

Trong closure này, bạn có thể định nghĩa các processes thuộc về pool. Sau khi một pool process được bắt đầu thông qua phương thức `start`, bạn có thể truy cập [collection](/docs/{{version}}/collections) các processes đang chạy thông qua phương thức `running`:

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

$pool = Process::pool(function (Pool $pool) {
    $pool->path(__DIR__)->command('bash import-1.sh');
    $pool->path(__DIR__)->command('bash import-2.sh');
    $pool->path(__DIR__)->command('bash import-3.sh');
})->start(function (string $type, string $output, int $key) {
    // ...
});

while ($pool->running()->isNotEmpty()) {
    // ...
}

$results = $pool->wait();
```

Như bạn có thể thấy, bạn có thể chờ tất cả các processes pool hoàn thành thực thi và giải quyết kết quả của chúng thông qua phương thức `wait`. Phương thức `wait` trả về một object có thể truy cập bằng array cho phép bạn truy xuất instance `ProcessResult` của mỗi process trong pool bằng key của nó:

```php
$results = $pool->wait();

echo $results[0]->output();
```

Hoặc, để thuận tiện, phương thức `concurrently` có thể được sử dụng để bắt đầu một pool process bất đồng bộ và ngay lập tức chờ kết quả của nó. Điều này có thể cung cấp cú pháp đặc biệt biểu đạt khi kết hợp với các khả năng hủy array của PHP:

```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});

echo $first->output();
```

<a name="naming-pool-processes"></a>

### Naming Pool Processes

Truy xuất kết quả pool process thông qua một key số không rất biểu đạt; do đó, Laravel cho phép bạn gán các key chuỗi cho mỗi process trong một pool thông qua phương thức `as`. Key này cũng sẽ được chuyển cho closure được cung cấp cho phương thức `start`, cho phép bạn xác định process nào output thuộc về:

```php
$pool = Process::pool(function (Pool $pool) {
    $pool->as('first')->command('bash import-1.sh');
    $pool->as('second')->command('bash import-2.sh');
    $pool->as('third')->command('bash import-3.sh');
})->start(function (string $type, string $output, string $key) {
    // ...
});

$results = $pool->wait();

return $results['first']->output();
```

<a name="pool-process-ids-and-signals"></a>

### Pool Process IDs and Signals

Vì phương thức `running` của pool process cung cấp một collection của tất cả các processes được gọi trong pool, bạn có thể dễ dàng truy xuất các ID process pool bên dưới:

```php
$processIds = $pool->running()->each->id();
```

Và, để thuận tiện, bạn có thể gọi phương thức `signal` trên một pool process để gửi một signal đến mỗi process trong pool:

```php
$pool->signal(SIGUSR2);
```

<a name="testing"></a>

## Testing

Nhiều dịch vụ Laravel cung cấp chức năng để giúp bạn viết các tests dễ dàng và biểu đạt, và dịch vụ process của Laravel không ngoại lệ. Phương thức `fake` của facade `Process` cho phép bạn hướng dẫn Laravel trả về kết quả stub / dummy khi các processes được gọi.

<a name="faking-processes"></a>

### Faking Processes

Để khám phá khả năng fake processes của Laravel, hãy tưởng tượng một route gọi một process:

```php
use Illuminate\Support\Facades\Process;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    Process::run('bash import.sh');

    return 'Import complete!';
});
```

Khi test route này, chúng ta có thể hướng dẫn Laravel trả về một kết quả process giả, thành công cho mỗi process được gọi bằng cách gọi phương thức `fake` trên facade `Process` mà không có đối số. Ngoài ra, chúng ta thậm chí có thể [assert](#available-assertions) rằng một process nhất định đã được "run":

```php
<?php

use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Process\PendingProcess;
use Illuminate\Support\Facades\Process;

test('process is invoked', function () {
    Process::fake();

    $response = $this->get('/import');

    // Simple process assertion...
    Process::assertRan('bash import.sh');

    // Or, inspecting the process configuration...
    Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
        return $process->command === 'bash import.sh' &&
               $process->timeout === 60;
    });
});
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Process\PendingProcess;
use Illuminate\Support\Facades\Process;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_process_is_invoked(): void
    {
        Process::fake();

        $response = $this->get('/import');

        // Simple process assertion...
        Process::assertRan('bash import.sh');

        // Or, inspecting the process configuration...
        Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
            return $process->command === 'bash import.sh' &&
                   $process->timeout === 60;
        });
    }
}
```

Như đã thảo luận, gọi phương thức `fake` trên facade `Process` sẽ hướng dẫn Laravel luôn trả về một kết quả process thành công mà không có output. Tuy nhiên, bạn có thể dễ dàng chỉ định output và mã thoát cho các processes được fake bằng cách sử dụng phương thức `result` của facade `Process`:

```php
Process::fake([
    '*' => Process::result(
        output: 'Test output',
        errorOutput: 'Test error output',
        exitCode: 1,
    ),
]);
```

<a name="faking-specific-processes"></a>

### Faking Specific Processes

Như bạn có thể nhận thấy trong một ví dụ trước đó, facade `Process` cho phép bạn chỉ định các kết quả giả khác nhau cho mỗi process bằng cách chuyển một array cho phương thức `fake`.

Các key của array nên đại diện cho các pattern lệnh bạn muốn fake và kết quả liên quan của chúng. Ký tự `*` có thể được sử dụng làm ký tự đại diện. Bất kỳ lệnh process nào không được fake sẽ thực sự được gọi. Bạn có thể sử dụng phương thức `result` của facade `Process` để xây dựng các kết quả stub / fake cho các lệnh này:

```php
Process::fake([
    'cat *' => Process::result(
        output: 'Test "cat" output',
    ),
    'ls *' => Process::result(
        output: 'Test "ls" output',
    ),
]);
```

Nếu bạn không cần tùy chỉnh mã thoát hoặc error output của một process được fake, bạn có thể thấy thuận tiện hơn để chỉ định kết quả process giả dưới dạng các chuỗi đơn giản:

```php
Process::fake([
    'cat *' => 'Test "cat" output',
    'ls *' => 'Test "ls" output',
]);
```

<a name="faking-process-sequences"></a>

### Faking Process Sequences

Nếu mã bạn đang test gọi nhiều processes với cùng một lệnh, bạn có thể muốn gán một kết quả process giả khác cho mỗi lần gọi process. Bạn có thể thực hiện điều này thông qua phương thức `sequence` của facade `Process`:

```php
Process::fake([
    'ls *' => Process::sequence()
        ->push(Process::result('First invocation'))
        ->push(Process::result('Second invocation')),
]);
```

<a name="faking-asynchronous-process-lifecycles"></a>

### Faking Asynchronous Process Lifecycles

Cho đến nay, chúng tôi đã chủ yếu thảo luận về việc fake các processes được gọi đồng bộ bằng cách sử dụng phương thức `run`. Tuy nhiên, nếu bạn đang cố gắng test mã tương tác với các processes bất đồng bộ được gọi thông qua `start`, bạn có thể cần một cách tiếp cận tinh vi hơn để mô tả các processes giả của bạn.

Ví dụ, hãy tưởng tượng route sau tương tác với một process bất đồng bộ:

```php
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    $process = Process::start('bash import.sh');

    while ($process->running()) {
        Log::info($process->latestOutput());
        Log::info($process->latestErrorOutput());
    }

    return 'Done';
});
```

Để fake đúng process này, chúng ta cần có thể mô tả bao nhiêu lần phương thức `running` nên trả về `true`. Ngoài ra, chúng tôi có thể muốn chỉ định nhiều dòng output nên được trả về theo trình tự. Để thực hiện điều này, chúng ta có thể sử dụng phương thức `describe` của facade `Process`:

```php
Process::fake([
    'bash import.sh' => Process::describe()
        ->output('First line of standard output')
        ->errorOutput('First line of error output')
        ->output('Second line of standard output')
        ->exitCode(0)
        ->iterations(3),
]);
```

Hãy đi sâu vào ví dụ trên. Sử dụng các phương thức `output` và `errorOutput`, chúng ta có thể chỉ định nhiều dòng output sẽ được trả về theo trình tự. Phương thức `exitCode` có thể được sử dụng để chỉ định mã thoát cuối cùng của process giả. Cuối cùng, phương thức `iterations` có thể được sử dụng để chỉ định bao nhiêu lần phương thức `running` nên trả về `true`.

<a name="available-assertions"></a>

### Available Assertions

Như [đã thảo luận trước đó](#faking-processes), Laravel cung cấp một số assertions process cho các feature tests của bạn. Chúng tôi sẽ thảo luận từng assertion này dưới đây.

<a name="assert-process-ran"></a>

#### assertRan

Assert rằng một process nhất định đã được gọi:

```php
use Illuminate\Support\Facades\Process;

Process::assertRan('ls -la');
```

Phương thức `assertRan` cũng chấp nhận một closure, sẽ nhận một instance của một process và một kết quả process, cho phép bạn kiểm tra các tùy chọn được cấu hình của process. Nếu closure này trả về `true`, assertion sẽ "pass":

```php
Process::assertRan(fn ($process, $result) =>
    $process->command === 'ls -la' &&
    $process->path === __DIR__ &&
    $process->timeout === 60
);
```

`$process` được chuyển cho closure `assertRan` là một instance của `Illuminate\Process\PendingProcess`, trong khi `$result` là một instance của `Illuminate\Contracts\Process\ProcessResult`.

<a name="assert-process-didnt-run"></a>

#### assertDidntRun

Assert rằng một process nhất định không được gọi:

```php
use Illuminate\Support\Facades\Process;

Process::assertDidntRun('ls -la');
```

Như phương thức `assertRan`, phương thức `assertDidntRun` cũng chấp nhận một closure, sẽ nhận một instance của một process và một kết quả process, cho phép bạn kiểm tra các tùy chọn được cấu hình của process. Nếu closure này trả về `true`, assertion sẽ "fail":

```php
Process::assertDidntRun(fn (PendingProcess $process, ProcessResult $result) =>
    $process->command === 'ls -la'
);
```

<a name="assert-process-ran-times"></a>

#### assertRanTimes

Assert rằng một process nhất định đã được gọi một số lần nhất định:

```php
use Illuminate\Support\Facades\Process;

Process::assertRanTimes('ls -la', times: 3);
```

Phương thức `assertRanTimes` cũng chấp nhận một closure, sẽ nhận một instance của `PendingProcess` và `ProcessResult`, cho phép bạn kiểm tra các tùy chọn được cấu hình của process. Nếu closure này trả về `true` và process được gọi số lần được chỉ định, assertion sẽ "pass":

```php
Process::assertRanTimes(function (PendingProcess $process, ProcessResult $result) {
    return $process->command === 'ls -la';
}, times: 3);
```

<a name="preventing-stray-processes"></a>

### Preventing Stray Processes

Nếu bạn muốn đảm bảo rằng tất cả các processes được gọi đã được fake trong suốt test riêng lẻ hoặc bộ test hoàn chỉnh của bạn, bạn có thể gọi phương thức `preventStrayProcesses`. Sau khi gọi phương thức này, bất kỳ processes nào không có kết quả giả tương ứng sẽ ném một ngoại lệ thay vì bắt đầu một process thực tế:

```php
use Illuminate\Support\Facades\Process;

Process::preventStrayProcesses();

Process::fake([
    'ls *' => 'Test output...',
]);

// Fake response is returned...
Process::run('ls -la');

// An exception is thrown...
Process::run('bash import.sh');
```
