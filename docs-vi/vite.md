# Asset Bundling (Vite)

- [Giới thiệu](#introduction)
- [Cài đặt & Thiết lập](#installation)
  - [Cài đặt Node](#installing-node)
  - [Cài đặt Vite và Plugin Laravel](#installing-vite-and-laravel-plugin)
  - [Cấu hình Vite](#configuring-vite)
  - [Tải Scripts và Styles của bạn](#loading-your-scripts-and-styles)
- [Chạy Vite](#running-vite)
- [Làm việc với JavaScript](#working-with-scripts)
  - [Aliases](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Svelte](#svelte)
  - [Inertia](#inertia)
  - [Xử lý URL](#url-processing)
- [Làm việc với Stylesheets](#working-with-stylesheets)
- [Làm việc với Fonts](#working-with-fonts)
  - [Font Providers](#font-providers)
  - [Local Fonts](#local-fonts)
  - [Font Options](#font-options)
- [Làm việc với Blade và Routes](#working-with-blade-and-routes)
  - [Xử lý Static Assets với Vite](#blade-processing-static-assets)
  - [Làm mới khi Lưu](#blade-refreshing-on-save)
  - [Aliases](#blade-aliases)
- [Asset Prefetching](#asset-prefetching)
- [Custom Base URLs](#custom-base-urls)
- [Biến môi trường](#environment-variables)
- [Tắt Vite trong Tests](#disabling-vite-in-tests)
- [Server-Side Rendering (SSR)](#ssr)
- [Thuộc tính thẻ Script và Style](#script-and-style-attributes)
  - [Content Security Policy (CSP) Nonce](#content-security-policy-csp-nonce)
  - [Subresource Integrity (SRI)](#subresource-integrity-sri)
  - [Thuộc tính tùy ý](#arbitrary-attributes)
- [Tùy chỉnh nâng cao](#advanced-customization)
  - [Dev Server Cross-Origin Resource Sharing (CORS)](#cors)
  - [Sửa lại Dev Server URLs](#correcting-dev-server-urls)

<a name="introduction"></a>
## Giới thiệu

[Vite](https://vitejs.dev) là một công cụ build frontend hiện đại cung cấp môi trường phát triển cực kỳ nhanh và đóng gói code của bạn cho production. Khi xây dựng ứng dụng với Laravel, bạn thường sẽ sử dụng Vite để đóng gói các file CSS và JavaScript của ứng dụng thành các tài sản sẵn sàng cho production.

Laravel tích hợp liền mạch với Vite bằng cách cung cấp plugin chính thức và directive Blade để tải tài sản của bạn cho phát triển và production.

<a name="installation"></a>
## Cài đặt & Thiết lập

> [!NOTE]
> Tài liệu dưới đây thảo luận về cách cài đặt và cấu hình plugin Laravel Vite thủ công. Tuy nhiên, [starter kits](/docs/{{version}}/starter-kits) của Laravel đã bao gồm tất cả các scaffolding này và là cách nhanh nhất để bắt đầu với Laravel và Vite.

<a name="installing-node"></a>
### Cài đặt Node

Bạn phải đảm bảo rằng Node.js (16+) và NPM được cài đặt trước khi chạy Vite và plugin Laravel:

```shell
node -v
npm -v
```

Bạn có thể dễ dàng cài đặt phiên bản mới nhất của Node và NPM bằng cách sử dụng các trình cài đặt đồ họa đơn giản từ [trang web chính thức của Node](https://nodejs.org/en/download/). Hoặc, nếu bạn đang sử dụng [Laravel Sail](https://laravel.com/docs/{{version}}/sail), bạn có thể gọi Node và NPM thông qua Sail:

```shell
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

<a name="installing-vite-and-laravel-plugin"></a>
### Cài đặt Vite và Plugin Laravel

Trong một cài đặt mới của Laravel, bạn sẽ tìm thấy file `package.json` ở gốc của cấu trúc thư mục ứng dụng của bạn. File `package.json` mặc định đã bao gồm mọi thứ bạn cần để bắt đầu sử dụng Vite và plugin Laravel. Bạn có thể cài đặt các phụ thuộc frontend của ứng dụng thông qua NPM:

```shell
npm install
```

<a name="configuring-vite"></a>
### Cấu hình Vite

Vite được cấu hình thông qua file `vite.config.js` ở gốc của dự án của bạn. Bạn có thể tùy chỉnh file này dựa trên nhu cầu của mình, và bạn cũng có thể cài đặt bất kỳ plugin nào khác mà ứng dụng của bạn yêu cầu, chẳng hạn như `@vitejs/plugin-react`, `@sveltejs/vite-plugin-svelte` hoặc `@vitejs/plugin-vue`.

Plugin Laravel Vite yêu cầu bạn chỉ định các entry points cho ứng dụng của bạn. Các này có thể là file JavaScript hoặc CSS, và bao gồm các ngôn ngữ được xử lý trước như TypeScript, JSX, TSX, và Sass.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

Nếu bạn đang xây dựng một SPA, bao gồm các ứng dụng được xây dựng bằng Inertia, Vite hoạt động tốt nhất mà không có CSS entry points:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

Thay vào đó, bạn nên import CSS của mình thông qua JavaScript. Thông thường, điều này sẽ được thực hiện trong file `resources/js/app.js` của ứng dụng của bạn:

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Plugin Laravel cũng hỗ trợ nhiều entry points và các tùy chọn cấu hình nâng cao như [SSR entry points](#ssr).

<a name="working-with-a-secure-development-server"></a>
#### Làm việc với Secure Development Server

Nếu máy chủ web phát triển cục bộ của bạn đang phục vụ ứng dụng của bạn qua HTTPS, bạn có thể gặp sự cố khi kết nối với máy chủ phát triển Vite.

Nếu bạn đang sử dụng [Laravel Herd](https://herd.laravel.com) và đã bảo mật trang web hoặc bạn đang sử dụng [Laravel Valet](/docs/{{version}}/valet) và đã chạy [lệnh secure](/docs/{{version}}/valet#securing-sites) đối với ứng dụng của mình, plugin Laravel Vite sẽ tự động phát hiện và sử dụng chứng chỉ TLS được tạo cho bạn.

Nếu bạn đã bảo mật trang web bằng cách sử dụng host không khớp với tên thư mục của ứng dụng, bạn có thể chỉ định thủ công host trong file `vite.config.js` của ứng dụng của mình:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            detectTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```

Khi sử dụng máy chủ web khác, bạn nên tạo chứng chỉ đáng tin cậy và cấu hình thủ công Vite để sử dụng các chứng chỉ được tạo:

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

Nếu bạn không thể tạo chứng chỉ đáng tin cậy cho hệ thống của mình, bạn có thể cài đặt và cấu hình plugin [@vitejs/plugin-basic-ssl](https://github.com/vitejs/vite-plugin-basic-ssl). Khi sử dụng chứng chỉ không đáng tin cậy, bạn sẽ cần chấp nhận cảnh báo chứng chỉ cho máy chủ phát triển Vite trong trình duyệt của mình bằng cách làm theo liên kết "Local" trong console của bạn khi chạy lệnh `npm run dev`.

<a name="configuring-hmr-in-sail-on-wsl2"></a>
#### Chạy Development Server trong Sail trên WSL2

Khi chạy máy chủ phát triển Vite trong [Laravel Sail](/docs/{{version}}/sail) trên Windows Subsystem for Linux 2 (WSL2), bạn nên thêm cấu hình sau vào file `vite.config.js` của mình để đảm bảo trình duyệt có thể giao tiếp với máy chủ phát triển:

```js
// ...

export default defineConfig({
    // ...
    server: { // [tl! add:start]
        hmr: {
            host: 'localhost',
        },
    }, // [tl! add:end]
});
```

Nếu các thay đổi file của bạn không được phản ánh trong trình duyệt trong khi máy chủ phát triển đang chạy, bạn cũng có thể cần cấu hình tùy chọn [server.watch.usePolling](https://vitejs.dev/config/server-options.html#server-watch) của Vite.

<a name="loading-your-scripts-and-styles"></a>
### Tải Scripts và Styles của bạn

Với các entry points Vite của bạn đã được cấu hình, bây giờ bạn có thể tham chiếu chúng trong directive `@vite()` Blade mà bạn thêm vào `<head>` của template gốc của ứng dụng:

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

Nếu bạn đang import CSS của mình thông qua JavaScript, bạn chỉ cần bao gồm entry point JavaScript:

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

Directive `@vite` sẽ tự động phát hiện máy chủ phát triển Vite và inject client Vite để kích hoạt Hot Module Replacement. Trong chế độ build, directive sẽ tải các tài sản đã biên dịch và có phiên bản, bao gồm bất kỳ CSS nào được import.

Nếu cần, bạn cũng có thể chỉ định đường dẫn build của các tài sản đã biên dịch khi gọi directive `@vite`:

```blade
<!doctype html>
<head>
    {{-- Đường dẫn build được đưa ra là tương đối với đường dẫn public. --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="inline-assets"></a>
#### Inline Assets

Đôi khi có thể cần thiết để bao gồm nội dung thô của tài sản thay vì liên kết đến URL có phiên bản của tài sản. Ví dụ, bạn có thể cần bao gồm nội dung tài sản trực tiếp vào trang của mình khi chuyển nội dung HTML đến trình tạo PDF. Bạn có thể xuất nội dung của tài sản Vite bằng phương thức `content` được cung cấp bởi facade `Vite`:

```blade
@use('Illuminate\Support\Facades\Vite')

<!doctype html>
<head>
    {{-- ... --}}

    <style>
        {!! Vite::content('resources/css/app.css') !!}
    </style>
    <script>
        {!! Vite::content('resources/js/app.js') !!}
    </script>
</head>
```

<a name="running-vite"></a>
## Chạy Vite

Có hai cách bạn có thể chạy Vite. Bạn có thể chạy máy chủ phát triển thông qua lệnh `dev`, điều này hữu ích khi phát triển cục bộ. Máy chủ phát triển sẽ tự động phát hiện các thay đổi đối với file của bạn và phản ánh chúng ngay lập tức trong bất kỳ cửa sổ trình duyệt nào đang mở.

Hoặc, chạy lệnh `build` sẽ tạo phiên bản và đóng gói các tài sản của ứng dụng và chuẩn bị sẵn sàng để bạn triển khai cho production:

```shell
# Chạy máy chủ phát triển Vite...
npm run dev

# Build và tạo phiên bản các tài sản cho production...
npm run build
```

Nếu bạn đang chạy máy chủ phát triển trong [Sail](/docs/{{version}}/sail) trên WSL2, bạn có thể cần một số tùy chọn [cấu hình bổ sung](#configuring-hmr-in-sail-on-wsl2).

<a name="working-with-scripts"></a>
## Làm việc với JavaScript

<a name="aliases"></a>
### Aliases

Theo mặc định, plugin Laravel cung cấp một alias phổ biến để giúp bạn bắt đầu nhanh chóng và thuận tiện import các tài sản của ứng dụng:

```js
{
    '@' => '/resources/js'
}
```

Bạn có thể ghi đè alias `'@'` bằng cách thêm alias của riêng mình vào file cấu hình `vite.config.js`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### Vue

Nếu bạn muốn xây dựng frontend của mình bằng framework [Vue](https://vuejs.org/), thì bạn cũng sẽ cần cài đặt plugin `@vitejs/plugin-vue`:

```shell
npm install --save-dev @vitejs/plugin-vue
```

Sau đó, bạn có thể bao gồm plugin trong file cấu hình `vite.config.js` của mình. Có một số tùy chọn bổ sung bạn sẽ cần khi sử dụng plugin Vue với Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // Plugin Vue sẽ viết lại URL tài sản, khi được tham chiếu
                    // trong Single File Components, để trỏ đến máy chủ web
                    // Laravel. Đặt điều này thành `null` cho phép plugin Laravel
                    // thay vào đó viết lại URL tài sản để trỏ đến máy chủ
                    // Vite.
                    base: null,

                    // Plugin Vue sẽ phân tích cú pháp các URL tuyệt đối và coi chúng
                    // là đường dẫn tuyệt đối đến các file trên đĩa. Đặt điều này thành
                    // `false` sẽ để lại các URL tuyệt đối không bị thay đổi để chúng có thể
                    // tham chiếu tài sản trong thư mục public như mong đợi.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> [!NOTE]
> [Starter kits](/docs/{{version}}/starter-kits) của Laravel đã bao gồm cấu hình Laravel, Vue và Vite phù hợp. Các starter kits này cung cấp cách nhanh nhất để bắt đầu với Laravel, Vue và Vite.

<a name="react"></a>
### React

Nếu bạn muốn xây dựng frontend của mình bằng framework [React](https://reactjs.org/), thì bạn cũng sẽ cần cài đặt plugin `@vitejs/plugin-react`:

```shell
npm install --save-dev @vitejs/plugin-react
```

Sau đó, bạn có thể bao gồm plugin trong file cấu hình `vite.config.js` của mình:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

Bạn sẽ cần đảm bảo rằng bất kỳ file nào chứa JSX có phần mở rộng `.jsx` hoặc `.tsx`, nhớ cập nhật entry point của bạn nếu cần, như [được hiển thị ở trên](#configuring-vite).

Bạn cũng sẽ cần bao gồm directive Blade `@viteReactRefresh` bổ sung cùng với directive `@vite` hiện có của bạn.

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

Directive `@viteReactRefresh` phải được gọi trước directive `@vite`.

> [!NOTE]
> [Starter kits](/docs/{{version}}/starter-kits) của Laravel đã bao gồm cấu hình Laravel, React và Vite phù hợp. Các starter kits này cung cấp cách nhanh nhất để bắt đầu với Laravel, React và Vite.

<a name="svelte"></a>
### Svelte

Nếu bạn muốn xây dựng frontend của mình bằng framework [Svelte](https://svelte.dev/), thì bạn cũng sẽ cần cài đặt plugin `@sveltejs/vite-plugin-svelte`:

```shell
npm install --save-dev @sveltejs/vite-plugin-svelte
```

Sau đó, bạn có thể bao gồm plugin trong file cấu hình `vite.config.js` của mình.

```js
import { svelte } from '@sveltejs/vite-plugin-svelte';
import laravel from 'laravel-vite-plugin';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    laravel({
      input: ['resources/js/app.ts'],
      ssr: 'resources/js/ssr.ts',
      refresh: true,
    }),
    svelte(),
  ],
});
```

> [!NOTE]
> [Starter kits](/docs/{{version}}/starter-kits) của Laravel đã bao gồm cấu hình Laravel, Svelte và Vite phù hợp. Các starter kits này cung cấp cách nhanh nhất để bắt đầu với Laravel, Svelte và Vite.

<a name="inertia"></a>
### Inertia

Plugin Laravel Vite cung cấp hàm `resolvePageComponent` thuận tiện để giúp bạn giải quyết các thành phần trang Inertia của mình. Dưới đây là ví dụ về helper đang được sử dụng với Vue 3; tuy nhiên, bạn cũng có thể sử dụng hàm này trong các framework khác như React hoặc Svelte:

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

Nếu bạn đang sử dụng tính năng code splitting của Vite với Inertia, chúng tôi khuyên bạn nên cấu hình [asset prefetching](#asset-prefetching).

> [!NOTE]
> [Starter kits](/docs/{{version}}/starter-kits) của Laravel đã bao gồm cấu hình Laravel, Inertia và Vite phù hợp. Các starter kits này cung cấp cách nhanh nhất để bắt đầu với Laravel, Inertia và Vite.

<a name="url-processing"></a>
### Xử lý URL

Khi sử dụng Vite và tham chiếu tài sản trong HTML, CSS hoặc JS của ứng dụng, có một số lưu ý cần xem xét. Đầu tiên, nếu bạn tham chiếu tài sản với đường dẫn tuyệt đối, Vite sẽ không bao gồm tài sản trong build; do đó, bạn nên đảm bảo rằng tài sản có sẵn trong thư mục public của bạn. Bạn nên tránh sử dụng đường dẫn tuyệt đối khi sử dụng [CSS entrypoint chuyên dụng](#configuring-vite) vì, trong quá trình phát triển, trình duyệt sẽ cố gắng tải các đường dẫn này từ máy chủ phát triển Vite, nơi CSS được lưu trữ, thay vì từ thư mục public của bạn.

Khi tham chiếu đường dẫn tài sản tương đối, bạn nên nhớ rằng các đường dẫn này tương đối với file nơi chúng được tham chiếu. Bất kỳ tài sản nào được tham chiếu qua đường dẫn tương đối sẽ được viết lại, tạo phiên bản và đóng gói bởi Vite.

Xem xét cấu trúc dự án sau:

```text
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

Ví dụ sau đây minh họa cách Vite sẽ xử lý các URL tương đối và tuyệt đối:

```html
<!-- Tài sản này không được xử lý bởi Vite và sẽ không được bao gồm trong build -->
<img src="/taylor.png">

<!-- Tài sản này sẽ được viết lại, tạo phiên bản và đóng gói bởi Vite -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## Làm việc với Stylesheets

> [!NOTE]
> [Starter kits](/docs/{{version}}/starter-kits) của Laravel đã bao gồm cấu hình Tailwind và Vite phù hợp. Hoặc, nếu bạn muốn sử dụng Tailwind và Laravel mà không sử dụng một trong các starter kits của chúng tôi, hãy xem [hướng dẫn cài đặt Tailwind cho Laravel](https://tailwindcss.com/docs/guides/laravel).

Tất cả các ứng dụng Laravel đã bao gồm Tailwind và file `vite.config.js` được cấu hình đúng. Vì vậy, bạn chỉ cần bắt đầu máy chủ phát triển Vite hoặc chạy lệnh `dev` Composer, điều này sẽ bắt đầu cả máy chủ phát triển Laravel và Vite:

```shell
composer run dev
```

CSS của ứng dụng có thể được đặt trong file `resources/css/app.css`.

<a name="working-with-fonts"></a>
## Làm việc với Fonts

Plugin Laravel Vite có thể phục vụ các font được tối ưu hóa và tự lưu trữ cho ứng dụng của bạn. Khi các font được cấu hình, plugin giải quyết các file font được yêu cầu, phát ra chúng dưới dạng tài sản Vite, tạo CSS font và viết manifest font có thể được tiêu thụ bởi directive [`@fonts`](/docs/{{version}}/blade#fonts) của Blade.

Để cấu hình fonts, import một hoặc nhiều helper provider từ `laravel-vite-plugin/fonts` và thêm chúng vào tùy chọn `fonts` của plugin Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { google } from 'laravel-vite-plugin/fonts';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            fonts: [
                google('Inter', {
                    alias: 'sans',
                    weights: [400, 500, 600, 700],
                    styles: ['normal', 'italic'],
                    subsets: ['latin'],
                    display: 'swap',
                    preload: [
                        { weight: 400 },
                        { weight: 700 },
                    ],
                    fallbacks: ['system-ui', 'sans-serif'],
                }),
            ],
        }),
    ],
});
```

Trong ví dụ này, font `Inter` sẽ có sẵn thông qua alias `sans`. Plugin sẽ tạo biến CSS `--font-sans` và class tiện ích `.font-sans` áp dụng font stack được tạo.

<a name="font-providers"></a>
### Font Providers

Plugin Laravel Vite bao gồm các helper provider cho Google Fonts, Bunny Fonts, Fontsource và local fonts:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { bunny, fontsource, google, local } from 'laravel-vite-plugin/fonts';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            fonts: [
                google('Inter', { alias: 'sans' }),
                bunny('Figtree', { alias: 'body' }),
                fontsource('JetBrains Mono', { alias: 'mono' }),
                local('Brand Sans', {
                    alias: 'brand',
                    src: 'resources/fonts/brand-sans',
                }),
            ],
        }),
    ],
});
```

Provider `fontsource` đọc fonts từ một gói Fontsource đã cài đặt. Theo mặc định, tên gói được lấy từ font family, chẳng hạn như `@fontsource/jetbrains-mono`. Nếu ứng dụng của bạn sử dụng tên gói khác, bạn có thể chỉ định nó bằng tùy chọn `package`.

<a name="local-fonts"></a>
### Local Fonts

Khi sử dụng local fonts, tùy chọn `src` có thể trỏ đến một file font duy nhất, một thư mục hoặc một glob pattern. Plugin sẽ phát hiện các file font được hỗ trợ và suy luận weight và style của chúng từ tên file của chúng:

```js
local('Brand Sans', {
    alias: 'brand',
    src: 'resources/fonts/brand-sans/*.woff2',
})
```

Nếu bạn cần kiểm soát đầy đủ các biến thể có sẵn, bạn có thể xác định chúng một cách rõ ràng bằng tùy chọn `variants`:

```js
local('Brand Sans', {
    alias: 'brand',
    variants: [
        { src: 'resources/fonts/BrandSans-Regular.woff2', weight: 400 },
        { src: 'resources/fonts/BrandSans-Italic.woff2', weight: 400, style: 'italic' },
        { src: ['resources/fonts/BrandSans-Bold.woff2', 'resources/fonts/BrandSans-Bold.ttf'], weight: 700 },
    ],
})
```

<a name="font-options"></a>
### Font Options

Tùy thuộc vào provider, các định nghĩa font có thể chấp nhận một số tùy chọn cho phép bạn tùy chỉnh CSS font được tạo:

<div class="content-list" markdown="1">

- `alias` xác định tên được sử dụng bởi directive `@fonts` của Blade và mặc định là slug của font family.
- `variable` xác định biến CSS được tạo và mặc định là `--font-{alias}`.
- `weights` xác định các weight font từ xa hoặc Fontsource nên được giải quyết và mặc định là `[400]`.
- `styles` xác định các style font từ xa hoặc Fontsource nên được giải quyết và mặc định là `['normal']`.
- `subsets` xác định các subset font từ xa hoặc Fontsource nên được giải quyết và mặc định là `['latin']`.
- `display` xác định giá trị `font-display` và mặc định là `swap`.
- `preload` kiểm soát các biến thể font WOFF2 nên được preload. Tùy chọn này có thể là `true`, `false`, hoặc một mảng các bộ chọn `{ weight, style }`.
- `fallbacks` xác định các font fallback bổ sung nên được thêm vào font stack được tạo.
- `optimizedFallbacks` cố gắng tạo các font face fallback được điều chỉnh metric bằng cách sử dụng gói `fontaine` tùy chọn và mặc định là `true`.

</div>

Local fonts được giải quyết từ các tùy chọn `src` hoặc `variants` được mô tả ở trên thay vì sử dụng `weights`, `styles` và `subsets`.

<a name="working-with-blade-and-routes"></a>
## Làm việc với Blade và Routes

<a name="blade-processing-static-assets"></a>
### Xử lý Static Assets với Vite

Khi tham chiếu tài sản trong JavaScript hoặc CSS của bạn, Vite tự động xử lý và tạo phiên bản cho chúng. Ngoài ra, khi xây dựng các ứng dụng dựa trên Blade, Vite cũng có thể xử lý và tạo phiên bản cho các tài sản tĩnh mà bạn chỉ tham chiếu trong các template Blade.

Tuy nhiên, để thực hiện điều này, bạn cần làm cho Vite nhận biết về tài sản của mình bằng cách chỉ định chúng trong tùy chọn `assets` của plugin. Tùy chọn này dành cho các file tĩnh mà bạn muốn tham chiếu trực tiếp với `Vite::asset`. Nếu bạn muốn Laravel tạo CSS font và các liên kết preload, hãy sử dụng tùy chọn [`fonts`](#working-with-fonts) thay thế.

Ví dụ, nếu bạn muốn xử lý và tạo phiên bản cho tất cả hình ảnh được lưu trữ trong `resources/images` và tất cả fonts được lưu trữ trong `resources/fonts`, bạn nên thêm nội dung sau vào cấu hình Vite của mình:

```js
laravel({
    input: 'resources/js/app.js',
    assets: ['resources/images/**', 'resources/fonts/**'],
})
```

Các tài sản này sẽ được xử lý bởi Vite khi chạy `npm run build`. Sau đó, bạn có thể tham chiếu các tài sản này trong các template Blade bằng phương thức `Vite::asset`, phương thức này sẽ trả về URL có phiên bản cho một tài sản đã cho:

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

> [!NOTE]
> Trước phiên bản 3 của plugin Laravel Vite, các tài sản tĩnh phải được import trong entry point của ứng dụng của bạn bằng cách sử dụng `import.meta.glob`. Tùy chọn `assets` được giới thiệu do các thay đổi trong Vite 8.

<a name="blade-refreshing-on-save"></a>
### Làm mới khi Lưu

Khi ứng dụng của bạn được xây dựng bằng cách sử dụng server-side rendering truyền thống với Blade, Vite có thể cải thiện quy trình phát triển của bạn bằng cách tự động làm mới trình duyệt khi bạn thực hiện thay đổi đối với các file view trong ứng dụng của mình. Để bắt đầu, bạn có thể chỉ định tùy chọn `refresh` là `true`.

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

Khi tùy chọn `refresh` là `true`, việc lưu file trong các thư mục sau sẽ kích hoạt trình duyệt thực hiện làm mới trang đầy đủ trong khi bạn đang chạy `npm run dev`:

- `app/Livewire/**`
- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

Việc theo dõi thư mục `routes/**` rất hữu ích nếu bạn đang sử dụng [Ziggy](https://github.com/tighten/ziggy) để tạo các liên kết route trong frontend của ứng dụng của bạn.

Nếu các đường dẫn mặc định này không phù hợp với nhu cầu của bạn, bạn có thể chỉ định danh sách đường dẫn của riêng mình để theo dõi:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
```

Dưới bề mặt, plugin Laravel Vite sử dụng gói [vite-plugin-full-reload](https://github.com/ElMassimo/vite-plugin-full-reload), gói này cung cấp một số tùy chọn cấu hình nâng cao để tinh chỉnh hành vi của tính năng này. Nếu bạn cần mức độ tùy chỉnh này, bạn có thể cung cấp định nghĩa `config`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
```

<a name="blade-aliases"></a>
### Aliases

Thật phổ biến trong các ứng dụng JavaScript để [tạo aliases](#aliases) cho các thư mục được tham chiếu thường xuyên. Nhưng, bạn cũng có thể tạo aliases để sử dụng trong Blade bằng cách sử dụng phương thức `macro` trên class `Illuminate\Support\Facades\Vite`. Thông thường, "macros" nên được xác định trong phương thức `boot` của một [service provider](/docs/{{version}}/providers):

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
}
```

Khi một macro đã được xác định, nó có thể được gọi trong các template của bạn. Ví dụ, chúng ta có thể sử dụng macro `image` được xác định ở trên để tham chiếu một tài sản nằm tại `resources/images/logo.png`:

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="asset-prefetching"></a>
## Asset Prefetching

Khi xây dựng một SPA bằng tính năng code splitting của Vite, các tài sản cần thiết được tìm nạp trên mỗi điều hướng trang. Hành vi này có thể dẫn đến việc hiển thị UI bị trì hoãn. Nếu đây là vấn đề đối với framework frontend của bạn, Laravel cung cấp khả năng prefetch eagerly các tài sản JavaScript và CSS của ứng dụng khi tải trang ban đầu.

Bạn có thể hướng dẫn Laravel eagerly prefetch các tài sản của mình bằng cách gọi phương thức `Vite::prefetch` trong phương thức `boot` của một [service provider](/docs/{{version}}/providers):

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Vite;
use Illuminate\Support\ServiceProvider;

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
        Vite::prefetch(concurrency: 3);
    }
}
```

Trong ví dụ trên, các tài sản sẽ được prefetch với tối đa `3` tải xuống đồng thời trên mỗi lần tải trang. Bạn có thể sửa đổi concurrency để phù hợp với nhu cầu của ứng dụng hoặc chỉ định không có giới hạn concurrency nếu ứng dụng nên tải xuống tất cả tài sản cùng một lúc:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::prefetch();
}
```

Theo mặc định, prefetching sẽ bắt đầu khi sự kiện [page _load_](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event) kích hoạt. Nếu bạn muốn tùy chỉnh khi prefetching bắt đầu, bạn có thể chỉ định một sự kiện mà Vite sẽ lắng nghe:

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::prefetch(event: 'vite:prefetch');
}
```

Với mã ở trên, prefetching bây giờ sẽ bắt đầu khi bạn manually dispatch sự kiện `vite:prefetch` trên đối tượng `window`. Ví dụ, bạn có thể để prefetching bắt đầu ba giây sau khi trang tải:

```html
<script>
    addEventListener('load', () => setTimeout(() => {
        dispatchEvent(new Event('vite:prefetch'))
    }, 3000))
</script>
```

<a name="custom-base-urls"></a>
## Custom Base URLs

Nếu các tài sản đã biên dịch Vite của bạn được triển khai đến một domain riêng biệt với ứng dụng của bạn, chẳng hạn như thông qua CDN, bạn phải chỉ định biến môi trường `ASSET_URL` trong file `.env` của ứng dụng của bạn:

```env
ASSET_URL=https://cdn.example.com
```

Sau khi cấu hình URL tài sản, tất cả các URL được viết lại cho tài sản của bạn sẽ được tiền tố với giá trị được cấu hình:

```text
https://cdn.example.com/build/assets/app.9dce8d17.js
```

Hãy nhớ rằng [URL tuyệt đối không được viết lại bởi Vite](#url-processing), vì vậy chúng sẽ không được tiền tố.

<a name="environment-variables"></a>
## Biến môi trường

Bạn có thể inject các biến môi trường vào JavaScript của mình bằng cách tiền tố chúng với `VITE_` trong file `.env` của ứng dụng của bạn:

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```
Bạn có thể truy cập các biến môi trường được inject thông qua đối tượng `import.meta.env`:

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## Tắt Vite trong Tests

Tích hợp Vite của Laravel sẽ cố gắng giải quyết các tài sản (assets) của bạn trong khi chạy tests, điều này yêu cầu bạn phải chạy Vite development server hoặc build các tài sản của mình.

Nếu bạn muốn mock Vite trong quá trình test, bạn có thể gọi phương thức `withoutVite`, phương thức này có sẵn cho bất kỳ test nào mở rộng lớp `TestCase` của Laravel:

```php tab=Pest
test('without vite example', function () {
    $this->withoutVite();

    // ...
});
```

```php tab=PHPUnit
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_vite_example(): void
    {
        $this->withoutVite();

        // ...
    }
}
```

Nếu bạn muốn tắt Vite cho tất cả các test, bạn có thể gọi phương thức `withoutVite` từ phương thức `setUp` trên lớp `TestCase` cơ sở của mình:

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## Server-Side Rendering (SSR)

Plugin Laravel Vite giúp việc thiết lập server-side rendering với Vite trở nên dễ dàng. Để bắt đầu, hãy tạo một SSR entry point tại `resources/js/ssr.js` và chỉ định entry point bằng cách truyền một tùy chọn cấu hình cho plugin Laravel:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

Để đảm bảo bạn không quên rebuild SSR entry point, chúng tôi khuyên bạn nên bổ sung script "build" trong `package.json` của ứng dụng để tạo SSR build của mình:

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

Sau đó, để build và khởi động SSR server, bạn có thể chạy các lệnh sau:

```shell
npm run build
node bootstrap/ssr/ssr.js
```

Nếu bạn đang sử dụng [SSR với Inertia](https://inertiajs.com/server-side-rendering), bạn có thể thay vào đó sử dụng lệnh Artisan `inertia:start-ssr` để khởi động SSR server:

```shell
php artisan inertia:start-ssr
```

> [!NOTE]
> Các [starter kits](/docs/{{version}}/starter-kits) của Laravel đã bao gồm cấu hình Laravel, Inertia SSR và Vite phù hợp. Các starter kits này cung cấp cách nhanh nhất để bắt đầu với Laravel, Inertia SSR và Vite.

<a name="script-and-style-attributes"></a>
## Script và Style Tag Attributes

<a name="content-security-policy-csp-nonce"></a>
### Content Security Policy (CSP) Nonce

Nếu bạn muốn bao gồm một [nonce attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) trên các thẻ script và style của mình như một phần của [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP), bạn có thể tạo hoặc chỉ định một nonce bằng cách sử dụng phương thức `useCspNonce` trong một [middleware](/docs/{{version}}/middleware) tùy chỉnh:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Vite;
use Symfony\Component\HttpFoundation\Response;

class AddContentSecurityPolicyHeaders
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
```

Sau khi gọi phương thức `useCspNonce`, Laravel sẽ tự động bao gồm các thuộc tính `nonce` trên tất cả các thẻ script và style được tạo.

Nếu bạn cần chỉ định nonce ở nơi khác, bao gồm cả [directive `@route` của Ziggy](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy) được bao gồm trong các [starter kits](/docs/{{version}}/starter-kits) của Laravel, bạn có thể lấy nó bằng phương thức `cspNonce`:

```blade
@routes(nonce: Vite::cspNonce())
```

Nếu bạn đã có một nonce mà bạn muốn hướng dẫn Laravel sử dụng, bạn có thể truyền nonce cho phương thức `useCspNonce`:

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### Subresource Integrity (SRI)

Nếu manifest Vite của bạn bao gồm các hash `integrity` cho các tài sản của mình, Laravel sẽ tự động thêm thuộc tính `integrity` trên bất kỳ thẻ script và style nào mà nó tạo ra để thực thi [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity). Theo mặc định, Vite không bao gồm hash `integrity` trong manifest của nó, nhưng bạn có thể bật nó bằng cách cài đặt plugin NPM [vite-plugin-manifest-sri](https://www.npmjs.com/package/vite-plugin-manifest-sri):

```shell
npm install --save-dev vite-plugin-manifest-sri
```

Sau đó bạn có thể bật plugin này trong file `vite.config.js` của mình:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```

Nếu cần thiết, bạn cũng có thể tùy chỉnh key manifest nơi có thể tìm thấy hash integrity:

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

Nếu bạn muốn tắt tính năng tự động phát hiện này hoàn toàn, bạn có thể truyền `false` cho phương thức `useIntegrityKey`:

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### Arbitrary Attributes

Nếu bạn cần bao gồm các thuộc tính bổ sung trên các thẻ script và style của mình, chẳng hạn như thuộc tính [data-turbo-track](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change), bạn có thể chỉ định chúng thông qua các phương thức `useScriptTagAttributes` và `useStyleTagAttributes`. Thông thường, các phương thức này nên được gọi từ một [service provider](/docs/{{version}}/providers):

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // Chỉ định giá trị cho thuộc tính...
    'async' => true, // Chỉ định một thuộc tính không có giá trị...
    'integrity' => false, // Loại bỏ một thuộc tính nếu không thì sẽ được bao gồm...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

Nếu bạn cần thêm thuộc tính có điều kiện, bạn có thể truyền một callback sẽ nhận đường dẫn nguồn tài sản, URL của nó, chunk manifest của nó, và toàn bộ manifest:

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> [!WARNING]
> Các đối số `$chunk` và `$manifest` sẽ là `null` trong khi Vite development server đang chạy.

<a name="advanced-customization"></a>
## Tùy Chỉnh Nâng Cao

Theo mặc định, plugin Vite của Laravel sử dụng các quy ước hợp lý nên hoạt động với phần lớn các ứng dụng; tuy nhiên, đôi khi bạn có thể cần tùy chỉnh hành vi của Vite. Để bật các tùy chọn tùy chỉnh bổ sung, chúng tôi cung cấp các phương thức và tùy chọn sau có thể được sử dụng thay cho directive Blade `@vite`:

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // Tùy chỉnh file "hot"...
            ->useBuildDirectory('bundle') // Tùy chỉnh thư mục build...
            ->useManifestFilename('assets.json') // Tùy chỉnh tên file manifest...
            ->withEntryPoints(['resources/js/app.js']) // Chỉ định các entry points...
            ->createAssetPathsUsing(function (string $path, ?bool $secure) { // Tùy chỉnh việc tạo đường dẫn backend cho các tài sản đã build...
                return "https://cdn.example.com/{$path}";
            })
    }}
</head>
```

Trong file `vite.config.js`, bạn sau đó nên chỉ định cùng cấu hình:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // Tùy chỉnh file "hot"...
            buildDirectory: 'bundle', // Tùy chỉnh thư mục build...
            input: ['resources/js/app.js'], // Chỉ định các entry points...
        }),
    ],
    build: {
      manifest: 'assets.json', // Tùy chỉnh tên file manifest...
    },
});
```

<a name="cors"></a>
### Dev Server Cross-Origin Resource Sharing (CORS)

Nếu bạn đang gặp vấn đề Cross-Origin Resource Sharing (CORS) trong trình duyệt trong khi lấy tài sản từ Vite dev server, bạn có thể cần cấp quyền truy cập origin tùy chỉnh cho dev server. Vite kết hợp với plugin Laravel cho phép các origin sau mà không cần cấu hình bổ sung:

- `::1`
- `127.0.0.1`
- `localhost`
- `*.test`
- `*.localhost`
- `APP_URL` trong `.env` của dự án

Cách dễ nhất để cho phép một origin tùy chỉnh cho dự án của bạn là đảm bảo rằng biến môi trường `APP_URL` của ứng dụng khớp với origin bạn đang truy cập trong trình duyệt. Ví dụ, nếu bạn đang truy cập `https://my-app.laravel`, bạn nên cập nhật `.env` của mình để khớp:

```env
APP_URL=https://my-app.laravel
```

Nếu bạn cần kiểm soát chi tiết hơn về các origin, chẳng hạn như hỗ trợ nhiều origin, bạn nên sử dụng [cấu hình CORS server tích hợp toàn diện và linh hoạt của Vite](https://vite.dev/config/server-options.html#server-cors). Ví dụ, bạn có thể chỉ định nhiều origin trong tùy chọn cấu hình `server.cors.origin` trong file `vite.config.js` của dự án:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
    ],
    server: {  // [tl! add]
        cors: {  // [tl! add]
            origin: [  // [tl! add]
                'https://backend.laravel',  // [tl! add]
                'http://admin.laravel:8566',  // [tl! add]
            ],  // [tl! add]
        },  // [tl! add]
    },  // [tl! add]
});
```

Bạn cũng có thể bao gồm các mẫu regex, điều này có thể hữu ích nếu bạn muốn cho phép tất cả các origin cho một tên miền cấp cao nhất nhất định, chẳng hạn như `*.laravel`:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
    ],
    server: {  // [tl! add]
        cors: {  // [tl! add]
            origin: [ // [tl! add]
                // Hỗ trợ: SCHEME://DOMAIN.laravel[:PORT] [tl! add]
                /^https?:\/\/.*\.laravel(:\d+)?$/, //[tl! add]
            ], // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

<a name="correcting-dev-server-urls"></a>
### Sửa Đổi Dev Server URLs

Một số plugin trong hệ sinh thái Vite giả định rằng các URL bắt đầu bằng dấu gạch chéo xuôi sẽ luôn trỏ đến Vite dev server. Tuy nhiên, do bản chất của tích hợp Laravel, điều này không đúng.

Ví dụ, plugin `vite-imagetools` xuất các URL như sau trong khi Vite đang phục vụ các tài sản của bạn:

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

Plugin `vite-imagetools` mong đợi rằng URL đầu ra sẽ được Vite chặn và plugin sau đó có thể xử lý tất cả các URL bắt đầu bằng `/@imagetools`. Nếu bạn đang sử dụng các plugin mong đợi hành vi này, bạn sẽ cần sửa đổi thủ công các URL. Bạn có thể làm điều này trong file `vite.config.js` của mình bằng cách sử dụng tùy chọn `transformOnServe`.

Trong ví dụ cụ thể này, chúng tôi sẽ thêm tiền tố dev server URL vào tất cả các lần xuất hiện của `/@imagetools` trong mã được tạo:

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll('/@imagetools', devServerUrl+'/@imagetools'),
        }),
        imagetools(),
    ],
});
```

Bây giờ, trong khi Vite đang phục vụ Assets, nó sẽ xuất các URL trỏ đến Vite dev server:

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```
