# Frontend

- [Introduction](#introduction)
- [Using PHP](#using-php)
    - [PHP and Blade](#php-and-blade)
    - [Livewire](#livewire)
    - [Starter Kits](#php-starter-kits)
- [Using React, Svelte, or Vue](#using-react-svelte-or-vue)
    - [Inertia](#inertia)
    - [Starter Kits](#inertia-starter-kits)
- [Bundling Assets](#bundling-assets)

<a name="introduction"></a>
## Introduction

Laravel là một backend framework cung cấp tất cả các tính năng bạn cần để xây dựng các ứng dụng web hiện đại, chẳng hạn như [routing](/docs/{{version}}/routing), [validation](/docs/{{version}}/validation), [caching](/docs/{{version}}/cache), [queues](/docs/{{version}}/queues), [file storage](/docs/{{version}}/filesystem), và hơn thế nữa. Tuy nhiên, chúng tôi tin rằng quan trọng để cung cấp cho các nhà phát triển một trải nghiệm full-stack đẹp, bao gồm các cách tiếp cận mạnh mẽ để xây dựng frontend của ứng dụng.

Có hai cách chính để giải quyết phát triển frontend khi xây dựng một ứng dụng với Laravel, và cách tiếp cận bạn chọn được xác định bởi việc bạn có muốn xây dựng frontend của mình bằng cách tận dụng PHP hay bằng cách sử dụng các frameworks JavaScript như React, Svelte, và Vue. Chúng tôi sẽ thảo luận cả hai tùy chọn này dưới đây để bạn có thể đưa ra quyết định có hiểu biết về cách tiếp cận tốt nhất cho phát triển frontend của ứng dụng.

<a name="using-php"></a>
## Using PHP

<a name="php-and-blade"></a>
### PHP and Blade

Trong quá khứ, hầu hết các ứng dụng PHP hiển thị HTML cho trình duyệt bằng cách sử dụng các HTML templates đơn giản xen kẽ với các câu lệnh PHP `echo` hiển thị dữ liệu được truy xuất từ database trong request:

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

Trong Laravel, cách tiếp cận này để hiển thị HTML vẫn có thể đạt được bằng cách sử dụng [views](/docs/{{version}}/views) và [Blade](/docs/{{version}}/blade). Blade là một ngôn ngữ templating cực kỳ nhẹ cung cấp cú pháp ngắn gọn thuận tiện để hiển thị dữ liệu, lặp qua dữ liệu, và hơn thế nữa:

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

Khi xây dựng các ứng dụng theo cách này, các gửi form và các tương tác trang khác thường nhận một tài liệu HTML hoàn toàn mới từ server và toàn bộ trang được hiển thị lại bởi trình duyệt. Ngay cả ngày nay, nhiều ứng dụng có thể hoàn toàn phù hợp để có frontend được xây dựng theo cách này bằng cách sử dụng các Blade templates đơn giản.

<a name="growing-expectations"></a>
#### Growing Expectations

Tuy nhiên, khi kỳ vọng của người dùng về các ứng dụng web đã trưởng thành, nhiều nhà phát triển đã thấy nhu cầu xây dựng các frontend động hơn với các tương tác cảm giác được hoàn thiện hơn. Nhìn vào điều này, một số nhà phát triển chọn bắt đầu xây dựng frontend của ứng dụng bằng cách sử dụng các frameworks JavaScript như React, Svelte, và Vue.

Những người khác, thích giữ với ngôn ngữ backend mà họ thoải mái, đã phát triển các giải pháp cho phép xây dựng các UI ứng dụng web hiện đại trong khi vẫn chủ yếu sử dụng ngôn ngữ backend của họ. Ví dụ, trong hệ sinh thái [Rails](https://rubyonrails.org/), điều này đã thúc đẩy việc tạo ra các thư viện như [Turbo](https://turbo.hotwired.dev/) [Hotwire](https://hotwired.dev/), và [Stimulus](https://stimulus.hotwired.dev/).

Trong hệ sinh thái Laravel, nhu cầu tạo ra các frontend động, hiện đại bằng cách chủ yếu sử dụng PHP đã dẫn đến việc tạo ra [Laravel Livewire](https://livewire.laravel.com) và [Alpine.js](https://alpinejs.dev/).

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://livewire.laravel.com) là một framework để xây dựng các frontend được hỗ trợ bởi Laravel cảm thấy động, hiện đại và sống động giống như các frontend được xây dựng với các frameworks JavaScript hiện đại như React, Svelte, và Vue.

Khi sử dụng Livewire, bạn sẽ tạo các "components" Livewire hiển thị một phần rời rạc của UI của bạn và phơi bày các phương thức và dữ liệu có thể được gọi và tương tác từ frontend của ứng dụng. Ví dụ, một component "Counter" đơn giản có thể trông như sau:

```php
<?php

use Livewire\Component;

new class extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }
};
?>

<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>

```

Như bạn có thể thấy, Livewire cho phép bạn viết các thuộc tính HTML mới như `wire:click` kết nối frontend và backend của ứng dụng Laravel của bạn. Ngoài ra, bạn có thể hiển thị trạng thái hiện tại của component bằng cách sử dụng các biểu thức Blade đơn giản.

Đối với nhiều người, Livewire đã cách mạng hóa phát triển frontend với Laravel, cho phép họ ở lại trong sự thoải mái của Laravel trong khi xây dựng các ứng dụng web động, hiện đại. Thông thường, các nhà phát triển sử dụng Livewire cũng sẽ sử dụng [Alpine.js](https://alpinejs.dev/) để "rải" JavaScript lên frontend của họ chỉ khi cần thiết, chẳng hạn như để hiển thị một cửa sổ dialog.

Nếu bạn mới với Laravel, chúng tôi khuyến nghị làm quen với cách sử dụng cơ bản của [views](/docs/{{version}}/views) và [Blade](/docs/{{version}}/blade). Sau đó, tham khảo tài liệu [Laravel Livewire chính thức](https://livewire.laravel.com/docs) để tìm hiểu cách đưa ứng dụng của bạn lên cấp tiếp theo với các components Livewire tương tác.

<a name="php-starter-kits"></a>
### Starter Kits

Nếu bạn muốn xây dựng frontend của mình bằng cách sử dụng PHP và Livewire, bạn có thể tận dụng [starter kit Livewire](/docs/{{version}}/starter-kits) của chúng tôi để thúc đẩy phát triển ứng dụng của bạn.

<a name="using-react-svelte-or-vue"></a>
## Using React, Svelte, or Vue

Mặc dù có thể xây dựng các frontend hiện đại bằng cách sử dụng Laravel và Livewire, nhiều nhà phát triển vẫn thích tận dụng sức mạnh của một framework JavaScript như React, Svelte, hoặc Vue. Điều này cho phép các nhà phát triển tận dụng hệ sinh thái phong phú của các packages và công cụ JavaScript có sẵn thông qua NPM.

Tuy nhiên, nếu không có công cụ bổ sung, kết hợp Laravel với React, Svelte, hoặc Vue sẽ khiến chúng ta cần giải quyết nhiều vấn đề phức tạp như routing phía client, hydration dữ liệu, và xác thực. Routing phía client thường được đơn giản hóa bằng cách sử dụng các frameworks React / Svelte / Vue có quan điểm như [Next](https://nextjs.org/) và [Nuxt](https://nuxt.com/); tuy nhiên, hydration dữ liệu và xác thực vẫn là các vấn đề phức tạp và khó giải quyết khi kết hợp một backend framework như Laravel với các frontend frameworks này.

Ngoài ra, các nhà phát triển phải duy trì hai kho lưu trữ mã riêng biệt, thường cần phối hợp bảo trì, phát hành, và triển khai trên cả hai kho lưu trữ. Trong khi các vấn đề này không thể vượt qua, chúng tôi không tin rằng đó là một cách phát triển ứng dụng hiệu quả hoặc thú vị.

<a name="inertia"></a>
### Inertia

May mắn thay, Laravel cung cấp điều tốt nhất của cả hai thế giới. [Inertia](https://inertiajs.com) thu hẹp khoảng cách giữa ứng dụng Laravel của bạn và frontend React, Svelte, hoặc Vue hiện đại của bạn, cho phép bạn xây dựng các frontend hoàn chỉnh, hiện đại bằng cách sử dụng React, Svelte, hoặc Vue trong khi tận dụng các routes và controllers của Laravel cho routing, hydration dữ liệu, và xác thực — tất cả trong một kho lưu trữ mã duy nhất. Với cách tiếp cận này, bạn có thể tận dụng sức mạnh đầy đủ của cả Laravel và React / Svelte / Vue mà không làm tê liệt khả năng của công cụ nào.

Sau khi cài đặt Inertia vào ứng dụng Laravel của bạn, bạn sẽ viết routes và controllers như bình thường. Tuy nhiên, thay vì trả về một Blade template từ controller của bạn, bạn sẽ trả về một trang Inertia:

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): Response
    {
        return Inertia::render('users/show', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Một trang Inertia tương ứng với một component React, Svelte, hoặc Vue, thường được lưu trữ trong thư mục `resources/js/pages` của ứng dụng. Dữ liệu được đưa cho trang thông qua phương thức `Inertia::render` sẽ được sử dụng để hydrate các "props" của component trang:

```jsx
import Layout from '@/layouts/authenticated';
import { Head } from '@inertiajs/react';

export default function Show({ user }) {
    return (
        <Layout>
            <Head title="Welcome" />
            <h1>Welcome</h1>
            <p>Hello {user.name}, welcome to Inertia.</p>
        </Layout>
    )
}
```

Như bạn có thể thấy, Inertia cho phép bạn tận dụng sức mạnh đầy đủ của React, Svelte, hoặc Vue khi xây dựng frontend của bạn, trong khi cung cấp một cầu nối nhẹ giữa backend được hỗ trợ bởi Laravel và frontend được hỗ trợ bởi JavaScript của bạn.

#### Server-Side Rendering

Nếu bạn lo lắng về việc đi sâu vào Inertia vì ứng dụng của bạn yêu cầu server-side rendering, đừng lo lắng. Inertia cung cấp [hỗ trợ server-side rendering](https://inertiajs.com/server-side-rendering). Và, khi triển khai ứng dụng của bạn thông qua [Laravel Cloud](https://cloud.laravel.com) hoặc [Laravel Forge](https://forge.laravel.com), thật dễ dàng để đảm bảo quy trình server-side rendering của Inertia luôn chạy.

<a name="inertia-starter-kits"></a>
### Starter Kits

Nếu bạn muốn xây dựng frontend của mình bằng cách sử dụng Inertia và React / Svelte / Vue, bạn có thể tận dụng [starter kits ứng dụng React, Svelte, hoặc Vue](/docs/{{version}}/starter-kits) của chúng tôi để thúc đẩy phát triển ứng dụng của bạn. Tất cả các starter kits này scaffold luồng xác thực backend và frontend của ứng dụng bằng cách sử dụng Inertia, React / Svelte / Vue, [Tailwind](https://tailwindcss.com), và [Vite](https://vitejs.dev) để bạn có thể bắt đầu xây dựng ý tưởng lớn tiếp theo của mình.

<a name="bundling-assets"></a>
## Bundling Assets

Bất kể bạn chọn phát triển frontend của mình bằng cách sử dụng Blade và Livewire hay React / Svelte / Vue và Inertia, bạn có thể cần bundle CSS của ứng dụng thành các assets sẵn sàng cho production. Tất nhiên, nếu bạn chọn xây dựng frontend của ứng dụng với React, Svelte, hoặc Vue, bạn cũng sẽ cần bundle các components của mình thành các assets JavaScript sẵn sàng cho trình duyệt.

Theo mặc định, Laravel sử dụng [Vite](https://vitejs.dev) để bundle các assets của bạn. Vite cung cấp thời gian xây dựng cực nhanh và Hot Module Replacement (HMR) gần như tức thì trong quá trình phát triển cục bộ. Trong tất cả các ứng dụng Laravel mới, bao gồm cả những ứng dụng sử dụng [starter kits](/docs/{{version}}/starter-kits) của chúng tôi, bạn sẽ tìm thấy một file `vite.config.js` tải plugin Laravel Vite nhẹ của chúng tôi làm cho việc sử dụng Vite với các ứng dụng Laravel trở nên thú vị.

Cách nhanh nhất để bắt đầu với Laravel và Vite là bắt đầu phát triển ứng dụng của bạn bằng cách sử dụng [starter kits ứng dụng](/docs/{{version}}/starter-kits) của chúng tôi, thúc đẩy ứng dụng của bạn bằng cách cung cấp scaffold xác thực frontend và backend.

> [!NOTE]
> Để biết tài liệu chi tiết hơn về việc sử dụng Vite với Laravel, hãy xem tài liệu [đặc biệt của chúng tôi về việc bundle và biên dịch các assets](/docs/{{version}}/vite).
