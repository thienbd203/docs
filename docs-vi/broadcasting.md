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

Sau khi bạn đã định nghĩa một event và đánh dấu nó với interface `ShouldBroadcast`, bạn chỉ cần fire event sử dụng dispatch method của event. Event dispatcher sẽ nhận thấy rằng event được đánh dấu với interface `ShouldBroadcast` và sẽ queue event để broadcasting:

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="only-to-others"></a>
### Only to Others

Khi xây dựng một ứng dụng sử dụng event broadcasting, bạn có thể đôi khi cần broadcast một event đến tất cả subscribers của một channel nhất định ngoại trừ current user. Bạn có thể thực hiện điều này sử dụng helper `broadcast` và phương thức `toOthers`:

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->toOthers();
```

Để hiểu rõ hơn khi nào bạn có thể muốn sử dụng phương thức `toOthers`, hãy tưởng tượng một ứng dụng task list nơi một user có thể tạo một task mới bằng cách nhập tên task. Để tạo một task, ứng dụng của bạn có thể thực hiện một request đến URL `/task` broadcast việc tạo task và trả về một JSON representation của task mới. Khi ứng dụng JavaScript của bạn nhận response từ end-point, nó có thể trực tiếp chèn task mới vào task list của nó như sau:

```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

Tuy nhiên, hãy nhớ rằng chúng ta cũng broadcast việc tạo task. Nếu ứng dụng JavaScript của bạn cũng đang listen cho event này để thêm tasks vào task list, bạn sẽ có các tasks trùng lặp trong list của bạn: một từ end-point và một từ broadcast. Bạn có thể giải quyết điều này bằng cách sử dụng phương thức `toOthers` để chỉ đạo broadcaster không broadcast event đến current user.

> [!WARNING]
> Your event must use the `Illuminate\Broadcasting\InteractsWithSockets` trait in order to call the `toOthers` method.

<a name="only-to-others-configuration"></a>
#### Configuration

Khi bạn khởi tạo một instance Laravel Echo, một socket ID được gán cho connection. Nếu bạn đang sử dụng một instance [Axios](https://github.com/axios/axios) global để thực hiện HTTP requests từ ứng dụng JavaScript của bạn, socket ID sẽ tự động được đính kèm vào mọi outgoing request làm một header `X-Socket-ID`. Sau đó, khi bạn gọi phương thức `toOthers`, Laravel sẽ extract socket ID từ header và chỉ đạo broadcaster không broadcast đến bất kỳ connections nào có socket ID đó.

Nếu bạn không sử dụng một instance Axios global, bạn sẽ cần cấu hình thủ công ứng dụng JavaScript của bạn để gửi header `X-Socket-ID` với tất cả outgoing requests. Bạn có thể retrieve socket ID sử dụng phương thức `Echo.socketId`:

```js
var socketId = Echo.socketId();
```

<a name="customizing-the-connection"></a>
### Customizing the Connection

Nếu ứng dụng của bạn tương tác với nhiều broadcast connections và bạn muốn broadcast một event sử dụng một broadcaster khác với default của bạn, bạn có thể chỉ định connection nào để push event đến sử dụng phương thức `via`:

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->via('pusher');
```

Ngoài ra, bạn có thể chỉ định broadcast connection của event bằng cách gọi phương thức `broadcastVia` trong constructor của event. Tuy nhiên, trước khi làm điều này, bạn nên đảm bảo rằng class event sử dụng trait `InteractsWithBroadcasting`:

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

Đôi khi, bạn có thể muốn broadcast một event đơn giản đến frontend của ứng dụng của bạn mà không cần tạo một event class chuyên dụng. Để đáp ứng điều này, facade `Broadcast` cho phép bạn broadcast "anonymous events":

```php
Broadcast::on('orders.'.$order->id)->send();
```

Ví dụ trên sẽ broadcast event sau:

```json
{
    "event": "AnonymousEvent",
    "data": "[]",
    "channel": "orders.1"
}
```

Sử dụng các phương thức `as` và `with`, bạn có thể tùy chỉnh tên và dữ liệu của event:

```php
Broadcast::on('orders.'.$order->id)
    ->as('OrderPlaced')
    ->with($order)
    ->send();
```

Ví dụ trên sẽ broadcast một event như sau:

```json
{
    "event": "OrderPlaced",
    "data": "{ id: 1, total: 100 }",
    "channel": "orders.1"
}
```

Nếu bạn muốn broadcast anonymous event trên một private hoặc presence channel, bạn có thể sử dụng các phương thức `private` và `presence`:

```php
Broadcast::private('orders.'.$order->id)->send();
Broadcast::presence('channels.'.$channel->id)->send();
```

Broadcasting một anonymous event sử dụng phương thức `send` dispatches event đến [queue](/docs/{{version}}/queues) của ứng dụng của bạn để xử lý. Tuy nhiên, nếu bạn muốn broadcast event ngay lập tức, bạn có thể sử dụng phương thức `sendNow`:

```php
Broadcast::on('orders.'.$order->id)->sendNow();
```

Để broadcast event đến tất cả channel subscribers ngoại trừ user hiện tại được xác thực, bạn có thể gọi phương thức `toOthers`:

```php
Broadcast::on('orders.'.$order->id)
    ->toOthers()
    ->send();
```

<a name="rescuing-broadcasts"></a>
### Rescuing Broadcasts

Khi queue server của ứng dụng của bạn không khả dụng hoặc Laravel gặp lỗi khi broadcast một event, một exception được ném ra thường gây cho end user thấy một application error. Vì event broadcasting thường là bổ sung cho core functionality của ứng dụng của bạn, bạn có thể ngăn các exceptions này làm gián đoạn user experience bằng cách implement interface `ShouldRescue` trên events của bạn.

Events implement interface `ShouldRescue` tự động sử dụng [rescue helper function](/docs/{{version}}/helpers#method-rescue) của Laravel trong các lần thử broadcast. Helper này bắt bất kỳ exceptions nào, báo cáo chúng cho exception handler của ứng dụng của bạn để logging, và cho phép ứng dụng tiếp tục thực thi bình thường mà không làm gián đoạn workflow của user:

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

Sau khi bạn đã [installed và instantiated Laravel Echo](#client-side-installation), bạn đã sẵn sàng để bắt đầu listen cho các events được broadcast từ ứng dụng Laravel của bạn. Đầu tiên, sử dụng phương thức `channel` để retrieve một instance của một channel, sau đó gọi phương thức `listen` để listen cho một event được chỉ định:

```js
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

Nếu bạn muốn listen cho các events trên một private channel, hãy sử dụng phương thức `private` thay thế. Bạn có thể tiếp tục chain các calls đến phương thức `listen` để listen cho nhiều events trên một channel duy nhất:

```js
Echo.private(`orders.${this.order.id}`)
    .listen(/* ... */)
    .listen(/* ... */)
    .listen(/* ... */);
```

<a name="stop-listening-for-events"></a>
#### Stop Listening for Events

Nếu bạn muốn ngừng listen cho một event nhất định mà không [leaving the channel](#leaving-a-channel), bạn có thể sử dụng phương thức `stopListening`:

```js
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated');
```

<a name="leaving-a-channel"></a>
### Leaving a Channel

Để leave một channel, bạn có thể gọi phương thức `leaveChannel` trên instance Echo của bạn:

```js
Echo.leaveChannel(`orders.${this.order.id}`);
```

Nếu bạn muốn leave một channel và cũng các private và presence channels liên kết của nó, bạn có thể gọi phương thức `leave`:

```js
Echo.leave(`orders.${this.order.id}`);
```
<a name="namespaces"></a>
### Namespaces

Bạn có thể nhận thấy trong các ví dụ trên rằng chúng ta không chỉ định namespace `App\Events` đầy đủ cho các class event. Điều này là do Echo sẽ tự động giả định rằng events nằm trong namespace `App\Events`. Tuy nhiên, bạn có thể cấu hình root namespace khi bạn instantiate Echo bằng cách truyền một tùy chọn cấu hình `namespace`:

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    namespace: 'App.Other.Namespace'
});
```

Ngoài ra, bạn có thể prefix các class event với một `.` khi subscribe chúng sử dụng Echo. Điều này sẽ cho phép bạn luôn chỉ định fully-qualified class name:

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        // ...
    });
```

<a name="using-react-or-vue"></a>
### Using React, Vue, or Svelte

Laravel Echo bao gồm các hooks React, Vue, và Svelte giúp việc listen cho các events trở nên dễ dàng. Để bắt đầu, hãy gọi hook `useEcho`, được sử dụng để listen cho private events. Hook `useEcho` sẽ tự động leave channels khi component tiêu thụ được unmounted:

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

Bạn có thể listen nhiều events bằng cách cung cấp một mảng events cho `useEcho`:

```js
useEcho(
    `orders.${orderId}`,
    ["OrderShipmentStatusUpdated", "OrderShipped"],
    (e) => {
        console.log(e.order);
    },
);
```

Bạn cũng có thể chỉ định shape của dữ liệu broadcast event payload, cung cấp type safety và editing convenience tốt hơn:

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

Hook `useEcho` sẽ tự động leave channels khi component tiêu thụ được unmounted; tuy nhiên, bạn có thể sử dụng các functions được trả về để thủ công stop / start listening to channels theo chương trình khi cần thiết:

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

Để connect đến một public channel, bạn có thể sử dụng hook `useEchoPublic`:

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

Để connect đến một presence channel, bạn có thể sử dụng hook `useEchoPresence`:

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

Bạn có thể retrieve trạng thái kết nối WebSocket hiện tại sử dụng hook `useConnectionStatus`, cung cấp reactive status tự động cập nhật khi trạng thái kết nối thay đổi:

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

Bạn có thể retrieve WebSocket socket ID hiện tại sử dụng hook `useSocketId`, cung cấp một reactive value tự động cập nhật khi kết nối reconnect với một socket ID mới:

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

Presence channels xây dựng trên security của private channels trong khi exposing feature bổ sung của awareness về ai đang subscribe đến channel. Điều này giúp việc xây dựng các tính năng ứng dụng collaborative mạnh mẽ, chẳng hạn như notifying users khi một user khác đang xem cùng một trang hoặc liệt kê các inhabitants của một chat room.

<a name="authorizing-presence-channels"></a>
### Authorizing Presence Channels

Tất cả presence channels cũng là private channels; do đó, users phải được [authorized để truy cập chúng](#authorizing-channels). Tuy nhiên, khi định nghĩa authorization callbacks cho presence channels, bạn sẽ không trả về `true` nếu user được authorize để join channel. Thay vào đó, bạn nên trả về một mảng dữ liệu về user.

Dữ liệu được trả về bởi authorization callback sẽ được cung cấp cho presence channel event listeners trong ứng dụng JavaScript của bạn. Nếu user không được authorize để join presence channel, bạn nên trả về `false` hoặc `null`:

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

Để join một presence channel, bạn có thể sử dụng phương thức `join` của Echo. Phương thức `join` sẽ trả về một implementation `PresenceChannel` mà, cùng với exposing phương thức `listen`, cho phép bạn subscribe đến các events `here`, `joining`, và `leaving`.

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

Callback `here` sẽ được thực thi ngay lập tức một khi channel được join thành công, và sẽ nhận một mảng chứa thông tin user cho tất cả các users khác hiện đang subscribe đến channel. Phương thức `joining` sẽ được thực thi khi một user mới join một channel, trong khi phương thức `leaving` sẽ được thực thi khi một user leave channel. Phương thức `error` sẽ được thực thi khi authentication endpoint trả về một HTTP status code khác 200 hoặc nếu có vấn đề khi parse JSON được trả về.

<a name="broadcasting-to-presence-channels"></a>
### Broadcasting to Presence Channels

Presence channels có thể nhận events giống như public hoặc private channels. Sử dụng ví dụ về một chatroom, chúng ta có thể muốn broadcast các events `NewMessage` đến presence channel của room. Để làm điều này, chúng ta sẽ trả về một instance `PresenceChannel` từ phương thức `broadcastOn` của event:

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

Giống như các events khác, bạn có thể sử dụng helper `broadcast` và phương thức `toOthers` để exclude current user khỏi việc nhận broadcast:

```php
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```

Giống như các types events khác, bạn có thể listen cho các events được gửi đến presence channels sử dụng phương thức `listen` của Echo:

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
> Trước khi đọc tài liệu sau về model broadcasting, chúng tôi khuyên bạn làm quen với các khái niệm chung của các dịch vụ model broadcasting của Laravel cũng như cách thủ công tạo và listen cho broadcast events.

Việc broadcast events khi [Eloquent models](/docs/{{version}}/eloquent) của ứng dụng của bạn được tạo, cập nhật, hoặc xóa là phổ biến. Tất nhiên, điều này có thể dễ dàng thực hiện bằng cách thủ công [defining custom events cho Eloquent model state changes](/docs/{{version}}/eloquent#events) và đánh dấu các events đó với interface `ShouldBroadcast`.

Tuy nhiên, nếu bạn không sử dụng các events này cho bất kỳ mục đích nào khác trong ứng dụng của bạn, việc tạo event classes chỉ để broadcast chúng có thể là cồng kềnh. Để khắc phục điều này, Laravel cho phép bạn chỉ định rằng một Eloquent model nên tự động broadcast các state changes của nó.

Để bắt đầu, Eloquent model của bạn nên sử dụng trait `Illuminate\Database\Eloquent\BroadcastsEvents`. Ngoài ra, model nên định nghĩa một phương thức `broadcastOn`, sẽ trả về một mảng channels mà các events của model nên broadcast trên:

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

Sau khi model của bạn bao gồm trait này và định nghĩa các broadcast channels của nó, nó sẽ bắt đầu tự động broadcast events khi một model instance được tạo, cập nhật, xóa, trashed, hoặc restored.

Ngoài ra, bạn có thể nhận thấy rằng phương thức `broadcastOn` nhận một đối số string `$event`. Đối số này chứa type của event đã xảy ra trên model và sẽ có giá trị là `created`, `updated`, `deleted`, `trashed`, hoặc `restored`. Bằng cách inspect giá trị của biến này, bạn có thể xác định channels nào (nếu có) mà model nên broadcast đến cho một event cụ thể:

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

Đôi khi, bạn có thể muốn tùy chỉnh cách Laravel tạo model broadcasting event bên dưới. Bạn có thể thực hiện điều này bằng cách định nghĩa một phương thức `newBroadcastableEvent` trên Eloquent model của bạn. Phương thức này nên trả về một instance `Illuminate\Database\Eloquent\BroadcastableModelEventOccurred`:

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

Như bạn có thể nhận thấy, phương thức `broadcastOn` trong ví dụ model ở trên không trả về các instances `Channel`. Thay vào đó, các Eloquent models được trả về trực tiếp. Nếu một instance Eloquent model được trả về bởi phương thức `broadcastOn` của model của bạn (hoặc được chứa trong một mảng được trả về bởi phương thức), Laravel sẽ tự động instantiate một private channel instance cho model sử dụng tên class và primary key identifier của model làm tên channel.

Vì vậy, một model `App\Models\User` với `id` là `1` sẽ được chuyển đổi thành một instance `Illuminate\Broadcasting\PrivateChannel` với tên là `App.Models.User.1`. Tất nhiên, ngoài việc trả về các instances Eloquent model từ phương thức `broadcastOn` của model của bạn, bạn có thể trả về các instances `Channel` hoàn chỉnh để có toàn quyền kiểm soát các tên channel của model:

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

Nếu bạn có kế hoạch trả về một channel instance một cách rõ ràng từ phương thức `broadcastOn` của model, bạn có thể truyền một instance Eloquent model vào constructor của channel. Khi làm điều này, Laravel sẽ sử dụng các model channel conventions đã thảo luận ở trên để chuyển đổi Eloquent model thành một chuỗi tên channel:

```php
return [new Channel($this->user)];
```

Nếu bạn cần xác định tên channel của một model, bạn có thể gọi phương thức `broadcastChannel` trên bất kỳ instance model nào. Ví dụ, phương thức này trả về chuỗi `App.Models.User.1` cho một model `App\Models\User` với `id` là `1`:

```php
$user->broadcastChannel();
```

<a name="model-broadcasting-event-conventions"></a>
#### Event Conventions

Vì model broadcast events không được liên kết với một "actual" event trong thư mục `App\Events` của ứng dụng của bạn, chúng được gán một tên và một payload dựa trên conventions. Convention của Laravel là broadcast event sử dụng tên class của model (không bao gồm namespace) và tên của model event đã trigger broadcast.

Vì vậy, ví dụ, một cập nhật cho model `App\Models\Post` sẽ broadcast một event đến ứng dụng client-side của bạn là `PostUpdated` với payload sau:

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

Việc xóa model `App\Models\User` sẽ broadcast một event có tên `UserDeleted`.

Nếu bạn muốn, bạn có thể định nghĩa một broadcast name và payload tùy chỉnh bằng cách thêm các phương thức `broadcastAs` và `broadcastWith` vào model của bạn. Các phương thức này nhận tên của model event / operation đang xảy ra, cho phép bạn tùy chỉnh tên và payload của event cho mỗi model operation. Nếu `null` được trả về từ phương thức `broadcastAs`, Laravel sẽ sử dụng các model broadcasting event name conventions đã thảo luận ở trên khi broadcast event:

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

Sau khi bạn đã thêm trait `BroadcastsEvents` vào model của bạn và định nghĩa phương thức `broadcastOn` của model, bạn đã sẵn sàng để bắt đầu listen cho các model events được broadcast trong ứng dụng client-side của bạn. Trước khi bắt đầu, bạn có thể muốn xem tài liệu hoàn chỉnh về [listening for events](#listening-for-events).

Đầu tiên, sử dụng phương thức `private` để retrieve một instance của một channel, sau đó gọi phương thức `listen` để listen cho một event được chỉ định. Thông thường, tên channel được đưa cho phương thức `private` nên tương ứng với [model broadcasting conventions](#model-broadcasting-conventions) của Laravel.

Sau khi bạn đã có một channel instance, bạn có thể sử dụng phương thức `listen` để listen cho một event cụ thể. Vì model broadcast events không được liên kết với một "actual" event trong thư mục `App\Events` của ứng dụng của bạn, [event name](#model-broadcasting-event-conventions) phải được prefix với một `.` để chỉ định nó không thuộc về một namespace cụ thể. Mỗi model broadcast event có một property `model` chứa tất cả các broadcastable properties của model:

```js
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.UserUpdated', (e) => {
        console.log(e.model);
    });
```

<a name="model-broadcasts-with-react-or-vue"></a>
#### Using React, Vue, or Svelte

Nếu bạn đang sử dụng React, Vue, hoặc Svelte, bạn có thể sử dụng hook `useEchoModel` được bao gồm trong Laravel Echo để dễ dàng listen cho model broadcasts:

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

Bạn cũng có thể chỉ định shape của dữ liệu model event payload, cung cấp type safety và editing convenience tốt hơn:

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

Đôi khi bạn có thể muốn broadcast một event đến các clients được kết nối khác mà không cần hit ứng dụng Laravel của bạn tại tất cả. Điều này có thể đặc biệt hữu ích cho các thứ như "typing" notifications, nơi bạn muốn alert users của ứng dụng của bạn rằng một user khác đang typing một message trên một màn hình nhất định.

Để broadcast client events, bạn có thể sử dụng phương thức `whisper` của Echo:

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

Để listen cho client events, bạn có thể sử dụng phương thức `listenForWhisper`:

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

Bằng cách pairing event broadcasting với [notifications](/docs/{{version}}/notifications), ứng dụng JavaScript của bạn có thể nhận các notifications mới khi chúng xảy ra mà không cần refresh trang. Trước khi bắt đầu, hãy chắc chắn đọc tài liệu về sử dụng [broadcast notification channel](/docs/{{version}}/notifications#broadcast-notifications).

Sau khi bạn đã cấu hình một notification để sử dụng broadcast channel, bạn có thể listen cho các broadcast events sử dụng phương thức `notification` của Echo. Nhớ rằng, tên channel nên khớp với tên class của entity nhận notifications:

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

Trong ví dụ này, tất cả notifications được gửi đến các instances `App\Models\User` qua kênh `broadcast` sẽ được nhận bởi callback. Một channel authorization callback cho channel `App.Models.User.{id}` được bao gồm trong file `routes/channels.php` của ứng dụng của bạn.

<a name="stop-listening-for-notifications"></a>
#### Stop Listening for Notifications

Nếu bạn muốn ngừng listen cho notifications mà không [leaving the channel](#leaving-a-channel), bạn có thể sử dụng phương thức `stopListeningForNotification`:

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
