# Quy Tắc Dịch Thuật Laravel Documentation - Tiếng Việt

## 📋 Tổng quan

Document này quy định các nguyên tắc và hướng dẫn khi dịch tài liệu Laravel Framework sang tiếng Việt để đảm bảo tính nhất quán, chính xác và dễ hiểu.

## 🎯 Mục tiêu

- Đảm bảo tính nhất quán về thuật ngữ và phong cách
- Giữ nguyên tính chính xác của nội dung kỹ thuật
- Tạo ra bản dịch tự nhiên, dễ hiểu cho người Việt
- Duy trì cấu trúc và format của bản gốc

## 🔤 Nguyên tắc chung

### 1. Thuật ngữ kỹ thuật

**GIỮ NGUYÊN TIẾNG ANH:**
- Tên framework, library, package: Laravel, PHP, Composer, npm, etc.
- Tên commands: `php artisan`, `composer install`, `npm run dev`
- Tên file và đường dẫn: `composer.json`, `app/Http/Controllers`
- Tên functions, classes, methods: `Route::get()`, `User::find()`
- Tên database terms: Table, Column, Index, Migration
- Tên HTTP terms: GET, POST, PUT, DELETE, Request, Response
- Tên Laravel-specific terms: Eloquent, Blade, Artisan, Facade, Middleware, etc.
- Tên programming concepts: Array, String, Integer, Boolean, Object, Class, etc.
- Tên tools và services: GitHub, Git, Docker, Redis, MySQL, etc.
- Tên abbreviations: API, HTTP, HTTPS, JSON, XML, SQL, CLI, etc.

**DỊCH SANG TIẾNG VIỆT:**
- Documentation → Tài liệu
- Configuration → Cấu hình
- Installation → Cài đặt
- Deployment → Triển khai
- Authentication → Xác thực
- Authorization → Phân quyền
- Database → Cơ sở dữ liệu
- Routing → Định tuyến
- Middleware → Phần mềm trung gian (hoặc giữ nguyên "Middleware")
- Controller → Bộ điều khiển (hoặc giữ nguyên "Controller")
- View → Giao diện / View
- Model → Mô hình / Model
- Migration → Di chuyển / Migration
- Seed → Dữ liệu mẫu / Seed
- Queue → Hàng đợi
- Cache → Bộ nhớ đệm
- Event → Sự kiện
- Listener → Bộ lắng nghe
- Provider → Nhà cung cấp
- Service Provider → Nhà cung cấp dịch vụ

### 2. Phong cách dịch thuật

**Sử dụng ngôn ngữ:**
- Dùng tiếng Việt chuẩn, trang trọng
- Tránh dùng từ lóng, tiếng địa phương
- Dùng cấu trúc câu tự nhiên theo tiếng Việt
- Tránh dịch từng từ (word-for-word translation)

**Ví dụ:**
- ❌ "Laravel là một web application framework với expressive, elegant syntax."
- ✅ "Laravel là một framework ứng dụng web với cú pháp rõ ràng và tinh tế."

### 3. Định dạng và Code

**GIỮ NGUYÊN:**
- Tất cả code blocks (```php, ```bash, etc.)
- Tất cả inline code (`code`)
- Tất cả command line examples
- Tất cả file paths và URLs
- Tất cả placeholders (`{{ variable }}`)
- Tất cả HTML tags và attributes

**DỊCH:**
- Các text description trong code comments (nếu cần thiết)
- Các text trong UI elements descriptions

### 4. Links và References

**GIỮ NGUYÊN:**
- Tất cả external links (https://laravel.com/docs, etc.)
- Tất cả internal links giữa các trang docs
- Tất cả anchor links (#section-name)

**CẦN CHÚ Ý:**
- Nếu link trỏ đến section đã dịch, update anchor text sang tiếng Việt
- Nếu link trỏ đến external resource, giữ nguyên

### 5. Tên file và cấu trúc

**GIỮ NGUYÊN:**
- Tên file markdown (ví dụ: `installation.md`, `routing.md`)
- Cấu trúc thư mục
- Front matter trong markdown (nếu có)

**TẠO MỚI:**
- File dịch tiếng Việt có thể đặt tên tương ứng: `installation-vi.md` hoặc dùng cấu trúc thư mục riêng

## 📝 Quy trình dịch thuật

### Bước 1: Chuẩn bị
1. Đọc file gốc để hiểu nội dung
2. Tìm các term và concept cần chú ý
3. Chuẩn bị resources (dictionary, glossary, reference docs)

### Bước 2: Dịch thuật
1. Dịch từng section theo thứ tự
2. Giữ nguyên format, headings, lists
3. Đảm bảo code blocks không bị thay đổi
4. Check spelling và grammar

### Bước 3: Review
1. Đọc lại bản dịch để đảm bảo tự nhiên
2. Check tính nhất quán về thuật ngữ
3. Verify các links và references
4. Test code examples (nếu có thể)

### Bước 4: Validation
1. So sánh với bản gốc để đảm bảo không bỏ sót nội dung
2. Check format và structure
3. Verify technical accuracy

## 🛠️ Công cụ hỗ trợ

### Recommended Tools:
- **VS Code** với extension:
  - Markdown All in One
  - Code Spell Checker
  - Vietnamese Language Pack
- **DeepL** hoặc **Google Translate** - cho reference (không dùng trực tiếp)
- **Laravel Official Docs** - để cross-reference terminology
- **Glossary** - term list riêng cho Laravel

### Dictionary Resources:
- Laravel Glossary (nếu có)
- Vietnamese IT Dictionary
- GitHub Laravel Docs (để xem cách community dịch)

## 📊 Checklist cho từng file

- [ ] Đọc và hiểu nội dung file gốc
- [ ] Xác định các term cần giữ nguyên tiếng Anh
- [ ] Dịch nội dung chính (heading, paragraphs)
- [ ] Giữ nguyên code blocks và examples
- [ ] Check và update links
- [ ] Review format và structure
- [ ] Spell check tiếng Việt
- [ ] Technical accuracy check
- [ ] Final review

## 🚫 Những điều cần tránh

1. **KHÔNG dịch thuật máy (Machine Translation) trực tiếp** - chỉ dùng作为 reference
2. **KHÔNG thay đổi code examples** - giữ nguyên 100%
3. **KHÔNG bỏ qua warnings và notes** - dịch đầy đủ
4. **KHÔNG thay đổi cấu trúc document** - giữ nguyên headings, sections
5. **KHÔNG thêm nội dung riêng** - chỉ dịch những gì có trong bản gốc
6. **KHÔNG dịch tên người, tên project** - giữ nguyên

## 📚 Term List (Glossary)

### Laravel Core Terms:
| Tiếng Anh | Tiếng Việt | Ghi chú |
|-----------|------------|---------|
| Laravel | Laravel | Giữ nguyên |
| Artisan | Artisan | Giữ nguyên |
| Eloquent | Eloquent | Giữ nguyên |
| Blade | Blade | Giữ nguyên |
| Facade | Facade | Giữ nguyên |
| Middleware | Middleware | Có thể dịch "Phần mềm trung gian" |
| Service Container | Service Container | Giữ nguyên hoặc "Container dịch vụ" |
| Service Provider | Service Provider | "Nhà cung cấp dịch vụ" |
| Route | Route | Giữ nguyên hoặc "Định tuyến" |
| Controller | Controller | Giữ nguyên hoặc "Bộ điều khiển" |
| Model | Model | Giữ nguyên hoặc "Mô hình" |
| View | View | Giữ nguyên hoặc "Giao diện" |
| Migration | Migration | Giữ nguyên hoặc "Di chuyển" |
| Seeder | Seeder | Giữ nguyên hoặc "Dữ liệu mẫu" |
| Factory | Factory | Giữ nguyên hoặc "Factory" |
| Request | Request | Giữ nguyên hoặc "Yêu cầu" |
| Response | Response | Giữ nguyên hoặc "Phản hồi" |

### General Technical Terms:
| Tiếng Anh | Tiếng Việt |
|-----------|------------|
| Framework | Framework |
| Package | Package / Gói |
| Library | Library / Thư viện |
| Dependency | Dependency / Phụ thuộc |
| Configuration | Cấu hình |
| Environment | Environment / Môi trường |
| Database | Database / Cơ sở dữ liệu |
| Authentication | Xác thực |
| Authorization | Phân quyền |
| Session | Session / Phiên làm việc |
| Cookie | Cookie |
| Cache | Cache / Bộ nhớ đệm |
| Queue | Queue / Hàng đợi |
| Event | Event / Sự kiện |
| Listener | Listener / Bộ lắng nghe |
| Broadcast | Broadcast / Phát sóng |
| API | API |
| HTTP | HTTP |
| HTTPS | HTTPS |
| JSON | JSON |
| XML | XML |
| CLI | CLI |
| SSH | SSH |

## 🤝 Contributing

Khi contribute bản dịch:
1. Follow quy tắc trong document này
2. Pull request description phải rõ ràng về những gì đã dịch
3. Tag reviewers để review
4. Sẵn sàng nhận feedback và chỉnh sửa

## 📞 Liên hệ

Nếu có thắc mắc về quy tắc dịch thuật, hãy:
- Mở issue trên repository
- Liên hệ maintainer
- Tham khảo discussion section

---

**Version:** 1.0
**Last Updated:** 2026-06-11
**Maintainer:** thienbd203