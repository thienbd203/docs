# Vòng đời Request

- [Giới thiệu](#introduction)
- [Tổng quan về Vòng đời](#lifecycle-overview)
    - [Bước đầu tiên](#first-steps)
    - [HTTP / Console Kernels](#http-console-kernels)
    - [Service Providers](#service-providers)
    - [Routing](#routing)
    - [Hoàn tất](#finishing-up)
- [Tập trung vào Service Providers](#focus-on-service-providers)

<a name="introduction"></a>
## Giới thiệu

Khi sử dụng bất kỳ công cụ nào trong "thế giới thực", bạn sẽ cảm thấy tự tin hơn nếu bạn hiểu cách công cụ đó hoạt động. Phát triển ứng dụng cũng không ngoại lệ. Khi bạn hiểu cách các công cụ phát triển của bạn hoạt động, bạn sẽ cảm thấy thoải mái và tự tin hơn khi sử dụng chúng.

Mục tiêu của tài liệu này là cung cấp cho bạn cái nhìn tổng quan ở mức cao về cách framework Laravel hoạt động. Bằng cách hiểu rõ hơn về framework tổng thể, mọi thứ sẽ bớt "ma thuật" hơn và bạn sẽ tự tin hơn khi xây dựng ứng dụng của mình. Nếu bạn không hiểu tất cả các thuật ngữ ngay lập tức, đừng nản lòng! Chỉ cần cố gắng nắm bắt cơ bản về những gì đang diễn ra, và kiến thức của bạn sẽ phát triển khi bạn khám phá các phần khác của tài liệu.

<a name="lifecycle-overview"></a>
## Tổng quan về Vòng đời

<a name="first-steps"></a>
### Bước đầu tiên

Điểm nhập cho tất cả các request đến ứng dụng Laravel là file `public/index.php`. Tất cả các request đều được hướng đến file này bởi cấu hình web server (Apache / Nginx) của bạn. File `index.php` không chứa nhiều code. Thay vào đó, nó là điểm bắt đầu để tải phần còn lại của framework.

File `index.php` tải định nghĩa autoloader được tạo bởi Composer, sau đó lấy một instance của ứng dụng Laravel từ `bootstrap/app.php`. Hành động đầu tiên mà chính Laravel thực hiện là tạo một instance của ứng dụng / [service container](/docs/{{version}}/container).

<a name="http-console-kernels"></a>
### HTTP / Console Kernels

Tiếp theo, request đến sẽ được gửi đến HTTP kernel hoặc console kernel, sử dụng phương thức `handleRequest` hoặc `handleCommand` của instance ứng dụng, tùy thuộc vào loại request đang vào ứng dụng. Hai kernel này đóng vai trò là vị trí trung tâm mà qua đó tất cả các request đi qua. Hiện tại, hãy chỉ tập trung vào HTTP kernel, là một instance của `Illuminate\Foundation\Http\Kernel`.

HTTP kernel định nghĩa một mảng `bootstrappers` sẽ được chạy trước khi request được thực thi. Các bootstrapper này cấu hình xử lý lỗi, cấu hình logging, [phát hiện môi trường ứng dụng](/docs/{{version}}/configuration#environment-configuration), và thực hiện các tác vụ khác cần được thực hiện trước khi request thực sự được xử lý. Thông thường, các class này xử lý cấu hình nội bộ của Laravel mà bạn không cần lo lắng.

HTTP kernel cũng chịu trách nhiệm chuyển request qua stack middleware của ứng dụng. Các middleware này xử lý việc đọc và ghi [HTTP session](/docs/{{version}}/session), xác định xem ứng dụng có đang ở chế độ bảo trì không, [xác minh CSRF token](/docs/{{version}}/csrf), và nhiều hơn nữa. Chúng ta sẽ nói thêm về những điều này sớm.

Chữ ký phương thức cho phương thức `handle` của HTTP kernel khá đơn giản: nó nhận một `Request` và trả về một `Response`. Hãy coi kernel như một hộp đen lớn đại diện cho toàn bộ ứng dụng của bạn. Cung cấp cho nó các HTTP request và nó sẽ trả về các HTTP response.

<a name="service-providers"></a>
### Service Providers

Một trong những hành động khởi động kernel quan trọng nhất là tải các [service providers](/docs/{{version}}/providers) cho ứng dụng của bạn. Service providers chịu trách nhiệm khởi động tất cả các thành phần khác nhau của framework, chẳng hạn như cơ sở dữ liệu, hàng đợi, xác thực, và các thành phần routing.

Laravel sẽ lặp qua danh sách các provider này và khởi tạo từng cái. Sau khi khởi tạo các provider, phương thức `register` sẽ được gọi trên tất cả các provider. Sau đó, khi tất cả các provider đã được đăng ký, phương thức `boot` sẽ được gọi trên từng provider. Điều này để đảm bảo rằng service providers có thể phụ thuộc vào mọi binding của container được đăng ký và sẵn sàng vào thời điểm phương thức `boot` của chúng được thực thi.

Về cơ bản, mọi tính năng chính mà Laravel cung cấp đều được khởi động và cấu hình bởi một service provider. Vì chúng khởi động và cấu hình rất nhiều tính năng mà framework cung cấp, service providers là khía cạnh quan trọng nhất của toàn bộ quá trình khởi động Laravel.

Mặc dù framework nội bộ sử dụng hàng chục service providers, bạn cũng có tùy chọn để tạo service provider của riêng mình. Bạn có thể tìm thấy danh sách các service provider do người dùng định nghĩa hoặc bên thứ ba mà ứng dụng của bạn đang sử dụng trong file `bootstrap/providers.php`.

<a name="routing"></a>
### Routing

Khi ứng dụng đã được khởi động và tất cả service providers đã được đăng ký, `Request` sẽ được chuyển đến router để phân phối. Router sẽ phân phối request đến một route hoặc controller, cũng như chạy bất kỳ middleware cụ thể cho route nào.

Middleware cung cấp cơ chế thuận tiện để lọc hoặc kiểm tra các HTTP request vào ứng dụng của bạn. Ví dụ, Laravel bao gồm một middleware xác minh xem người dùng của ứng dụng của bạn có được xác thực không. Nếu người dùng không được xác thực, middleware sẽ chuyển hướng người dùng đến màn hình đăng nhập. Tuy nhiên, nếu người dùng đã được xác thực, middleware sẽ cho phép request tiếp tục sâu hơn vào ứng dụng. Một số middleware được gán cho tất cả các route trong ứng dụng, như `PreventRequestsDuringMaintenance`, trong khi một số chỉ được gán cho các route cụ thể hoặc nhóm route. Bạn có thể tìm hiểu thêm về middleware bằng cách đọc [tài liệu middleware đầy đủ](/docs/{{version}}/middleware).

Nếu request đi qua tất cả middleware được gán của route khớp, phương thức route hoặc controller sẽ được thực thi và response được trả về bởi phương thức route hoặc controller sẽ được gửi lại qua chuỗi middleware của route.

<a name="finishing-up"></a>
### Hoàn tất

Khi route hoặc phương thức controller trả về một response, response sẽ đi ngược ra ngoài qua middleware của route, cho phép ứng dụng cơ hội để sửa đổi hoặc kiểm tra response đi ra.

Cuối cùng, khi response đi ngược qua middleware, phương thức `handle` của HTTP kernel trả về đối tượng response cho `handleRequest` của instance ứng dụng, và phương thức này gọi phương thức `send` trên response được trả về. Phương thức `send` gửi nội dung response đến trình duyệt web của người dùng. Chúng ta đã hoàn thành hành trình của mình qua toàn bộ vòng đời request của Laravel!

<a name="focus-on-service-providers"></a>
## Tập trung vào Service Providers

Service providers thực sự là chìa khóa để khởi động một ứng dụng Laravel. Instance ứng dụng được tạo, các service providers được đăng ký, và request được chuyển đến ứng dụng đã khởi động. Nó thực sự đơn giản như vậy!

Việc nắm vững cách một ứng dụng Laravel được xây dựng và khởi động qua service providers là rất có giá trị. Các service provider do người dùng định nghĩa của ứng dụng của bạn được lưu trữ trong thư mục `app/Providers`.

Theo mặc định, `AppServiceProvider` khá trống. Provider này là nơi tuyệt vời để thêm khởi động và binding service container của riêng ứng dụng của bạn. Đối với các ứng dụng lớn, bạn có thể muốn tạo một số service provider, mỗi cái có khởi động chi tiết hơn cho các dịch vụ cụ thể được sử dụng bởi ứng dụng của bạn.
