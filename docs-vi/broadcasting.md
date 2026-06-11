# Broadcasting

- [Introduction](#introduction)
- [Quickstart](#quickstart)
- [Server Side Installation](#server-side-installation)
    - [Reverb](#reverb)
    - [Pusher Channels](#pusher-channels)
    - [Ably](#ably)
- [Client Side Installation](#client-side-installation)
    - [Reverb](#client-reverb)
    - [Pusher Channels](#client-pusher-channels)
    - [Ably](#client-ably)
- [Concept Overview](#concept-overview)
    - [Using an Example Application](#using-example-application)
- [Defining Broadcast Events](#defining-broadcast-events)
    - [Broadcast Name](#broadcast-name)
    - [Broadcast Data](#broadcast-data)
    - [Broadcast Queue](#broadcast-queue)
    - [Broadcast Conditions](#broadcast-conditions)
    - [Broadcasting and Database Transactions](#broadcasting-and-database-transactions)
- [Authorizing Channels](#authorizing-channels)
    - [Defining Authorization Callbacks](#defining-authorization-callbacks)
    - [Defining Channel Classes](#defining-channel-classes)
- [Broadcasting Events](#broadcasting-events)
    - [Only to Others](#only-to-others)
    - [Customizing the Connection](#customizing-the-connection)
    - [Anonymous Events](#anonymous-events)
    - [Rescuing Broadcasts](#rescuing-broadcasts)
- [Receiving Broadcasts](#receiving-broadcasts)
    - [Listening for Events](#listening-for-events)
    - [Leaving a Channel](#leaving-a-channel)
    - [Namespaces](#namespaces)
    - [Using React, Vue, or Svelte](#using-react-or-vue)
- [Presence Channels](#presence-channels)
    - [Authorizing Presence Channels](#authorizing-presence-channels)
    - [Joining Presence Channels](#joining-presence-channels)
    - [Broadcasting to Presence Channels](#broadcasting-to-presence-channels)
- [Model Broadcasting](#model-broadcasting)
    - [Model Broadcasting Conventions](#model-broadcasting-conventions)
    - [Listening for Model Broadcasts](#listening-for-model-broadcasts)
- [Client Events](#client-events)
- [Notifications](#notifications)

<a name="introduction"></a>
## Introduction

Trong nhiều ứng dụng web hiện đại, WebSockets được sử dụng để implement các giao diện người dùng realtime, live-updating. Khi một số dữ liệu được cập nhật trên server, một message thường được gửi qua kết nối WebSocket để được xử lý bởi client. WebSockets cung cấp một giải pháp thay thế hiệu quả hơn so với việc liên tục polling server của ứng dụng của bạn để tìm các thay đổi dữ liệu nên được phản ánh trong UI của bạn.

Ví dụ, hãy tưởng tượng ứng dụng của bạn có thể export dữ liệu của người dùng thành một file CSV và email nó cho họ. Tuy nhiên, việc tạo file CSV này mất vài phút nên bạn chọn tạo và gửi CSV trong một [queued job](/docs/{{version}}/queues). Khi CSV đã được tạo và gửi cho người dùng, chúng ta có thể sử dụng event broadcasting để dispatch một sự kiện `App\Events\UserDataExported` được nhận bởi JavaScript của ứng dụng của chúng ta. Khi sự kiện được nhận, chúng ta có thể hiển thị một message cho người dùng rằng CSV của họ đã được emailed cho họ mà họ không bao giờ cần refresh trang.

Để hỗ trợ bạn xây dựng các loại tính năng này, Laravel làm cho việc "broadcast" các [sự kiện](/docs/{{version}}/events) Laravel phía server của bạn qua kết nối WebSocket trở nên dễ dàng. Broadcasting các sự kiện Laravel của bạn cho phép bạn chia sẻ cùng tên sự kiện và dữ liệu giữa ứng dụng Laravel phía server và ứng dụng JavaScript phía client của bạn.

Các khái niệm cốt lõi đằng sau broadcasting rất đơn giản: clients kết nối đến các channels được đặt tên trên frontend, trong khi ứng dụng Laravel của bạn broadcast các sự kiện đến các channels này trên backend. Các sự kiện này có thể chứa bất kỳ dữ liệu bổ sung nào bạn muốn cung cấp cho frontend.

<a name="supported-drivers"></a>
#### Supported Drivers

Theo mặc định, Laravel bao gồm ba driver broadcasting phía server để bạn chọn: [Laravel Reverb](https://reverb.laravel.com), [Pusher Channels](https://pusher.com/channels), và [Ably](https://ably.com).

> [!NOTE]
> Before diving into event broadcasting, make sure you have read Laravel's documentation on [events and listeners](/docs/{{version}}/events).

<a name="quickstart"></a>
## Quickstart

Theo mặc định, broadcasting không được bật trong các ứng dụng Laravel mới. Bạn có thể bật broadcasting sử dụng lệnh Artisan `install:broadcasting`:

```shell
php artisan install:broadcasting
```

Lệnh `install:broadcasting` sẽ nhắc bạn chọn dịch vụ event broadcasting nào bạn muốn sử dụng. Ngoài ra, nó sẽ tạo file cấu hình `config/broadcasting.php` và file `routes/channels.php` nơi bạn có thể đăng ký các routes và callbacks authorization broadcast của ứng dụng của bạn.

Laravel supports several broadcast drivers out of the box: [Laravel Reverb](/docs/{{version}}/reverb), [Pusher Channels](https://pusher.com/channels), [Ably](https://ably.com), and a `log` driver for local development and debugging. Additionally, a `null` driver is included which allows you to disable broadcasting during testing. A configuration example is included for each of these drivers in the `config/broadcasting.php` configuration file.

Tất cả cấu hình event broadcasting của ứng dụng của bạn được lưu trữ trong file cấu hình `config/broadcasting.php`. Đừng lo lắng nếu file này không tồn tại trong ứng dụng của bạn; nó sẽ được tạo khi bạn chạy lệnh Artisan `install:broadcasting`.

<a name="quickstart-next-steps"></a>
#### Next Steps

Sau khi bạn đã bật event broadcasting, bạn đã sẵn sàng để tìm hiểu thêm về [defining broadcast events](#defining-broadcast-events) và [listening for events](#listening-for-events). Nếu bạn đang sử dụng [starter kits](/docs/{{version}}/starter-kits) React, Vue, hoặc Svelte của Laravel, bạn có thể lắng nghe các sự kiện sử dụng [useEcho hook](#using-react-or-vue) của Echo.

> [!NOTE]
> Before broadcasting any events, you should first configure and run a [queue worker](/docs/{{version}}/queues). All event broadcasting is done via queued jobs so that the response time of your application is not seriously affected by events being broadcast.

<a name="server-side-installation"></a>
## Server Side Installation

Để bắt đầu sử dụng event broadcasting của Laravel, chúng ta cần thực hiện một số cấu hình trong ứng dụng Laravel cũng như cài đặt một vài gói.

Event broadcasting được thực hiện bởi một driver broadcasting phía server broadcast các sự kiện Laravel của bạn để Laravel Echo (một thư viện JavaScript) có thể nhận chúng trong client trình duyệt. Đừng lo lắng - chúng ta sẽ đi qua từng phần của quá trình cài đặt từng bước.

<a name="reverb"></a>
### Reverb

Để nhanh chóng bật hỗ trợ cho các tính năng broadcasting của Laravel khi sử dụng Reverb như event broadcaster của bạn, hãy gọi lệnh Artisan `install:broadcasting` với tùy chọn `--reverb`. Lệnh Artisan này sẽ cài đặt các packages Composer và NPM cần thiết của Reverb và cập nhật file `.env` của ứng dụng của bạn với các biến thích hợp:

```shell
php artisan install:broadcasting --reverb
```

<a name="reverb-manual-installation"></a>
#### Manual Installation

Khi chạy lệnh `install:broadcasting`, bạn sẽ được nhắc cài đặt [Laravel Reverb](/docs/{{version}}/reverb). Tất nhiên, bạn cũng có thể cài đặt Reverb thủ công sử dụng Composer package manager:

```shell
composer require laravel/reverb
```

Sau khi package được cài đặt, bạn có thể chạy lệnh cài đặt của Reverb để publish cấu hình, thêm các biến môi trường cần thiết của Reverb, và bật event broadcasting trong ứng dụng của bạn:

```shell
php artisan reverb:install
```

You can find detailed Reverb installation and usage instructions in the [Reverb documentation](/docs/{{version}}/reverb).

<a name="pusher-channels"></a>
### Pusher Channels

Để nhanh chóng bật hỗ trợ cho các tính năng broadcasting của Laravel khi sử dụng Pusher như event broadcaster của bạn, hãy gọi lệnh Artisan `install:broadcasting` với tùy chọn `--pusher`. Lệnh Artisan này sẽ nhắc bạn nhập thông tin xác thực Pusher, cài đặt các SDK PHP và JavaScript của Pusher, và cập nhật file `.env` của ứng dụng của bạn với các biến thích hợp:

```shell
php artisan install:broadcasting --pusher
```

<a name="pusher-manual-installation"></a>
#### Manual Installation

Để cài đặt hỗ trợ Pusher thủ công, bạn nên cài đặt Pusher Channels PHP SDK sử dụng Composer package manager:

```shell
composer require pusher/pusher-php-server
```

Next, you should configure your Pusher Channels credentials in the `config/broadcasting.php` configuration file. An example Pusher Channels configuration is already included in this file, allowing you to quickly specify your key, secret, and application ID. Typically, you should configure your Pusher Channels credentials in your application's `.env` file:

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"
```

Cấu hình `pusher` trong file `config/broadcasting.php` cũng cho phép bạn chỉ định các `options` bổ sung được hỗ trợ bởi Channels, chẳng hạn như cluster.

Sau đó, đặt environment variable `BROADCAST_CONNECTION` thành `pusher` trong file `.env` của ứng dụng của bạn:

```ini
BROADCAST_CONNECTION=pusher
```

Finally, you are ready to install and configure [Laravel Echo](#client-side-installation), which will receive the broadcast events on the client-side.

<a name="ably"></a>
### Ably

> [!NOTE]
> The documentation below discusses how to use Ably in "Pusher compatibility" mode. However, the Ably team recommends and maintains a broadcaster and Echo client that is able to take advantage of the unique capabilities offered by Ably. For more information on using the Ably maintained drivers, please [consult Ably's Laravel broadcaster documentation](https://github.com/ably/laravel-broadcaster).

Để nhanh chóng bật hỗ trợ cho các tính năng broadcasting của Laravel khi sử dụng [Ably](https://ably.com) như event broadcaster của bạn, hãy gọi lệnh Artisan `install:broadcasting` với tùy chọn `--ably`. Lệnh Artisan này sẽ nhắc bạn nhập thông tin xác thực Ably, cài đặt các SDK PHP và JavaScript của Ably, và cập nhật file `.env` của ứng dụng của bạn với các biến thích hợp:

```shell
php artisan install:broadcasting --ably
```

**Before continuing, you should enable Pusher protocol support in your Ably application settings. You may enable this feature within the "Protocol Adapter Settings" portion of your Ably application's settings dashboard.**

<a name="ably-manual-installation"></a>
#### Manual Installation

Để cài đặt hỗ trợ Ably thủ công, bạn nên cài đặt Ably PHP SDK sử dụng Composer package manager:

```shell
composer require ably/ably-php
```

Next, you should configure your Ably credentials in the `config/broadcasting.php` configuration file. An example Ably configuration is already included in this file, allowing you to quickly specify your key. Typically, this value should be set via the `ABLY_KEY` [environment variable](/docs/{{version}}/configuration#environment-configuration):

```ini
ABLY_KEY=your-ably-key
```

Sau đó, đặt environment variable `BROADCAST_CONNECTION` thành `ably` trong file `.env` của ứng dụng của bạn:

```ini
BROADCAST_CONNECTION=ably
```

Finally, you are ready to install and configure [Laravel Echo](#client-side-installation), which will receive the broadcast events on the client-side.

<a name="client-side-installation"></a>
## Client Side Installation

<a name="client-reverb"></a>
### Reverb

[Laravel Echo](https://github.com/laravel/echo) is a JavaScript library that makes it painless to subscribe to channels and listen for events broadcast by your server-side broadcasting driver.

Khi cài đặt Laravel Reverb qua lệnh Artisan `install:broadcasting`, scaffolding và cấu hình của Reverb và Echo sẽ được tự động inject vào ứng dụng của bạn. Tuy nhiên, nếu bạn muốn cấu hình Laravel Echo thủ công, bạn có thể làm như vậy bằng cách làm theo hướng dẫn dưới đây.

<a name="reverb-client-manual-installation"></a>
#### Manual Installation

Để cấu hình Laravel Echo thủ công cho frontend của ứng dụng, trước tiên cài đặt package `pusher-js` vì Reverb sử dụng giao thức Pusher cho WebSocket subscriptions, channels, và messages:

```shell
npm install --save-dev laravel-echo pusher-js
```

Sau khi Echo được cài đặt, bạn đã sẵn sàng để tạo một instance Echo mới trong JavaScript của ứng dụng. Một nơi tuyệt vời để làm điều này là ở cuối file `resources/js/app.js` được bao gồm với Laravel framework:

```js tab=JavaScript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "reverb",
    // key: import.meta.env.VITE_REVERB_APP_KEY,
    // wsHost: import.meta.env.VITE_REVERB_HOST,
    // wsPort: import.meta.env.VITE_REVERB_PORT,
    // wssPort: import.meta.env.VITE_REVERB_PORT,
    // forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    // enabledTransports: ['ws', 'wss'],
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "reverb",
    // key: import.meta.env.VITE_REVERB_APP_KEY,
    // wsHost: import.meta.env.VITE_REVERB_HOST,
    // wsPort: import.meta.env.VITE_REVERB_PORT,
    // wssPort: import.meta.env.VITE_REVERB_PORT,
    // forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    // enabledTransports: ['ws', 'wss'],
});
```

```js tab=Svelte
import { configureEcho } from "@laravel/echo-svelte";

configureEcho({
    broadcaster: "reverb",
    // key: import.meta.env.VITE_REVERB_APP_KEY,
    // wsHost: import.meta.env.VITE_REVERB_HOST,
    // wsPort: import.meta.env.VITE_REVERB_PORT,
    // wssPort: import.meta.env.VITE_REVERB_PORT,
    // forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    // enabledTransports: ['ws', 'wss'],
});
```

Next, you should compile your application's assets:

```shell
npm run build
```

> [!WARNING]
> The Laravel Echo `reverb` broadcaster requires laravel-echo v1.16.0+.

<a name="client-pusher-channels"></a>
### Pusher Channels

[Laravel Echo](https://github.com/laravel/echo) is a JavaScript library that makes it painless to subscribe to channels and listen for events broadcast by your server-side broadcasting driver.

Khi cài đặt hỗ trợ broadcasting qua lệnh Artisan `install:broadcasting --pusher`, scaffolding và cấu hình của Pusher và Echo sẽ được tự động inject vào ứng dụng của bạn. Tuy nhiên, nếu bạn muốn cấu hình Laravel Echo thủ công, bạn có thể làm như vậy bằng cách làm theo hướng dẫn dưới đây.

<a name="pusher-client-manual-installation"></a>
#### Manual Installation

Để cấu hình Laravel Echo thủ công cho frontend của ứng dụng, trước tiên cài đặt các packages `laravel-echo` và `pusher-js` sử dụng giao thức Pusher cho WebSocket subscriptions, channels, và messages:

```shell
npm install --save-dev laravel-echo pusher-js
```

Sau khi Echo được cài đặt, bạn đã sẵn sàng để tạo một instance Echo mới trong file `resources/js/app.js` của ứng dụng:

```js tab=JavaScript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "pusher",
    // key: import.meta.env.VITE_PUSHER_APP_KEY,
    // cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    // forceTLS: true,
    // wsHost: import.meta.env.VITE_PUSHER_HOST,
    // wsPort: import.meta.env.VITE_PUSHER_PORT,
    // wssPort: import.meta.env.VITE_PUSHER_PORT,
    // enabledTransports: ["ws", "wss"],
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "pusher",
    // key: import.meta.env.VITE_PUSHER_APP_KEY,
    // cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    // forceTLS: true,
    // wsHost: import.meta.env.VITE_PUSHER_HOST,
    // wsPort: import.meta.env.VITE_PUSHER_PORT,
    // wssPort: import.meta.env.VITE_PUSHER_PORT,
    // enabledTransports: ["ws", "wss"],
});
```

```js tab=Svelte
import { configureEcho } from "@laravel/echo-svelte";

configureEcho({
    broadcaster: "pusher",
    // key: import.meta.env.VITE_PUSHER_APP_KEY,
    // cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    // forceTLS: true,
    // wsHost: import.meta.env.VITE_PUSHER_HOST,
    // wsPort: import.meta.env.VITE_PUSHER_PORT,
    // wssPort: import.meta.env.VITE_PUSHER_PORT,
    // enabledTransports: ["ws", "wss"],
});
```

Next, you should define the appropriate values for the Pusher environment variables in your application's `.env` file. If these variables do not already exist in your `.env` file, you should add them:

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"

VITE_APP_NAME="${APP_NAME}"
VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

Sau khi bạn đã điều chỉnh cấu hình Echo theo nhu cầu của ứng dụng, bạn có thể compile các assets của ứng dụng:

```shell
npm run build
```

> [!NOTE]
> To learn more about compiling your application's JavaScript assets, please consult the documentation on [Vite](/docs/{{version}}/vite).

<a name="using-an-existing-client-instance"></a>
#### Using an Existing Client Instance

Nếu bạn đã có một instance client Pusher Channels được cấu hình trước mà bạn muốn Echo sử dụng, bạn có thể truyền nó cho Echo qua tùy chọn cấu hình `client`:

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

const options = {
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY
}

window.Echo = new Echo({
    ...options,
    client: new Pusher(options.key, options)
});
```

<a name="client-ably"></a>
### Ably

> [!NOTE]
> The documentation below discusses how to use Ably in "Pusher compatibility" mode. However, the Ably team recommends and maintains a broadcaster and Echo client that is able to take advantage of the unique capabilities offered by Ably. For more information on using the Ably maintained drivers, please [consult Ably's Laravel broadcaster documentation](https://github.com/ably/laravel-broadcaster).

[Laravel Echo](https://github.com/laravel/echo) is a JavaScript library that makes it painless to subscribe to channels and listen for events broadcast by your server-side broadcasting driver.

Khi cài đặt hỗ trợ broadcasting qua lệnh Artisan `install:broadcasting --ably`, scaffolding và cấu hình của Ably và Echo sẽ được tự động inject vào ứng dụng của bạn. Tuy nhiên, nếu bạn muốn cấu hình Laravel Echo thủ công, bạn có thể làm như vậy bằng cách làm theo hướng dẫn dưới đây.

<a name="ably-client-manual-installation"></a>
#### Manual Installation

To manually configure Laravel Echo for your application's frontend, first install the `laravel-echo` and `pusher-js` packages which utilize the Pusher protocol for WebSocket subscriptions, channels, and messages:

```shell
npm install --save-dev laravel-echo pusher-js
```

**Before continuing, you should enable Pusher protocol support in your Ably application settings. You may enable this feature within the "Protocol Adapter Settings" portion of your Ably application's settings dashboard.**

Sau khi Echo được cài đặt, bạn đã sẵn sàng để tạo một instance Echo mới trong file `resources/js/app.js` của ứng dụng:

```js tab=JavaScript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    wsHost: 'realtime-pusher.ably.io',
    wsPort: 443,
    disableStats: true,
    encrypted: true,
});
```

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "ably",
    // key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    // wsHost: "realtime-pusher.ably.io",
    // wsPort: 443,
    // disableStats: true,
    // encrypted: true,
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "ably",
    // key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    // wsHost: "realtime-pusher.ably.io",
    // wsPort: 443,
    // disableStats: true,
    // encrypted: true,
});
```

```js tab=Svelte
import { configureEcho } from "@laravel/echo-svelte";

configureEcho({
    broadcaster: "ably",
    // key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    // wsHost: "realtime-pusher.ably.io",
    // wsPort: 443,
    // disableStats: true,
    // encrypted: true,
});
```

You may have noticed our Ably Echo configuration references a `VITE_ABLY_PUBLIC_KEY` environment variable. This variable's value should be your Ably public key. Your public key is the portion of your Ably key that occurs before the `:` character.

Sau khi bạn đã điều chỉnh cấu hình Echo theo nhu cầu của bạn, bạn có thể compile các assets của ứng dụng:

```shell
npm run dev
```

> [!NOTE]
> To learn more about compiling your application's JavaScript assets, please consult the documentation on [Vite](/docs/{{version}}/vite).

<a name="concept-overview"></a>
## Concept Overview

Event broadcasting của Laravel cho phép bạn broadcast các sự kiện Laravel phía server của bạn đến ứng dụng JavaScript phía client của bạn sử dụng cách tiếp cận dựa trên driver cho WebSockets. Hiện tại, Laravel đi kèm với các driver [Laravel Reverb](https://reverb.laravel.com), [Pusher Channels](https://pusher.com/channels), và [Ably](https://ably.com). Các sự kiện có thể được tiêu thụ dễ dàng trên phía client sử dụng gói JavaScript [Laravel Echo](#client-side-installation).

Các sự kiện được broadcast qua các "channels", có thể được chỉ định là public hoặc private. Bất kỳ khách truy cập nào đến ứng dụng của bạn có thể subscribe đến một public channel mà không cần bất kỳ xác thực hay authorization nào; tuy nhiên, để subscribe đến một private channel, người dùng phải được xác thực và được authorize để lắng nghe trên channel đó.

<a name="using-example-application"></a>
### Using an Example Application

Trước khi đi sâu vào từng thành phần của event broadcasting, hãy lấy một cái nhìn tổng quan sử dụng một cửa hàng thương mại điện tử làm ví dụ.

Trong ứng dụng của chúng ta, hãy giả sử chúng ta có một trang cho phép người dùng xem trạng thái vận chuyển cho các đơn hàng của họ. Hãy cũng giả sử rằng một sự kiện `OrderShipmentStatusUpdated` được kích hoạt khi một cập nhật trạng thái vận chuyển được xử lý bởi ứng dụng:

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="the-shouldbroadcast-interface"></a>
#### The `ShouldBroadcast` Interface

Khi một người dùng đang xem một trong các đơn hàng của họ, chúng ta không muốn họ phải refresh trang để xem các cập nhật trạng thái. Thay vào đó, chúng ta muốn broadcast các cập nhật đến ứng dụng khi chúng được tạo. Vì vậy, chúng ta cần đánh dấu sự kiện `OrderShipmentStatusUpdated` với interface `ShouldBroadcast`. Điều này sẽ chỉ dẫn Laravel broadcast sự kiện khi nó được kích hoạt:

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class OrderShipmentStatusUpdated implements ShouldBroadcast
{
    /**
     * The order instance.
     *
     * @var \App\Models\Order
     */
    public $order;
}
```

Interface `ShouldBroadcast` yêu cầu sự kiện của chúng ta định nghĩa một phương thức `broadcastOn`. Phương thức này chịu trách nhiệm trả về các channels mà sự kiện nên broadcast trên. Một stub trống của phương thức này đã được định nghĩa trên các class sự kiện được tạo, vì vậy chúng ta chỉ cần điền vào chi tiết của nó. Chúng ta chỉ muốn người tạo đơn hàng có thể xem các cập nhật trạng thái, vì vậy chúng ta sẽ broadcast sự kiện trên một private channel được gắn với đơn hàng:

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channel the event should broadcast on.
 */
public function broadcastOn(): Channel
{
    return new PrivateChannel('orders.'.$this->order->id);
}
```

Nếu bạn muốn sự kiện broadcast trên nhiều channels, bạn có thể trả về một `array` thay vì:

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channels the event should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(): array
{
    return [
        new PrivateChannel('orders.'.$this->order->id),
        // ...
    ];
}
```

<a name="example-application-authorizing-channels"></a>
#### Authorizing Channels

Remember, users must be authorized to listen on private channels. We may define our channel authorization rules in our application's `routes/channels.php` file. In this example, we need to verify that any user attempting to listen on the private `orders.1` channel is actually the creator of the order:

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

Phương thức `channel` chấp nhận hai đối số: tên của channel và một callback trả về `true` hoặc `false` chỉ định liệu người dùng có được authorize để lắng nghe trên channel hay không.

Tất cả các authorization callbacks nhận người dùng được xác thực hiện tại làm đối số đầu tiên của họ và bất kỳ tham số wildcard bổ sung nào làm các đối số tiếp theo của họ. Trong ví dụ này, chúng ta đang sử dụng placeholder `{orderId}` để chỉ định rằng phần "ID" của tên channel là một wildcard.

<a name="listening-for-event-broadcasts"></a>
#### Listening for Event Broadcasts

Tiếp theo, tất cả những gì còn lại là lắng nghe sự kiện trong ứng dụng JavaScript của chúng ta. Chúng ta có thể làm điều này sử dụng [Laravel Echo](#client-side-installation). Các hooks React, Vue, và Svelte tích hợp của Laravel Echo làm cho việc bắt đầu trở nên đơn giản, và theo mặc định, tất cả các thuộc tính public của sự kiện sẽ được bao gồm trên sự kiện broadcast:

```js tab=React
import { useEcho } from "@laravel/echo-react";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
</script>
```

```svelte tab=Svelte
<script>
import { useEcho } from "@laravel/echo-svelte";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
</script>
```

<a name="defining-broadcast-events"></a>
## Defining Broadcast Events

Để thông báo cho Laravel rằng một sự kiện cụ thể nên được broadcast, bạn phải implement interface `Illuminate\Contracts\Broadcasting\ShouldBroadcast` trên class sự kiện. Interface này đã được import vào tất cả các class sự kiện được tạo bởi framework nên bạn có thể dễ dàng thêm nó vào bất kỳ sự kiện nào của mình.

Interface `ShouldBroadcast` yêu cầu bạn implement một phương thức duy nhất: `broadcastOn`. Phương thức `broadcastOn` nên trả về một channel hoặc mảng các channels mà sự kiện nên broadcast trên. Các channels nên là các instance của `Channel`, `PrivateChannel`, hoặc `PresenceChannel`. Các instance của `Channel` đại diện cho các public channels mà bất kỳ người dùng nào có thể subscribe, trong khi `PrivateChannels` và `PresenceChannels` đại diện cho các private channels yêu cầu [channel authorization](#authorizing-channels):

```php
<?php

namespace App\Events;

use App\Models\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('user.'.$this->user->id),
        ];
    }
}
```

After implementing the `ShouldBroadcast` interface, you only need to [fire the event](/docs/{{version}}/events) as you normally would. Once the event has been fired, a [queued job](/docs/{{version}}/queues) will automatically broadcast the event using your specified broadcast driver.

<a name="broadcast-name"></a>
### Broadcast Name

Theo mặc định, Laravel sẽ broadcast sự kiện sử dụng tên class của sự kiện. Tuy nhiên, bạn có thể tùy chỉnh tên broadcast bằng cách định nghĩa một phương thức `broadcastAs` trên sự kiện:

```php
/**
 * The event's broadcast name.
 */
public function broadcastAs(): string
{
    return 'server.created';
}
```

If you customize the broadcast name using the `broadcastAs` method, you should make sure to register your listener with a leading `.` character. This will instruct Echo to not prepend the application's namespace to the event:

```javascript
.listen('.server.created', function (e) {
    // ...
});
```

<a name="broadcast-data"></a>
### Broadcast Data

Khi một sự kiện được broadcast, tất cả các thuộc tính `public` của nó được tự động serialize và broadcast như payload của sự kiện, cho phép bạn truy cập bất kỳ dữ liệu public nào của nó từ ứng dụng JavaScript của bạn. Vì vậy, ví dụ, nếu sự kiện của bạn có một thuộc tính public duy nhất `$user` chứa một model Eloquent, payload broadcast của sự kiện sẽ là:

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

Tuy nhiên, nếu bạn muốn có kiểm soát tinh chỉnh hơn về payload broadcast của mình, bạn có thể thêm một phương thức `broadcastWith` vào sự kiện của bạn. Phương thức này nên trả về mảng dữ liệu mà bạn muốn broadcast như payload của sự kiện:

```php
/**
 * Get the data to broadcast.
 *
 * @return array<string, mixed>
 */
public function broadcastWith(): array
{
    return ['id' => $this->user->id];
}
```

<a name="broadcast-queue"></a>
### Broadcast Queue

Theo mặc định, mỗi broadcast event được đặt trên queue mặc định cho kết nối queue mặc định được chỉ định trong file cấu hình `queue.php` của bạn. Bạn có thể tùy chỉnh kết nối queue và tên được sử dụng bởi broadcaster bằng cách sử dụng các thuộc tính `Connection` và `Queue` trên class sự kiện của bạn:

```php
use Illuminate\Queue\Attributes\Connection;
use Illuminate\Queue\Attributes\Queue;

#[Connection('redis')]
#[Queue('default')]
class ServerCreated implements ShouldBroadcast
{
    // ...
}
```

Ngoài ra, bạn có thể tùy chỉnh tên queue bằng cách định nghĩa một phương thức `broadcastQueue` trên sự kiện của bạn:

```php
/**
 * The name of the queue on which to place the broadcasting job.
 */
public function broadcastQueue(): string
{
    return 'default';
}
```

If you would like to broadcast your event using the `sync` queue instead of the default queue driver, you can implement the `ShouldBroadcastNow` interface instead of `ShouldBroadcast`:

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class OrderShipmentStatusUpdated implements ShouldBroadcastNow
{
    // ...
}
```

<a name="broadcast-conditions"></a>
### Broadcast Conditions

Đôi khi bạn muốn broadcast event của bạn chỉ khi một điều kiện nhất định là true. Bạn có thể định nghĩa các điều kiện này bằng cách thêm một phương thức `broadcastWhen` vào class event của bạn:

```php
/**
 * Determine if this event should broadcast.
 */
public function broadcastWhen(): bool
{
    return $this->order->value > 100;
}
```

<a name="broadcasting-and-database-transactions"></a>
#### Broadcasting and Database Transactions

Khi broadcast events được dispatch trong database transactions, chúng có thể được xử lý bởi queue trước khi database transaction đã commit. Khi điều này xảy ra, bất kỳ updates nào bạn đã thực hiện cho models hoặc database records trong database transaction có thể chưa được phản ánh trong database. Ngoài ra, bất kỳ models hoặc database records nào được tạo trong transaction có thể không tồn tại trong database. Nếu event của bạn phụ thuộc vào các models này, các lỗi unexpected có thể xảy ra khi job broadcast event được xử lý.

Nếu tùy chọn cấu hình `after_commit` của queue connection của bạn được đặt thành `false`, bạn vẫn có thể chỉ định rằng một broadcast event cụ thể nên được dispatch sau khi tất cả các database transactions mở đã được commit bằng cách implement interface `ShouldDispatchAfterCommit` trên class event:

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast, ShouldDispatchAfterCommit
{
    use SerializesModels;
}
```

> [!NOTE]
> Để biết thêm thông tin về cách giải quyết các vấn đề này, hãy xem tài liệu về [queued jobs và database transactions](/docs/{{version}}/queues#jobs-and-database-transactions).

<a name="authorizing-channels"></a>
## Authorizing Channels

Private channels yêu cầu bạn authorize rằng user hiện tại được xác thực có thể thực sự listen trên channel. Điều này được thực hiện bằng cách thực hiện một HTTP request đến ứng dụng Laravel của bạn với tên channel và cho phép ứng dụng của bạn xác định xem user có thể listen trên channel đó hay không. Khi sử dụng [Laravel Echo](#client-side-installation), HTTP request để authorize subscriptions đến private channels sẽ được thực hiện tự động.

Khi broadcasting được cài đặt, Laravel cố gắng tự động register route `/broadcasting/auth` để xử lý các authorization requests. Nếu Laravel không thể tự động register các routes này, bạn có thể register chúng thủ công trong file `/bootstrap/app.php` của ứng dụng của bạn:

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    channels: __DIR__.'/../routes/channels.php',
    health: '/up',
)
```

<a name="defining-authorization-callbacks"></a>
### Defining Authorization Callbacks

Tiếp theo, chúng ta cần định nghĩa logic thực sự xác định xem user hiện tại được xác thực có thể listen trên một channel nhất định hay không. Điều này được thực hiện trong file `routes/channels.php` được tạo bởi lệnh Artisan `install:broadcasting`. Trong file này, bạn có thể sử dụng phương thức `Broadcast::channel` để register các channel authorization callbacks:

```php
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

Phương thức `channel` chấp nhận hai đối số: tên của channel và một callback trả về `true` hoặc `false` chỉ định xem user có được authorize để listen trên channel hay không.

Tất cả authorization callbacks nhận user hiện tại được xác thực làm đối số đầu tiên của chúng và bất kỳ tham số wildcard bổ sung nào làm các đối số tiếp theo của chúng. Trong ví dụ này, chúng ta đang sử dụng placeholder `{orderId}` để chỉ định rằng phần "ID" của tên channel là một wildcard.

Bạn có thể xem danh sách các broadcast authorization callbacks của ứng dụng của bạn sử dụng lệnh Artisan `channel:list`:

```shell
php artisan channel:list
```

<a name="authorization-callback-model-binding"></a>
#### Authorization Callback Model Binding

Giống như HTTP routes, channel routes cũng có thể tận dụng implicit và explicit [route model binding](/docs/{{version}}/routing#route-model-binding). Ví dụ, thay vì nhận một string hoặc numeric order ID, bạn có thể request một instance `Order` model thực tế:

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{order}', function (User $user, Order $order) {
    return $user->id === $order->user_id;
});
```

> [!WARNING]
> Khác với HTTP route model binding, channel model binding không hỗ trợ automatic [implicit model binding scoping](/docs/{{version}}/routing#implicit-model-binding-scoping). Tuy nhiên, điều này hiếm khi là một vấn đề vì hầu hết các channels có thể được scoped dựa trên một unique, primary key của một model duy nhất.

<a name="authorization-callback-authentication"></a>
#### Authorization Callback Authentication

Private và presence broadcast channels authenticate user hiện tại qua default authentication guard của ứng dụng của bạn. Nếu user không được xác thực, channel authorization bị tự động deny và authorization callback không bao giờ được thực thi. Tuy nhiên, bạn có thể gán nhiều, custom guards nên authenticate incoming request nếu cần thiết:

```php
Broadcast::channel('channel', function () {
    // ...
}, ['guards' => ['web', 'admin']]);
```

<a name="defining-channel-classes"></a>
### Defining Channel Classes

Nếu ứng dụng của bạn đang tiêu thụ nhiều channels khác nhau, file `routes/channels.php` của bạn có thể trở nên cồng kềnh. Vì vậy, thay vì sử dụng closures để authorize channels, bạn có thể sử dụng channel classes. Để tạo một channel class, hãy sử dụng lệnh Artisan `make:channel`. Lệnh này sẽ đặt một channel class mới trong thư mục `App/Broadcasting`.

```shell
php artisan make:channel OrderChannel
```

Next, register your channel in your `routes/channels.php` file:

```php
use App\Broadcasting\OrderChannel;

Broadcast::channel('orders.{order}', OrderChannel::class);
```

Cuối cùng, bạn có thể đặt authorization logic cho channel của bạn trong phương thức `join` của channel class. Phương thức `join` này sẽ chứa cùng logic mà bạn thường sẽ đặt trong channel authorization closure của bạn. Bạn cũng có thể tận dụng channel model binding:

```php
<?php

namespace App\Broadcasting;

use App\Models\Order;
use App\Models\User;

class OrderChannel
{
    /**
     * Create a new channel instance.
     */
    public function __construct() {}

    /**
     * Authenticate the user's access to the channel.
     */
    public function join(User $user, Order $order): array|bool
    {
        return $user->id === $order->user_id;
    }
}
```

> [!NOTE]
> Giống như nhiều classes khác trong Laravel, channel classes sẽ tự động được resolve bởi [service container](/docs/{{version}}/container). Vì vậy, bạn có thể type-hint bất kỳ dependencies nào cần thiết bởi channel của bạn trong constructor của nó.

<a name="broadcasting-events"></a>
## Broadcasting Events

Once you have defined an event and marked it with the `ShouldBroadcast` interface, you only need to fire the event using the event's dispatch method. The event dispatcher will notice that the event is marked with the `ShouldBroadcast` interface and will queue the event for broadcasting:

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="only-to-others"></a>
### Only to Others

When building an application that utilizes event broadcasting, you may occasionally need to broadcast an event to all subscribers to a given channel except for the current user. You may accomplish this using the `broadcast` helper and the `toOthers` method:

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->toOthers();
```

To better understand when you may want to use the `toOthers` method, let's imagine a task list application where a user may create a new task by entering a task name. To create a task, your application might make a request to a `/task` URL which broadcasts the task's creation and returns a JSON representation of the new task. When your JavaScript application receives the response from the end-point, it might directly insert the new task into its task list like so:

```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

However, remember that we also broadcast the task's creation. If your JavaScript application is also listening for this event in order to add tasks to the task list, you will have duplicate tasks in your list: one from the end-point and one from the broadcast. You may solve this by using the `toOthers` method to instruct the broadcaster to not broadcast the event to the current user.

> [!WARNING]
> Your event must use the `Illuminate\Broadcasting\InteractsWithSockets` trait in order to call the `toOthers` method.

<a name="only-to-others-configuration"></a>
#### Configuration

When you initialize a Laravel Echo instance, a socket ID is assigned to the connection. If you are using a global [Axios](https://github.com/axios/axios) instance to make HTTP requests from your JavaScript application, the socket ID will automatically be attached to every outgoing request as an `X-Socket-ID` header. Then, when you call the `toOthers` method, Laravel will extract the socket ID from the header and instruct the broadcaster to not broadcast to any connections with that socket ID.

If you are not using a global Axios instance, you will need to manually configure your JavaScript application to send the `X-Socket-ID` header with all outgoing requests. You may retrieve the socket ID using the `Echo.socketId` method:

```js
var socketId = Echo.socketId();
```

<a name="customizing-the-connection"></a>
### Customizing the Connection

If your application interacts with multiple broadcast connections and you want to broadcast an event using a broadcaster other than your default, you may specify which connection to push an event to using the `via` method:

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->via('pusher');
```

Alternatively, you may specify the event's broadcast connection by calling the `broadcastVia` method within the event's constructor. However, before doing so, you should ensure that the event class uses the `InteractsWithBroadcasting` trait:

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithBroadcasting;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class OrderShipmentStatusUpdated implements ShouldBroadcast
{
    use InteractsWithBroadcasting;

    /**
     * Create a new event instance.
     */
    public function __construct()
    {
        $this->broadcastVia('pusher');
    }
}
```

<a name="anonymous-events"></a>
### Anonymous Events

Sometimes, you may want to broadcast a simple event to your application's frontend without creating a dedicated event class. To accommodate this, the `Broadcast` facade allows you to broadcast "anonymous events":

```php
Broadcast::on('orders.'.$order->id)->send();
```

The example above will broadcast the following event:

```json
{
    "event": "AnonymousEvent",
    "data": "[]",
    "channel": "orders.1"
}
```

Using the `as` and `with` methods, you may customize the event's name and data:

```php
Broadcast::on('orders.'.$order->id)
    ->as('OrderPlaced')
    ->with($order)
    ->send();
```

The example above will broadcast an event like the following:

```json
{
    "event": "OrderPlaced",
    "data": "{ id: 1, total: 100 }",
    "channel": "orders.1"
}
```

If you would like to broadcast the anonymous event on a private or presence channel, you may utilize the `private` and `presence` methods:

```php
Broadcast::private('orders.'.$order->id)->send();
Broadcast::presence('channels.'.$channel->id)->send();
```

Broadcasting an anonymous event using the `send` method dispatches the event to your application's [queue](/docs/{{version}}/queues) for processing. However, if you would like to broadcast the event immediately, you may use the `sendNow` method:

```php
Broadcast::on('orders.'.$order->id)->sendNow();
```

To broadcast the event to all channel subscribers except the currently authenticated user, you can invoke the `toOthers` method:

```php
Broadcast::on('orders.'.$order->id)
    ->toOthers()
    ->send();
```

<a name="rescuing-broadcasts"></a>
### Rescuing Broadcasts

When your application's queue server is unavailable or Laravel encounters an error while broadcasting an event, an exception is thrown that typically causes the end user to see an application error. Since event broadcasting is often supplementary to your application's core functionality, you can prevent these exceptions from disrupting the user experience by implementing the `ShouldRescue` interface on your events.

Events that implement the `ShouldRescue` interface automatically utilize Laravel's [rescue helper function](/docs/{{version}}/helpers#method-rescue) during broadcast attempts. This helper catches any exceptions, reports them to your application's exception handler for logging, and allows the application to continue executing normally without interrupting the user's workflow:

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Broadcasting\ShouldRescue;

class ServerCreated implements ShouldBroadcast, ShouldRescue
{
    // ...
}
```

<a name="receiving-broadcasts"></a>
## Receiving Broadcasts

<a name="listening-for-events"></a>
### Listening for Events

Once you have [installed and instantiated Laravel Echo](#client-side-installation), you are ready to start listening for events that are broadcast from your Laravel application. First, use the `channel` method to retrieve an instance of a channel, then call the `listen` method to listen for a specified event:

```js
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

If you would like to listen for events on a private channel, use the `private` method instead. You may continue to chain calls to the `listen` method to listen for multiple events on a single channel:

```js
Echo.private(`orders.${this.order.id}`)
    .listen(/* ... */)
    .listen(/* ... */)
    .listen(/* ... */);
```

<a name="stop-listening-for-events"></a>
#### Stop Listening for Events

If you would like to stop listening to a given event without [leaving the channel](#leaving-a-channel), you may use the `stopListening` method:

```js
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated');
```

<a name="leaving-a-channel"></a>
### Leaving a Channel

To leave a channel, you may call the `leaveChannel` method on your Echo instance:

```js
Echo.leaveChannel(`orders.${this.order.id}`);
```

If you would like to leave a channel and also its associated private and presence channels, you may call the `leave` method:

```js
Echo.leave(`orders.${this.order.id}`);
```
<a name="namespaces"></a>
### Namespaces

You may have noticed in the examples above that we did not specify the full `App\Events` namespace for the event classes. This is because Echo will automatically assume the events are located in the `App\Events` namespace. However, you may configure the root namespace when you instantiate Echo by passing a `namespace` configuration option:

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    namespace: 'App.Other.Namespace'
});
```

Alternatively, you may prefix event classes with a `.` when subscribing to them using Echo. This will allow you to always specify the fully-qualified class name:

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        // ...
    });
```

<a name="using-react-or-vue"></a>
### Using React, Vue, or Svelte

Laravel Echo includes React, Vue, and Svelte hooks that make it painless to listen for events. To get started, invoke the `useEcho` hook, which is used to listen for private events. The `useEcho` hook will automatically leave channels when the consuming component is unmounted:

```js tab=React
import { useEcho } from "@laravel/echo-react";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
</script>
```

```svelte tab=Svelte
<script>
import { useEcho } from "@laravel/echo-svelte";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
</script>
```

You may listen to multiple events by providing an array of events to `useEcho`:

```js
useEcho(
    `orders.${orderId}`,
    ["OrderShipmentStatusUpdated", "OrderShipped"],
    (e) => {
        console.log(e.order);
    },
);
```

You may also specify the shape of the broadcast event payload data, providing greater type safety and editing convenience:

```ts
type OrderData = {
    order: {
        id: number;
        user: {
            id: number;
            name: string;
        };
        created_at: string;
    };
};

useEcho<OrderData>(`orders.${orderId}`, "OrderShipmentStatusUpdated", (e) => {
    console.log(e.order.id);
    console.log(e.order.user.id);
});
```

The `useEcho` hook will automatically leave channels when the consuming component is unmounted; however, you may utilize the returned functions to manually stop / start listening to channels programmatically when necessary:

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { leaveChannel, leave, stopListening, listen } = useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);

// Stop listening without leaving channel...
stopListening();

// Start listening again...
listen();

// Leave channel...
leaveChannel();

// Leave a channel and also its associated private and presence channels...
leave();
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { leaveChannel, leave, stopListening, listen } = useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);

// Stop listening without leaving channel...
stopListening();

// Start listening again...
listen();

// Leave channel...
leaveChannel();

// Leave a channel and also its associated private and presence channels...
leave();
</script>
```

```svelte tab=Svelte
<script>
import { useEcho } from "@laravel/echo-svelte";

const { leaveChannel, leave, stopListening, listen } = useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);

// Stop listening without leaving channel...
stopListening();

// Start listening again...
listen();

// Leave channel...
leaveChannel();

// Leave a channel and also its associated private and presence channels...
leave();
</script>
```

<a name="react-vue-connecting-to-public-channels"></a>
#### Connecting to Public Channels

To connect to a public channel, you may use the `useEchoPublic` hook:

```js tab=React
import { useEchoPublic } from "@laravel/echo-react";

useEchoPublic("posts", "PostPublished", (e) => {
    console.log(e.post);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoPublic } from "@laravel/echo-vue";

useEchoPublic("posts", "PostPublished", (e) => {
    console.log(e.post);
});
</script>
```

```svelte tab=Svelte
<script>
import { useEchoPublic } from "@laravel/echo-svelte";

useEchoPublic("posts", "PostPublished", (e) => {
    console.log(e.post);
});
</script>
```

<a name="react-vue-connecting-to-presence-channels"></a>
#### Connecting to Presence Channels

To connect to a presence channel, you may use the `useEchoPresence` hook:

```js tab=React
import { useEchoPresence } from "@laravel/echo-react";

useEchoPresence("posts", "PostPublished", (e) => {
    console.log(e.post);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoPresence } from "@laravel/echo-vue";

useEchoPresence("posts", "PostPublished", (e) => {
    console.log(e.post);
});
</script>
```

```svelte tab=Svelte
<script>
import { useEchoPresence } from "@laravel/echo-svelte";

useEchoPresence("posts", "PostPublished", (e) => {
    console.log(e.post);
});
</script>
```

<a name="react-vue-connection-status"></a>
#### Connection Status

You may retrieve the current WebSocket connection status using the `useConnectionStatus` hook, which provides reactive status that automatically updates when the connection state changes:

```js tab=React
import { useConnectionStatus } from "@laravel/echo-react";

function ConnectionIndicator() {
    const status = useConnectionStatus();

    return <div>Connection: {status}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useConnectionStatus } from "@laravel/echo-vue";

const status = useConnectionStatus();
</script>

<template>
    <div>Connection: {{ status }}</div>
</template>
```

```svelte tab=Svelte
<script>
import { useConnectionStatus } from "@laravel/echo-svelte";

const status = useConnectionStatus();
</script>

<div>Connection: {status()}</div>
```

The possible status values are:

<div class="content-list" markdown="1">

- `connected` - Successfully connected to the WebSocket server.
- `connecting` - Initial connection attempt in progress.
- `reconnecting` - Attempting to reconnect after a disconnection.
- `disconnected` - Not connected and not attempting to reconnect.
- `failed` - Connection failed and won't retry.

</div>

<a name="react-vue-socket-id"></a>
#### Socket ID

You may retrieve the current WebSocket socket ID using the `useSocketId` hook, which provides a reactive value that automatically updates when the connection reconnects with a new socket ID:

```js tab=React
import { useSocketId } from "@laravel/echo-react";

function SocketIndicator() {
    const socketId = useSocketId();

    return <div>Socket ID: {socketId}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useSocketId } from "@laravel/echo-vue";

const socketId = useSocketId();
</script>

<template>
    <div>Socket ID: {{ socketId }}</div>
</template>
```

```svelte tab=Svelte
<script>
import { useSocketId } from "@laravel/echo-svelte";

const socketId = useSocketId();
</script>

<div>Socket ID: {socketId()}</div>
```

<a name="presence-channels"></a>
## Presence Channels

Presence channels build on the security of private channels while exposing the additional feature of awareness of who is subscribed to the channel. This makes it easy to build powerful, collaborative application features such as notifying users when another user is viewing the same page or listing the inhabitants of a chat room.

<a name="authorizing-presence-channels"></a>
### Authorizing Presence Channels

All presence channels are also private channels; therefore, users must be [authorized to access them](#authorizing-channels). However, when defining authorization callbacks for presence channels, you will not return `true` if the user is authorized to join the channel. Instead, you should return an array of data about the user.

The data returned by the authorization callback will be made available to the presence channel event listeners in your JavaScript application. If the user is not authorized to join the presence channel, you should return `false` or `null`:

```php
use App\Models\User;

Broadcast::channel('chat.{roomId}', function (User $user, int $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

<a name="joining-presence-channels"></a>
### Joining Presence Channels

To join a presence channel, you may use Echo's `join` method. The `join` method will return a `PresenceChannel` implementation which, along with exposing the `listen` method, allows you to subscribe to the `here`, `joining`, and `leaving` events.

```js
Echo.join(`chat.${roomId}`)
    .here((users) => {
        // ...
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    })
    .error((error) => {
        console.error(error);
    });
```

The `here` callback will be executed immediately once the channel is joined successfully, and will receive an array containing the user information for all of the other users currently subscribed to the channel. The `joining` method will be executed when a new user joins a channel, while the `leaving` method will be executed when a user leaves the channel. The `error` method will be executed when the authentication endpoint returns an HTTP status code other than 200 or if there is a problem parsing the returned JSON.

<a name="broadcasting-to-presence-channels"></a>
### Broadcasting to Presence Channels

Presence channels may receive events just like public or private channels. Using the example of a chatroom, we may want to broadcast `NewMessage` events to the room's presence channel. To do so, we'll return an instance of `PresenceChannel` from the event's `broadcastOn` method:

```php
/**
 * Get the channels the event should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(): array
{
    return [
        new PresenceChannel('chat.'.$this->message->room_id),
    ];
}
```

As with other events, you may use the `broadcast` helper and the `toOthers` method to exclude the current user from receiving the broadcast:

```php
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```

As typical of other types of events, you may listen for events sent to presence channels using Echo's `listen` method:

```js
Echo.join(`chat.${roomId}`)
    .here(/* ... */)
    .joining(/* ... */)
    .leaving(/* ... */)
    .listen('NewMessage', (e) => {
        // ...
    });
```

<a name="model-broadcasting"></a>
## Model Broadcasting

> [!WARNING]
> Before reading the following documentation about model broadcasting, we recommend you become familiar with the general concepts of Laravel's model broadcasting services as well as how to manually create and listen to broadcast events.

It is common to broadcast events when your application's [Eloquent models](/docs/{{version}}/eloquent) are created, updated, or deleted. Of course, this can easily be accomplished by manually [defining custom events for Eloquent model state changes](/docs/{{version}}/eloquent#events) and marking those events with the `ShouldBroadcast` interface.

However, if you are not using these events for any other purposes in your application, it can be cumbersome to create event classes for the sole purpose of broadcasting them. To remedy this, Laravel allows you to indicate that an Eloquent model should automatically broadcast its state changes.

To get started, your Eloquent model should use the `Illuminate\Database\Eloquent\BroadcastsEvents` trait. In addition, the model should define a `broadcastOn` method, which will return an array of channels that the model's events should broadcast on:

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Database\Eloquent\BroadcastsEvents;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    use BroadcastsEvents, HasFactory;

    /**
     * Get the user that the post belongs to.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the channels that model events should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>
     */
    public function broadcastOn(string $event): array
    {
        return [$this, $this->user];
    }
}
```

Once your model includes this trait and defines its broadcast channels, it will begin automatically broadcasting events when a model instance is created, updated, deleted, trashed, or restored.

In addition, you may have noticed that the `broadcastOn` method receives a string `$event` argument. This argument contains the type of event that has occurred on the model and will have a value of `created`, `updated`, `deleted`, `trashed`, or `restored`. By inspecting the value of this variable, you may determine which channels (if any) the model should broadcast to for a particular event:

```php
/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<string, array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>>
 */
public function broadcastOn(string $event): array
{
    return match ($event) {
        'deleted' => [],
        default => [$this, $this->user],
    };
}
```

<a name="customizing-model-broadcasting-event-creation"></a>
#### Customizing Model Broadcasting Event Creation

Occasionally, you may wish to customize how Laravel creates the underlying model broadcasting event. You may accomplish this by defining a `newBroadcastableEvent` method on your Eloquent model. This method should return an `Illuminate\Database\Eloquent\BroadcastableModelEventOccurred` instance:

```php
use Illuminate\Database\Eloquent\BroadcastableModelEventOccurred;

/**
 * Create a new broadcastable model event for the model.
 */
protected function newBroadcastableEvent(string $event): BroadcastableModelEventOccurred
{
    return (new BroadcastableModelEventOccurred(
        $this, $event
    ))->dontBroadcastToCurrentUser();
}
```

<a name="model-broadcasting-conventions"></a>
### Model Broadcasting Conventions

<a name="model-broadcasting-channel-conventions"></a>
#### Channel Conventions

As you may have noticed, the `broadcastOn` method in the model example above did not return `Channel` instances. Instead, Eloquent models were returned directly. If an Eloquent model instance is returned by your model's `broadcastOn` method (or is contained in an array returned by the method), Laravel will automatically instantiate a private channel instance for the model using the model's class name and primary key identifier as the channel name.

So, an `App\Models\User` model with an `id` of `1` would be converted into an `Illuminate\Broadcasting\PrivateChannel` instance with a name of `App.Models.User.1`. Of course, in addition to returning Eloquent model instances from your model's `broadcastOn` method, you may return complete `Channel` instances in order to have full control over the model's channel names:

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(string $event): array
{
    return [
        new PrivateChannel('user.'.$this->id)
    ];
}
```

If you plan to explicitly return a channel instance from your model's `broadcastOn` method, you may pass an Eloquent model instance to the channel's constructor. When doing so, Laravel will use the model channel conventions discussed above to convert the Eloquent model into a channel name string:

```php
return [new Channel($this->user)];
```

If you need to determine the channel name of a model, you may call the `broadcastChannel` method on any model instance. For example, this method returns the string `App.Models.User.1` for an `App\Models\User` model with an `id` of `1`:

```php
$user->broadcastChannel();
```

<a name="model-broadcasting-event-conventions"></a>
#### Event Conventions

Since model broadcast events are not associated with an "actual" event within your application's `App\Events` directory, they are assigned a name and a payload based on conventions. Laravel's convention is to broadcast the event using the class name of the model (not including the namespace) and the name of the model event that triggered the broadcast.

So, for example, an update to the `App\Models\Post` model would broadcast an event to your client-side application as `PostUpdated` with the following payload:

```json
{
    "model": {
        "id": 1,
        "title": "My first post"
        ...
    },
    ...
    "socket": "someSocketId"
}
```

The deletion of the `App\Models\User` model would broadcast an event named `UserDeleted`.

If you would like, you may define a custom broadcast name and payload by adding a `broadcastAs` and `broadcastWith` method to your model. These methods receive the name of the model event / operation that is occurring, allowing you to customize the event's name and payload for each model operation. If `null` is returned from the `broadcastAs` method, Laravel will use the model broadcasting event name conventions discussed above when broadcasting the event:

```php
/**
 * The model event's broadcast name.
 */
public function broadcastAs(string $event): string|null
{
    return match ($event) {
        'created' => 'post.created',
        default => null,
    };
}

/**
 * Get the data to broadcast for the model.
 *
 * @return array<string, mixed>
 */
public function broadcastWith(string $event): array
{
    return match ($event) {
        'created' => ['title' => $this->title],
        default => ['model' => $this],
    };
}
```

<a name="listening-for-model-broadcasts"></a>
### Listening for Model Broadcasts

Once you have added the `BroadcastsEvents` trait to your model and defined your model's `broadcastOn` method, you are ready to start listening for broadcasted model events within your client-side application. Before getting started, you may wish to consult the complete documentation on [listening for events](#listening-for-events).

First, use the `private` method to retrieve an instance of a channel, then call the `listen` method to listen for a specified event. Typically, the channel name given to the `private` method should correspond to Laravel's [model broadcasting conventions](#model-broadcasting-conventions).

Once you have obtained a channel instance, you may use the `listen` method to listen for a particular event. Since model broadcast events are not associated with an "actual" event within your application's `App\Events` directory, the [event name](#model-broadcasting-event-conventions) must be prefixed with a `.` to indicate it does not belong to a particular namespace. Each model broadcast event has a `model` property which contains all of the broadcastable properties of the model:

```js
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.UserUpdated', (e) => {
        console.log(e.model);
    });
```

<a name="model-broadcasts-with-react-or-vue"></a>
#### Using React, Vue, or Svelte

If you are using React, Vue, or Svelte, you may use Laravel Echo's included `useEchoModel` hook to easily listen for model broadcasts:

```js tab=React
import { useEchoModel } from "@laravel/echo-react";

useEchoModel("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoModel } from "@laravel/echo-vue";

useEchoModel("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model);
});
</script>
```

```svelte tab=Svelte
<script>
import { useEchoModel } from "@laravel/echo-svelte";

useEchoModel("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model);
});
</script>
```

You may also specify the shape of the model event payload data, providing greater type safety and editing convenience:

```ts
type User = {
    id: number;
    name: string;
    email: string;
};

useEchoModel<User, "App.Models.User">("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model.id);
    console.log(e.model.name);
});
```

<a name="client-events"></a>
## Client Events

> [!NOTE]
> When using [Pusher Channels](https://pusher.com/channels), you must enable the "Client Events" option in the "App Settings" section of your [application dashboard](https://dashboard.pusher.com/) in order to send client events.

Sometimes you may wish to broadcast an event to other connected clients without hitting your Laravel application at all. This can be particularly useful for things like "typing" notifications, where you want to alert users of your application that another user is typing a message on a given screen.

To broadcast client events, you may use Echo's `whisper` method:

```js tab=JavaScript
Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        name: this.user.name
    });
```

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().whisper('typing', { name: user.name });
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().whisper('typing', { name: user.name });
</script>
```

```svelte tab=Svelte
<script>
import { useEcho } from "@laravel/echo-svelte";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().whisper('typing', { name: user.name });
</script>
```

To listen for client events, you may use the `listenForWhisper` method:

```js tab=JavaScript
Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().listenForWhisper('typing', (e) => {
    console.log(e.name);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().listenForWhisper('typing', (e) => {
    console.log(e.name);
});
</script>
```

```svelte tab=Svelte
<script>
import { useEcho } from "@laravel/echo-svelte";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().listenForWhisper('typing', (e) => {
    console.log(e.name);
});
</script>
```

<a name="notifications"></a>
## Notifications

By pairing event broadcasting with [notifications](/docs/{{version}}/notifications), your JavaScript application may receive new notifications as they occur without needing to refresh the page. Before getting started, be sure to read over the documentation on using [the broadcast notification channel](/docs/{{version}}/notifications#broadcast-notifications).

Once you have configured a notification to use the broadcast channel, you may listen for the broadcast events using Echo's `notification` method. Remember, the channel name should match the class name of the entity receiving the notifications:

```js tab=JavaScript
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

```js tab=React
import { useEchoModel } from "@laravel/echo-react";

const { channel } = useEchoModel('App.Models.User', userId);

channel().notification((notification) => {
    console.log(notification.type);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoModel } from "@laravel/echo-vue";

const { channel } = useEchoModel('App.Models.User', userId);

channel().notification((notification) => {
    console.log(notification.type);
});
</script>
```

```svelte tab=Svelte
<script>
import { useEchoModel } from "@laravel/echo-svelte";

const { channel } = useEchoModel('App.Models.User', userId);

channel().notification((notification) => {
    console.log(notification.type);
});
</script>
```

In this example, all notifications sent to `App\Models\User` instances via the `broadcast` channel would be received by the callback. A channel authorization callback for the `App.Models.User.{id}` channel is included in your application's `routes/channels.php` file.

<a name="stop-listening-for-notifications"></a>
#### Stop Listening for Notifications

If you would like to stop listening to notifications without [leaving the channel](#leaving-a-channel), you may use the `stopListeningForNotification` method:

```js
const callback = (notification) => {
    console.log(notification.type);
}

// Start listening...
Echo.private(`App.Models.User.${userId}`)
    .notification(callback);

// Stop listening (callback must be the same)...
Echo.private(`App.Models.User.${userId}`)
    .stopListeningForNotification(callback);
```
