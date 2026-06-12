# Starter Kits

- [Introduction](#introduction)
- [Creating an Application Using a Starter Kit](#creating-an-application)
- [Available Starter Kits](#available-starter-kits)
    - [React](#react)
    - [Svelte](#svelte)
    - [Vue](#vue)
    - [Livewire](#livewire)
- [Starter Kit Customization](#starter-kit-customization)
    - [React](#react-customization)
    - [Svelte](#svelte-customization)
    - [Vue](#vue-customization)
    - [Livewire](#livewire-customization)
- [Authentication](#authentication)
    - [Enabling and Disabling Features](#enabling-and-disabling-features)
    - [Customizing User Creation and Password Reset](#customizing-actions)
    - [Two-Factor Authentication](#two-factor-authentication)
    - [Rate Limiting](#rate-limiting)
- [Teams](#teams)
- [WorkOS AuthKit Authentication](#workos)
- [Inertia SSR](#inertia-ssr)
- [Community Maintained Starter Kits](#community-maintained-starter-kits)
- [Frequently Asked Questions](#faqs)

<a name="introduction"></a>
## Introduction

Để giúp bạn có một khởi đầu thuận lợi khi xây dựng ứng dụng Laravel mới của mình, chúng tôi rất vui được cung cấp [application starter kits](https://laravel.com/starter-kits). Các starter kits này giúp bạn có một khởi đầu thuận lợi khi xây dựng ứng dụng Laravel tiếp theo của bạn, và bao gồm các routes, controllers, và views bạn cần để đăng ký và xác thực người dùng của ứng dụng. Các starter kits sử dụng [Laravel Fortify](/docs/{{version}}/fortify) để cung cấp xác thực.

Trong khi bạn được chào đón sử dụng các starter kits này, chúng không được yêu cầu. Bạn tự do xây dựng ứng dụng của riêng mình từ đầu bằng cách chỉ cần cài đặt một bản sao mới của Laravel. Dù theo cách nào, chúng tôi biết bạn sẽ xây dựng điều gì tuyệt vời!

<a name="creating-an-application"></a>
## Creating an Application Using a Starter Kit

Để tạo một ứng dụng Laravel mới bằng cách sử dụng một trong các starter kits của chúng tôi, trước tiên bạn nên [cài đặt PHP và công cụ CLI Laravel](/docs/{{version}}/installation#installing-php). Nếu bạn đã có PHP và Composer được cài đặt, bạn có thể cài đặt công cụ CLI Laravel installer thông qua Composer:

```shell
composer global require laravel/installer
```

Sau đó, tạo một ứng dụng Laravel mới bằng cách sử dụng công cụ CLI Laravel installer. Laravel installer sẽ nhắc bạn chọn starter kit ưa thích của bạn:

```shell
laravel new my-app
```

Sau khi tạo ứng dụng Laravel của bạn, bạn chỉ cần cài đặt các dependencies frontend của nó thông qua NPM và bắt đầu server phát triển Laravel:

```shell
cd my-app
npm install && npm run build
composer run dev
```

Sau khi bạn đã bắt đầu server phát triển Laravel, ứng dụng của bạn sẽ có thể truy cập trong trình duyệt web của bạn tại [http://localhost:8000](http://localhost:8000).

<a name="available-starter-kits"></a>
## Available Starter Kits

<a name="react"></a>
### React

Starter kit React của chúng tôi cung cấp một điểm khởi đầu mạnh mẽ, hiện đại để xây dựng các ứng dụng Laravel với một frontend React sử dụng [Inertia](https://inertiajs.com).

Inertia cho phép bạn xây dựng các ứng dụng React single-page hiện đại bằng cách sử dụng routing và controllers phía server cổ điển. Điều này cho phép bạn tận dụng sức mạnh frontend của React kết hợp với năng suất backend đáng kinh ngạc của Laravel và biên dịch Vite cực nhanh.

Starter kit React sử dụng React 19, TypeScript, Tailwind, và thư viện component [shadcn/ui](https://ui.shadcn.com).

<a name="svelte"></a>
### Svelte

Starter kit Svelte của chúng tôi cung cấp một điểm khởi đầu mạnh mẽ, hiện đại để xây dựng các ứng dụng Laravel với một frontend Svelte sử dụng [Inertia](https://inertiajs.com).

Inertia cho phép bạn xây dựng các ứng dụng Svelte single-page hiện đại bằng cách sử dụng routing và controllers phía server cổ điển. Điều này cho phép bạn tận hưởng sức mạnh frontend của Svelte kết hợp với năng suất backend đáng kinh ngạc của Laravel và biên dịch Vite cực nhanh.

Starter kit Svelte sử dụng Svelte 5, TypeScript, Tailwind, và thư viện component [shadcn-svelte](https://www.shadcn-svelte.com/).

<a name="vue"></a>
### Vue

Starter kit Vue của chúng tôi cung cấp một điểm khởi đầu tuyệt vời để xây dựng các ứng dụng Laravel với một frontend Vue sử dụng [Inertia](https://inertiajs.com).

Inertia cho phép bạn xây dựng các ứng dụng Vue single-page hiện đại bằng cách sử dụng routing và controllers phía server cổ điển. Điều này cho phép bạn tận hưởng sức mạnh frontend của Vue kết hợp với năng suất backend đáng kinh ngạc của Laravel và biên dịch Vite cực nhanh.

Starter kit Vue sử dụng Vue Composition API, TypeScript, Tailwind, và thư viện component [shadcn-vue](https://www.shadcn-vue.com/).

<a name="livewire"></a>
### Livewire

Starter kit Livewire của chúng tôi cung cấp điểm khởi đầu hoàn hảo để xây dựng các ứng dụng Laravel với một frontend [Laravel Livewire](https://livewire.laravel.com).

Livewire là một cách mạnh mẽ để xây dựng các UI frontend động, reactive chỉ bằng cách sử dụng PHP. Nó là một lựa chọn tuyệt vời cho các teams chủ yếu sử dụng các templates Blade và đang tìm kiếm một giải pháp thay thế đơn giản hơn cho các frameworks SPA được điều khiển bởi JavaScript như React, Svelte, và Vue.

Starter kit Livewire sử dụng Livewire, Tailwind, và thư viện component [Flux UI](https://fluxui.dev).

<a name="starter-kit-customization"></a>
## Starter Kit Customization

<a name="react-customization"></a>
### React

Starter kit React của chúng tôi được xây dựng với Inertia 3, React 19, Tailwind 4, và [shadcn/ui](https://ui.shadcn.com). Như với tất cả các starter kits của chúng tôi, tất cả mã backend và frontend tồn tại trong ứng dụng của bạn để cho phép tùy chỉnh đầy đủ.

Phần lớn mã frontend nằm trong thư mục `resources/js`. Bạn tự do sửa đổi bất kỳ mã nào để tùy chỉnh giao diện và hành vi của ứng dụng:

```text
resources/js/
├── components/    # Reusable React components
├── hooks/         # React hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

Để xuất bản các components shadcn bổ sung, trước tiên [tìm component bạn muốn xuất bản](https://ui.shadcn.com). Sau đó, xuất bản component bằng cách sử dụng `npx`:

```shell
npx shadcn@latest add switch
```

Trong ví dụ này, lệnh sẽ xuất bản component Switch vào `resources/js/components/ui/switch.tsx`. Sau khi component đã được xuất bản, bạn có thể sử dụng nó trong bất kỳ trang nào của bạn:

```jsx
import { Switch } from "@/components/ui/switch"

const MyPage = () => {
  return (
    <div>
      <Switch />
    </div>
  );
};

export default MyPage;
```

<a name="react-available-layouts"></a>
#### Available Layouts

Starter kit React bao gồm hai layouts chính khác nhau để bạn chọn: một layout "sidebar" và một layout "header". Layout sidebar là mặc định, nhưng bạn có thể chuyển sang layout header bằng cách sửa đổi layout được nhập ở đầu file `resources/js/layouts/app-layout.tsx` của ứng dụng:

```js
import AppLayoutTemplate from '@/layouts/app/app-sidebar-layout'; // [tl! remove]
import AppLayoutTemplate from '@/layouts/app/app-header-layout'; // [tl! add]
```

<a name="react-sidebar-variants"></a>
#### Sidebar Variants

Layout sidebar bao gồm ba biến thể khác nhau: biến thể sidebar mặc định, biến thể "inset", và biến thể "floating". Bạn có thể chọn biến thể bạn thích nhất bằng cách sửa đổi component `resources/js/components/app-sidebar.tsx`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="react-authentication-page-layout-variants"></a>
#### Authentication Page Layout Variants

Các trang xác thực được bao gồm với starter kit React, chẳng hạn như trang đăng nhập và trang đăng ký, cũng cung cấp ba biến thể layout khác nhau: "simple", "card", và "split".

Để thay đổi layout xác thực của bạn, sửa đổi layout được nhập ở đầu file `resources/js/layouts/auth-layout.tsx` của ứng dụng:

```js
import AuthLayoutTemplate from '@/layouts/auth/auth-simple-layout'; // [tl! remove]
import AuthLayoutTemplate from '@/layouts/auth/auth-split-layout'; // [tl! add]
```

<a name="svelte-customization"></a>
### Svelte

Starter kit Svelte của chúng tôi được xây dựng với Inertia 3, Svelte 5, Tailwind, và [shadcn-svelte](https://www.shadcn-svelte.com/). Như với tất cả các starter kits của chúng tôi, tất cả mã backend và frontend tồn tại trong ứng dụng của bạn để cho phép tùy chỉnh đầy đủ.

Phần lớn mã frontend nằm trong thư mục `resources/js`. Bạn tự do sửa đổi bất kỳ mã nào để tùy chỉnh giao diện và hành vi của ứng dụng:

```text
resources/js/
├── components/    # Reusable Svelte components
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration and Svelte rune modules
├── pages/         # Page components
└── types/         # TypeScript definitions
```

Để xuất bản các components shadcn-svelte bổ sung, trước hết [tìm component bạn muốn xuất bản](https://www.shadcn-svelte.com). Sau đó, xuất bản component bằng cách sử dụng `npx`:

```shell
npx shadcn-svelte@latest add switch
```

Trong ví dụ này, lệnh sẽ xuất bản component Switch vào `resources/js/components/ui/switch/switch.svelte`. Sau khi component đã được xuất bản, bạn có thể sử dụng nó trong bất kỳ trang nào của bạn:

```svelte
<script lang="ts">
    import { Switch } from '@/components/ui/switch'
</script>

<div>
    <Switch />
</div>
```

<a name="svelte-available-layouts"></a>
#### Available Layouts

Starter kit Svelte bao gồm hai layouts chính khác nhau để bạn chọn: một layout "sidebar" và một layout "header". Layout sidebar là mặc định, nhưng bạn có thể chuyển sang layout header bằng cách sửa đổi layout được nhập ở đầu file `resources/js/layouts/AppLayout.svelte` của ứng dụng:

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.svelte'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.svelte'; // [tl! add]
```

<a name="svelte-sidebar-variants"></a>
#### Sidebar Variants

Layout sidebar bao gồm ba biến thể khác nhau: biến thể sidebar mặc định, biến thể "inset", và biến thể "floating". Bạn có thể chọn biến thể bạn thích nhất bằng cách sửa đổi component `resources/js/components/AppSidebar.svelte`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="svelte-authentication-page-layout-variants"></a>
#### Authentication Page Layout Variants

Các trang xác thực được bao gồm với starter kit Svelte, chẳng hạn như trang đăng nhập và trang đăng ký, cũng cung cấp ba biến thể layout khác nhau: "simple", "card", và "split".

Để thay đổi layout xác thực của bạn, sửa đổi layout được nhập ở đầu file `resources/js/layouts/AuthLayout.svelte` của ứng dụng:

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.svelte'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.svelte'; // [tl! add]
```

<a name="vue-customization"></a>
### Vue

Starter kit Vue của chúng tôi được xây dựng với Inertia 3, Vue 3 Composition API, Tailwind, và [shadcn-vue](https://www.shadcn-vue.com/). Như với tất cả các starter kits của chúng tôi, tất cả mã backend và frontend tồn tại trong ứng dụng của bạn để cho phép tùy chỉnh đầy đủ.

Phần lớn mã frontend nằm trong thư mục `resources/js`. Bạn tự do sửa đổi bất kỳ mã nào để tùy chỉnh giao diện và hành vi của ứng dụng:

```text
resources/js/
├── components/    # Reusable Vue components
├── composables/   # Vue composables / hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

Để xuất bản các components shadcn-vue bổ sung, trước hết [tìm component bạn muốn xuất bản](https://www.shadcn-vue.com). Sau đó, xuất bản component bằng cách sử dụng `npx`:

```shell
npx shadcn-vue@latest add switch
```

Trong ví dụ này, lệnh sẽ xuất bản component Switch vào `resources/js/components/ui/Switch.vue`. Sau khi component đã được xuất bản, bạn có thể sử dụng nó trong bất kỳ trang nào của bạn:

```vue
<script setup lang="ts">
import { Switch } from '@/components/ui/switch'
</script>

<template>
    <div>
        <Switch />
    </div>
</template>
```

<a name="vue-available-layouts"></a>
#### Available Layouts

Starter kit Vue bao gồm hai layouts chính khác nhau để bạn chọn: một layout "sidebar" và một layout "header". Layout sidebar là mặc định, nhưng bạn có thể chuyển sang layout header bằng cách sửa đổi layout được nhập ở đầu file `resources/js/layouts/AppLayout.vue` của ứng dụng:

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.vue'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.vue'; // [tl! add]
```

<a name="vue-sidebar-variants"></a>
#### Sidebar Variants

Layout sidebar bao gồm ba biến thể khác nhau: biến thể sidebar mặc định, biến thể "inset", và biến thể "floating". Bạn có thể chọn biến thể bạn thích nhất bằng cách sửa đổi component `resources/js/components/AppSidebar.vue`:

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="vue-authentication-page-layout-variants"></a>
#### Authentication Page Layout Variants

Các trang xác thực được bao gồm với starter kit Vue, chẳng hạn như trang đăng nhập và trang đăng ký, cũng cung cấp ba biến thể layout khác nhau: "simple", "card", và "split".

Để thay đổi layout xác thực của bạn, sửa đổi layout được nhập ở đầu file `resources/js/layouts/AuthLayout.vue` của ứng dụng:

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.vue'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.vue'; // [tl! add]
```

<a name="livewire-customization"></a>
### Livewire

Starter kit Livewire của chúng tôi được xây dựng với Livewire 4, Tailwind, và [Flux UI](https://fluxui.dev/). Như với tất cả các starter kits của chúng tôi, tất cả mã backend và frontend tồn tại trong ứng dụng của bạn để cho phép tùy chỉnh đầy đủ.

Phần lớn mã frontend nằm trong thư mục `resources/views`. Bạn tự do sửa đổi bất kỳ mã nào để tùy chỉnh giao diện và hành vi của ứng dụng:

```text
resources/views
├── components            # Reusable components
├── flux                  # Customized Flux components
├── layouts               # Application layouts
├── pages                 # Livewire pages
├── partials              # Reusable Blade partials
├── dashboard.blade.php   # Authenticated user dashboard
├── welcome.blade.php     # Guest user welcome page
```

<a name="livewire-available-layouts"></a>
#### Available Layouts

Starter kit Livewire bao gồm hai layouts chính khác nhau để bạn chọn: một layout "sidebar" và một layout "header". Layout sidebar là mặc định, nhưng bạn có thể chuyển sang layout header bằng cách sửa đổi layout được sử dụng bởi file `resources/views/layouts/app.blade.php` của ứng dụng. Ngoài ra, bạn nên thêm attribute `container` vào component Flux chính:

```blade
<x-layouts::app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts::app.header>
```

<a name="livewire-authentication-page-layout-variants"></a>
#### Authentication Page Layout Variants

Các trang xác thực được bao gồm với starter kit Livewire, chẳng hạn như trang đăng nhập và trang đăng ký, cũng cung cấp ba biến thể layout khác nhau: "simple", "card", và "split".

Để thay đổi layout xác thực của bạn, sửa đổi layout được sử dụng bởi file `resources/views/layouts/auth.blade.php` của ứng dụng:

```blade
<x-layouts::auth.split>
    {{ $slot }}
</x-layouts::auth.split>
```

<a name="authentication"></a>
## Authentication

Tất cả các starter kits sử dụng [Laravel Fortify](/docs/{{version}}/fortify) để xử lý xác thực. Fortify cung cấp routes, controllers, và logic cho đăng nhập, đăng ký, reset mật khẩu, xác thực email, và hơn thế nữa.

Fortify tự động đăng ký các routes xác thực sau dựa trên các tính năng được bật trong file cấu hình `config/fortify.php` của ứng dụng:

|| Route                              | Method | Description                         |
|| ---------------------------------- | ------ | ----------------------------------- |
|| `/login`                           | `GET`    | Display login form                  |
|| `/login`                           | `POST`   | Authenticate user                   |
|| `/logout`                          | `POST`   | Log user out                        |
|| `/register`                        | `GET`    | Display registration form           |
|| `/register`                        | `POST`   | Create new user                     |
|| `/forgot-password`                 | `GET`    | Display password reset request form |
|| `/forgot-password`                 | `POST`   | Send password reset link            |
|| `/reset-password/{token}`          | `GET`    | Display password reset form         |
|| `/reset-password`                  | `POST`   | Update password                     |
|| `/email/verify`                    | `GET`    | Display email verification notice   |
|| `/email/verify/{id}/{hash}`        | `GET`    | Verify email address                |
|| `/email/verification-notification` | `POST`   | Resend verification email           |
|| `/user/confirm-password`           | `GET`    | Display password confirmation form  |
|| `/user/confirm-password`           | `POST`   | Confirm password                    |
|| `/two-factor-challenge`            | `GET`    | Display 2FA challenge form          |
|| `/two-factor-challenge`            | `POST`   | Verify 2FA code                     |

Lệnh Artisan `php artisan route:list` có thể được sử dụng để hiển thị tất cả các routes trong ứng dụng của bạn.

<a name="enabling-and-disabling-features"></a>
### Enabling and Disabling Features

Bạn có thể kiểm soát các tính năng Fortify nào được bật trong file cấu hình `config/fortify.php` của ứng dụng:

```php
use Laravel\Fortify\Features;

'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

Để tắt một tính năng, comment out hoặc xóa mục nhập tính năng đó khỏi array `features`. Ví dụ, xóa `Features::registration()` để tắt đăng ký công khai.

Khi sử dụng các starter kits [React](#react), [Svelte](#svelte) hoặc [Vue](#vue), bạn cũng sẽ cần xóa bất kỳ tham chiếu nào đến routes của tính năng bị tắt trong mã frontend của bạn. Ví dụ, nếu bạn tắt xác thực email, bạn nên xóa các imports và tham chiếu đến các routes `verification` trong các components React, Svelte, hoặc Vue của bạn. Điều này là cần thiết vì các starter kits này sử dụng Wayfinder cho routing type-safe, tạo ra các định nghĩa route tại thời điểm build. Nếu bạn tham chiếu các routes không còn tồn tại, ứng dụng của bạn sẽ không thể build.

<a name="customizing-actions"></a>
### Customizing User Creation and Password Reset

Khi một người dùng đăng ký hoặc reset mật khẩu của họ, Fortify gọi các lớp action nằm trong thư mục `app/Actions/Fortify` của ứng dụng:

|| File                          | Description                           |
|| ----------------------------- | ------------------------------------- |
|| `CreateNewUser.php`           | Validates and creates new users       |
|| `ResetUserPassword.php`       | Validates and updates user passwords  |
|| `PasswordValidationRules.php` | Defines password validation rules     |

Ví dụ, để tùy chỉnh logic đăng ký của ứng dụng, bạn nên sửa đổi action `CreateNewUser`:

```php
public function create(array $input): User
{
    Validator::make($input, [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'email', 'max:255', 'unique:users'],
        'phone' => ['required', 'string', 'max:20'], // [tl! add]
        'password' => $this->passwordRules(),
    ])->validate();

    return User::create([
        'name' => $input['name'],
        'email' => $input['email'],
        'phone' => $input['phone'], // [tl! add]
        'password' => Hash::make($input['password']),
    ]);
}
```

<a name="two-factor-authentication"></a>
### Two-Factor Authentication

Starter kits bao gồm xác thực hai yếu tố (2FA) tích hợp sẵn, cho phép người dùng bảo mật tài khoản của họ bằng cách sử dụng bất kỳ ứng dụng authenticator tương thích TOTP nào. 2FA được bật theo mặc định thông qua `Features::twoFactorAuthentication()` trong file cấu hình `config/fortify.php` của ứng dụng.

Tùy chọn `confirm` yêu cầu người dùng xác minh một mã trước khi 2FA được bật hoàn toàn, trong khi `confirmPassword` yêu cầu xác nhận mật khẩu trước khi bật hoặc tắt 2FA. Để biết thêm chi tiết, hãy xem [tài liệu xác thực hai yếu tố của Fortify](/docs/{{version}}/fortify#two-factor-authentication).

<a name="rate-limiting"></a>
### Rate Limiting

Rate limiting ngăn chặn brute-forcing và các nỗ lực đăng nhập lặp lại làm quá tải các endpoints xác thực của bạn. Bạn có thể tùy chỉnh hành vi rate limiting của Fortify trong `FortifyServiceProvider` của ứng dụng:

```php
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;

RateLimiter::for('login', function ($request) {
    return Limit::perMinute(5)->by($request->email.$request->ip());
});
```

<a name="teams"></a>
## Teams

Các starter kits React, Svelte, Vue, và Livewire cũng có thể được tạo với hỗ trợ team. Khi tính năng team được bật, mỗi người dùng thuộc về một hoặc nhiều teams và có một team hiện tại. Trong quá trình đăng ký, người dùng mới tự động được cấp một team cá nhân. Các starter kits cũng bao gồm các màn hình quản lý team để tạo teams, chuyển đổi giữa các teams, mời thành viên, và cập nhật chi tiết team.

Khi một route được scope đến team hiện tại, slug của team hiện tại được bao gồm trong URL. Ví dụ, route dashboard trở thành `/{current_team}/dashboard`, trong khi các trang quản lý team sử dụng các routes như `settings/teams/{team}`. Khi sử dụng các tham số route `{current_team}` và `{team}`, các starter kits tự động đảm bảo rằng người dùng được xác thực thuộc về team được yêu cầu trước khi cho phép truy cập vào route.

Để thuận tiện hơn cho việc tạo các URLs có nhận thức về team, các starter kits đăng ký các mặc định URL cho team hiện tại của người dùng được xác thực. Điều này cho phép các cuộc gọi đến các helpers như `route('dashboard')` tự động bao gồm slug của team hiện tại. Khi người dùng đăng nhập, đăng ký, hoặc chuyển đổi teams, các starter kits cập nhật team hiện tại và làm mới các mặc định URL này để các links được tạo tiếp tục sử dụng ngữ cảnh team đúng.

Khi tạo hoặc đổi tên một team, các starter kits cũng ngăn người dùng chọn các tên được dành riêng có thể tạo ra các segments route không an toàn hoặc xung đột. Ví dụ, các tên sẽ xung đột với các tiền tố route như `settings`, `login`, hoặc `dashboard` có thể không được sử dụng.

<a name="workos"></a>
## WorkOS AuthKit Authentication

Theo mặc định, các starter kits React, Svelte, Vue, và Livewire đều sử dụng hệ thống xác thực tích hợp sẵn của Laravel để cung cấp đăng nhập, đăng ký, reset mật khẩu, xác thực email, và hơn thế nữa. Ngoài ra, chúng tôi cũng cung cấp một biến thể [WorkOS AuthKit](https://authkit.com) được hỗ trợ bởi mỗi starter kit cung cấp:

<div class="content-list" markdown="1">

- Xác thực xã hội (Google, Microsoft, GitHub, và Apple)
- Xác thực passkey
- "Magic Auth" dựa trên email
- SSO

</div>

Sử dụng WorkOS làm nhà cung cấp xác thực của bạn [yêu cầu một tài khoản WorkOS](https://workos.com). WorkOS cung cấp xác thực miễn phí cho các ứng dụng lên đến 1 triệu người dùng hoạt động hàng tháng.

Để sử dụng WorkOS AuthKit làm nhà cung cấp xác thực của ứng dụng, chọn tùy chọn WorkOS khi tạo ứng dụng mới được hỗ trợ bởi starter kit thông qua `laravel new`.

### Configuring Your WorkOS Starter Kit

Sau khi tạo một ứng dụng mới bằng cách sử dụng một starter kit được hỗ trợ bởi WorkOS, bạn nên đặt các biến môi trường `WORKOS_CLIENT_ID`, `WORKOS_API_KEY`, và `WORKOS_REDIRECT_URL` trong file `.env` của ứng dụng. Các biến này nên khớp với các giá trị được cung cấp cho bạn trong dashboard WorkOS cho ứng dụng của bạn:

```ini
WORKOS_CLIENT_ID=your-client-id
WORKOS_API_KEY=your-api-key
WORKOS_REDIRECT_URL="${APP_URL}/authenticate"
```

Ngoài ra, bạn nên cấu hình URL trang chủ ứng dụng trong dashboard WorkOS của bạn. URL này là nơi người dùng sẽ được chuyển hướng sau khi họ đăng xuất khỏi ứng dụng của bạn.

<a name="configuring-authkit-authentication-methods"></a>
#### Configuring AuthKit Authentication Methods

Khi sử dụng một starter kit được hỗ trợ bởi WorkOS, chúng tôi khuyến nghị bạn tắt xác thực "Email + Password" trong các cài đặt cấu hình WorkOS AuthKit của ứng dụng, cho phép người dùng chỉ xác thực thông qua các nhà cung cấp xác thực xã hội, passkeys, "Magic Auth", và SSO. Điều này cho phép ứng dụng của bạn hoàn toàn tránh xử lý mật khẩu người dùng.

<a name="configuring-authkit-session-timeouts"></a>
#### Configuring AuthKit Session Timeouts

Ngoài ra, chúng tôi khuyến nghị bạn cấu hình timeout không hoạt động của session WorkOS AuthKit để khớp với ngưỡng timeout session được cấu hình của ứng dụng Laravel, thường là hai giờ.

<a name="inertia-ssr"></a>
### Inertia SSR

Các starter kits React, Svelte, và Vue tương thích với các khả năng [server-side rendering](https://inertiajs.com/server-side-rendering) của Inertia. Để xây dựng một bundle tương thích Inertia SSR cho ứng dụng, chạy lệnh `build:ssr`:

```shell
npm run build:ssr
```

Để thuận tiện, một lệnh `composer dev:ssr` cũng có sẵn. Lệnh này sẽ bắt đầu server phát triển Laravel và server Inertia SSR sau khi xây dựng một bundle tương thích SSR cho ứng dụng, cho phép bạn kiểm tra ứng dụng của bạn cục bộ bằng cách sử dụng engine server-side rendering của Inertia:

```shell
composer dev:ssr
```

<a name="community-maintained-starter-kits"></a>
### Community Maintained Starter Kits

Khi tạo một ứng dụng Laravel mới bằng cách sử dụng Laravel installer, bạn có thể cung cấp bất kỳ starter kit được duy trì bởi cộng đồng nào có sẵn trên Packagist cho cờ `--using`:

```shell
laravel new my-app --using=example/starter-kit
```

<a name="creating-starter-kits"></a>
#### Creating Starter Kits

Để đảm bảo starter kit của bạn có sẵn cho những người khác, bạn sẽ cần xuất bản nó lên [Packagist](https://packagist.org). Starter kit của bạn nên định nghĩa các biến môi trường được yêu cầu của nó trong file `.env.example`, và bất kỳ lệnh post-cài đặt cần thiết nào nên được liệt kê trong array `post-create-project-cmd` của file `composer.json` của starter kit.

<a name="faqs"></a>
### Frequently Asked Questions

<a name="faq-upgrade"></a>
#### How do I upgrade?

Mỗi starter kit cung cấp cho bạn một điểm khởi đầu vững chắc cho ứng dụng tiếp theo của bạn. Với quyền sở hữu đầy đủ mã, bạn có thể tinh chỉnh, tùy chỉnh và xây dựng ứng dụng của mình chính xác như bạn hình dung. Tuy nhiên, không cần cập nhật chính starter kit.

<a name="faq-enable-email-verification"></a>
#### How do I enable email verification?

Xác thực email có thể được thêm bằng cách bỏ comment import `MustVerifyEmail` trong model `App/Models/User.php` của bạn và đảm bảo model implements interface `MustVerifyEmail`:

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
// ...

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

Sau khi đăng ký, người dùng sẽ nhận được một email xác thực. Để hạn chế truy cập vào một số routes nhất định cho đến khi địa chỉ email của người dùng được xác thực, thêm middleware `verified` vào các routes:

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('dashboard', function () {
        return Inertia::render('dashboard');
    })->name('dashboard');
});
```

> [!NOTE]
> Xác thực email không được yêu cầu khi sử dụng biến thể [WorkOS](#workos) của các starter kits.

<a name="faq-modify-email-template"></a>
#### How do I modify the default email template?

Bạn có thể muốn tùy chỉnh template email mặc định để phù hợp hơn với thương hiệu của ứng dụng. Để sửa đổi template này, bạn nên xuất bản các views email vào ứng dụng của bạn với lệnh sau:

```
php artisan vendor:publish --tag=laravel-mail
```

Điều này sẽ tạo ra một số file trong `resources/views/vendor/mail`. Bạn có thể sửa đổi bất kỳ file nào trong số này cũng như file `resources/views/vendor/mail/themes/default.css` để thay đổi giao diện và hình thức của template email mặc định.
