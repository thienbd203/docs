# Laravel Pint

- [Giới thiệu](#introduction)
- [Cài đặt](#installation)
- [Chạy Pint](#running-pint)
- [Cấu hình Pint](#configuring-pint)
    - [Presets](#presets)
    - [Rules](#rules)
    - [Loại trừ Files / Folders](#excluding-files-or-folders)
- [Continuous Integration](#continuous-integration)
    - [GitHub Actions](#running-tests-on-github-actions)

<a name="introduction"></a>
## Giới thiệu

[Laravel Pint](https://github.com/laravel/pint) là một công cụ sửa code style PHP có quan điểm riêng cho những người theo chủ nghĩa tối giản. Pint được xây dựng dựa trên [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) và giúp bạn dễ dàng đảm bảo code style của bạn luôn sạch và nhất quán.

Pint được cài đặt tự động với tất cả các ứng dụng Laravel mới nên bạn có thể bắt đầu sử dụng ngay lập tức. Theo mặc định, Pint không yêu cầu bất kỳ cấu hình nào và sẽ sửa các vấn đề về code style trong code của bạn bằng cách tuân theo coding style có quan điểm của Laravel.

<a name="installation"></a>
## Cài đặt

Pint được bao gồm trong các bản phát hành gần đây của Laravel framework, nên việc cài đặt thường không cần thiết. Tuy nhiên, đối với các ứng dụng cũ hơn, bạn có thể cài đặt Laravel Pint thông qua Composer:

```shell
composer require laravel/pint --dev
```

<a name="running-pint"></a>
## Chạy Pint

Bạn có thể yêu cầu Pint sửa các vấn đề về code style bằng cách gọi binary `pint` có sẵn trong thư mục `vendor/bin` của dự án:

```shell
./vendor/bin/pint
```

Nếu bạn muốn Pint chạy ở chế độ song song (experimental) để cải thiện hiệu suất, bạn có thể sử dụng tùy chọn `--parallel`:

```shell
./vendor/bin/pint --parallel
```

Chế độ song song cũng cho phép bạn chỉ định số lượng process tối đa để chạy thông qua tùy chọn `--max-processes`. Nếu tùy chọn này không được cung cấp, Pint sẽ sử dụng mọi core có sẵn trên máy của bạn:

```shell
./vendor/bin/pint --parallel --max-processes=4
```

Bạn cũng có thể chạy Pint trên các file hoặc thư mục cụ thể:

```shell
./vendor/bin/pint app/Models

./vendor/bin/pint app/Models/User.php
```

Pint sẽ hiển thị danh sách chi tiết tất cả các file mà nó cập nhật. Bạn có thể xem thêm chi tiết về các thay đổi của Pint bằng cách cung cấp tùy chọn `-v` khi gọi Pint:

```shell
./vendor/bin/pint -v
```

Nếu bạn muốn Pint chỉ kiểm tra code của bạn để tìm lỗi style mà không thực sự thay đổi các file, bạn có thể sử dụng tùy chọn `--test`. Pint sẽ trả về exit code khác 0 nếu tìm thấy bất kỳ lỗi code style nào:

```shell
./vendor/bin/pint --test
```

Nếu bạn muốn Pint chỉ sửa đổi các file khác biệt so với branch được cung cấp theo Git, bạn có thể sử dụng tùy chọn `--diff=[branch]`. Điều này có thể được sử dụng hiệu quả trong môi trường CI của bạn (như GitHub actions) để tiết kiệm thời gian bằng cách chỉ kiểm tra các file mới hoặc đã sửa đổi:

```shell
./vendor/bin/pint --diff=main
```

Nếu bạn muốn Pint chỉ sửa đổi các file có các thay đổi chưa commit theo Git, bạn có thể sử dụng tùy chọn `--dirty`:

```shell
./vendor/bin/pint --dirty
```

Nếu bạn muốn Pint sửa bất kỳ file nào có lỗi code style nhưng cũng thoát với exit code khác 0 nếu có bất kỳ lỗi nào được sửa, bạn có thể sử dụng tùy chọn `--repair`:

```shell
./vendor/bin/pint --repair
```

<a name="configuring-pint"></a>
## Cấu hình Pint

Như đã đề cập trước đó, Pint không yêu cầu bất kỳ cấu hình nào. Tuy nhiên, nếu bạn muốn tùy chỉnh presets, rules, hoặc các thư mục được kiểm tra, bạn có thể làm điều đó bằng cách tạo file `pint.json` trong thư mục gốc của dự án:

```json
{
    "preset": "laravel"
}
```

Ngoài ra, nếu bạn muốn sử dụng `pint.json` từ một thư mục cụ thể, bạn có thể cung cấp tùy chọn `--config` khi gọi Pint:

```shell
./vendor/bin/pint --config vendor/my-company/coding-style/pint.json
```

<a name="presets"></a>
### Presets

Presets định nghĩa một tập hợp các rules có thể được sử dụng để sửa các vấn đề về code style trong code của bạn. Theo mặc định, Pint sử dụng preset `laravel`, sẽ sửa các vấn đề bằng cách tuân theo coding style có quan điểm của Laravel. Tuy nhiên, bạn có thể chỉ định một preset khác bằng cách cung cấp tùy chọn `--preset` cho Pint:

```shell
./vendor/bin/pint --preset psr12
```

Nếu bạn muốn, bạn cũng có thể đặt preset trong file `pint.json` của dự án:

```json
{
    "preset": "psr12"
}
```

Các preset hiện được hỗ trợ bởi Pint là: `laravel`, `per`, `psr12`, `symfony`, và `empty`.

<a name="rules"></a>
### Rules

Rules là các hướng dẫn style mà Pint sẽ sử dụng để sửa các vấn đề về code style trong code của bạn. Như đã đề cập ở trên, presets là các nhóm rules được định nghĩa sẵn nên sẽ hoàn hảo cho hầu hết các dự án PHP, vì vậy bạn thường không cần lo lắng về các rules riêng lẻ mà chúng chứa.

Tuy nhiên, nếu bạn muốn, bạn có thể bật hoặc tắt các rules cụ thể trong file `pint.json` của mình hoặc sử dụng preset `empty` và định nghĩa các rules từ đầu:

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "array_indentation": false,
        "new_with_parentheses": {
            "anonymous_class": true,
            "named_class": true
        }
    }
}
```

Pint được xây dựng dựa trên [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer). Do đó, bạn có thể sử dụng bất kỳ rules nào của nó để sửa các vấn đề về code style trong dự án của bạn: [PHP CS Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator).

<a name="custom-rules"></a>
#### Custom Rules

Ngoài các rules của PHP CS Fixer, Pint cung cấp các custom rules có tiền tố `Pint/`. Các rules này không được bật theo mặc định, nhưng bạn có thể bật chúng trong file `pint.json` của mình.

<a name="phpdoc-type-annotations-only"></a>
##### `Pint/phpdoc_type_annotations_only`

Rule này xóa tất cả các comment và prose docblock khỏi code của bạn, chỉ giữ lại các dòng chứa annotations `@` như `@param`, `@return`, `@var`, `@phpstan-type`, v.v:

```php
/**
 * Get the posts for the user. [tl! remove]
 * [tl! remove]
 * @return HasMany<Post, $this>
 */
public function posts(): HasMany
```

Các comment một dòng và comment block không có annotations `@` sẽ bị xóa hoàn toàn. Nếu bạn muốn giữ một comment cụ thể, bạn có thể thêm tiền tố nó với `@note`, `@warning`, hoặc `@todo`:

```php
// @note This comment will be preserved.
```

Để bật rule này, thêm nó vào file `pint.json` của bạn:

```json
{
    "preset": "laravel",
    "rules": {
        "Pint/phpdoc_type_annotations_only": true
    }
}
```

> [!NOTE]
> Rule này tự động bỏ qua các file trong thư mục `config`, vì các file cấu hình thường dựa vào comments cho tài liệu.

<a name="excluding-files-or-folders"></a>
### Loại trừ Files / Folders

Theo mặc định, Pint sẽ kiểm tra tất cả các file `.php` trong dự án của bạn ngoại trừ những file trong thư mục `vendor`. Nếu bạn muốn loại trừ thêm các thư mục, bạn có thể làm điều đó bằng cách sử dụng tùy chọn cấu hình `exclude`:

```json
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

Nếu bạn muốn loại trừ tất cả các file chứa một pattern tên cụ thể, bạn có thể làm điều đó bằng cách sử dụng tùy chọn cấu hình `notName`:

```json
{
    "notName": [
        "*-my-file.php"
    ]
}
```

Nếu bạn muốn loại trừ một file bằng cách cung cấp đường dẫn chính xác đến file, bạn có thể làm điều đó bằng cách sử dụng tùy chọn cấu hình `notPath`:

```json
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```

<a name="continuous-integration"></a>
## Continuous Integration

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

Để tự động linting dự án của bạn với Laravel Pint, bạn có thể cấu hình [GitHub Actions](https://github.com/features/actions) để chạy Pint bất cứ khi nào code mới được đẩy lên GitHub. Đầu tiên, hãy đảm bảo cấp quyền "Read and write permissions" cho workflows trong GitHub tại **Settings > Actions > General > Workflow permissions**. Sau đó, tạo file `.github/workflows/lint.yml` với nội dung sau:

```yaml
name: Fix Code Style

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
      matrix:
        php: [8.4]

    steps:
      - name: Checkout code
        uses: actions/checkout@v5

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          tools: pint

      - name: Run Pint
        run: pint

      - name: Commit linted files
        uses: stefanzweifel/git-auto-commit-action@v6
```
