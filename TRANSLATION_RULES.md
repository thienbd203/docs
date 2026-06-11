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

**GIỮ NGUYÊN TIẾNG ANH (Technical Terms):**
- Tên framework, library, package: Laravel, PHP, Composer, npm, etc.
- Tên commands: `php artisan`, `composer install`, `npm run dev`
- Tên file và đường dẫn: `composer.json`, `app/Http/Controllers`
- Tên functions, classes, methods: `Route::get()`, `User::find()`
- Tên Laravel-specific terms: Eloquent, Blade, Artisan, Facade, Middleware, Controller, Model, View, Route, etc.
- Tên database terms: Table, Column, Index, Migration, Seeder, Factory
- Tên HTTP terms: GET, POST, PUT, DELETE, Request, Response
- Tên authentication/security terms: Authentication, Authorization, CSRF, Encryption, Hashing
- Tên programming concepts: Array, String, Integer, Boolean, Object, Class, etc.
- Tên tools và services: GitHub, Git, Docker, Redis, MySQL, etc.
- Tên abbreviations: API, HTTP, HTTPS, JSON, XML, SQL, CLI, etc.
- **Các term kỹ thuật phổ biến:** Documentation, Configuration, Installation, Deployment, Database, Routing, Middleware, Controller, View, Model, Migration, Seed, Queue, Cache, Event, Listener, Provider, Service Provider

**LÝ DO GIỮ NGUYÊN:**
- Quen thuộc với developers Việt Nam
- Tự nhiên hơn khi đọc technical docs
- Dễ search khi làm việc với code
- Consistent với global Laravel community
- Tránh việc dịch không chuẩn gây hiểu nhầm

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

### Laravel Core Terms (LUÔN LUÔN GIỮ NGUYÊN):
| Term | Category |
|------|----------|
| Laravel | Framework |
| Artisan | CLI Tool |
| Eloquent | ORM |
| Blade | Template Engine |
| Facade | Design Pattern |
| Middleware | HTTP Middleware |
| Service Container | Dependency Injection |
| Service Provider | Bootstrap Component |
| Route | Routing |
| Controller | MVC Pattern |
| Model | MVC Pattern |
| View | MVC Pattern |
| Migration | Database |
| Seeder | Database |
| Factory | Testing/Database |
| Request | HTTP |
| Response | HTTP |

### General Technical Terms (LUÔN LUÔN GIỮ NGUYÊN):
| Term | Category |
|------|----------|
| Framework | Development |
| Package | Dependency |
| Library | Dependency |
| Dependency | Development |
| Configuration | Development |
| Environment | Development |
| Database | Data Storage |
| Authentication | Security |
| Authorization | Security |
| Session | State Management |
| Cookie | HTTP |
| Cache | Performance |
| Queue | Background Jobs |
| Event | Architecture |
| Listener | Event Handling |
| Broadcast | Real-time |
| API | Architecture |
| HTTP | Protocol |
| HTTPS | Protocol |
| JSON | Data Format |
| XML | Data Format |
| CLI | Interface |
| SSH | Protocol |

### Terms Có Thể Dịch (Context-dependent):
| Term | Tiếng Việt | Context |
|------|-----------|---------|
| Documentation | Tài liệu | Khi nói về quá trình tạo docs |
| Deployment | Triển khai | Khi nói về process |
| Installation | Cài đặt | Khi nói về process |
| Upgrade | Nâng cấp | Khi nói về process |

**NOTE:** Hầu hết các technical terms nên giữ nguyên tiếng Anh. Chỉ dịch khi nói về process/action, không phải về concept/kỹ thuật.

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

**Version:** 1.1
**Last Updated:** 2026-06-11
**Maintainer:** thienbd203
**Changes:** Updated to keep most technical terms in English for better readability and consistency with developer community