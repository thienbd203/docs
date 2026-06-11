# Laravel Mix

- [Giới thiệu](#introduction)

<a name="introduction"></a>
## Giới thiệu

> [!WARNING]
> Laravel Mix là một legacy package không còn được active maintain. [Vite](/docs/{{version}}/vite) có thể được sử dụng như một modern alternative.

[Laravel Mix](https://github.com/laravel-mix/laravel-mix), một package được phát triển bởi [Laracasts](https://laracasts.com) creator Jeffrey Way, cung cấp một fluent API để định nghĩa [webpack](https://webpack.js.org) build steps cho Laravel application của bạn sử dụng một số CSS và JavaScript pre-processors phổ biến.

Nói cách khác, Mix làm cho việc compile và minify CSS và JavaScript files của application trở nên dễ dàng. Thông qua simple method chaining, bạn có thể fluently định nghĩa asset pipeline của mình. Ví dụ:

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

Nếu bạn từng bị confused và overwhelmed về việc bắt đầu với webpack và asset compilation, bạn sẽ yêu thích Laravel Mix. Tuy nhiên, bạn không bắt buộc phải sử dụng nó khi phát triển application; bạn được tự do sử dụng bất kỳ asset pipeline tool nào bạn muốn, hoặc thậm chí không sử dụng tool nào.

> [!NOTE]
> Vite đã thay thế Laravel Mix trong các Laravel installations mới. Để xem tài liệu Mix, hãy truy cập [official Laravel Mix](https://laravel-mix.com/) website. Nếu bạn muốn chuyển sang Vite, hãy xem [Vite migration guide](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite) của chúng tôi.
