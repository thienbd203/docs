# Template Dịch Thuật - Laravel Documentation

Sử dụng template này khi dịch file mới. Copy và điền thông tin của bạn.

## 📋 Thông tin file dịch

- **Tên file gốc:** `docs-src/filename.md`
- **Tên file dịch:** `docs-vi/filename.md`
- **Priority:** 1/2/3/4/5/6/7/8/9/10
- **Ước tính độ khó:** Dễ/Trung bình/Khó
- **Translator:** @username
- **Ngày bắt đầu:** DD/MM/YYYY
- **Ngày hoàn thành dự kiến:** DD/MM/YYYY

## 🔤 Term list đặc biệt (nếu có)

Liệt kê các term đặc biệt trong file này cần lưu ý:

| Term gốc | Bản dịch | Ghi chú |
|----------|----------|---------|
| example | ví dụ | - |
| specific term | term cụ thể | Chỉ dùng trong context này |

## 📝 Notes và Questions

Ghi chú bất kỳ vấn đề hoặc câu hỏi trong quá trình dịch:

- Câu hỏi 1: ...
- Câu hỏi 2: ...
- Issue cần clarify: ...

## ✅ Checklist dịch thuật

### Chuẩn bị:
- [ ] Đọc file gốc hoàn toàn
- [ ] Hiểu nội dung và context
- [ ] Xác định các term cần giữ nguyên tiếng Anh
- [ ] Xác định các term cần dịch sang tiếng Việt
- [ ] Chuẩn bị resources (dictionary, glossary)

### Dịch thuật:
- [ ] Dịch title và headings
- [ ] Dịch paragraphs chính
- [ ] Dịch lists và bullet points
- [ ] Dịch tables (nếu có)
- [ ] Dịch notes và warnings
- [ ] GIỮ NGUYÊN tất cả code blocks
- [ ] GIỮ NGUYÊN tất cả inline code
- [ ] GIỮ NGUYÊN tất cả command examples
- [ ] GIỮ NGUYÊN tất cả file paths
- [ ] GIỮ NGUYÊN tất cả URLs

### Review:
- [ ] Check ngữ pháp tiếng Việt
- [ ] Check spelling tiếng Việt
- [ ] Đảm bảo câu văn tự nhiên
- [ ] Check term consistency
- [ ] Verify format với bản gốc
- [ ] Check headings structure
- [ ] Check links và references
- [ ] Check tables format (nếu có)
- [ ] Check images và alt text (nếu có)

### Technical:
- [ ] Verify technical accuracy
- [ ] Check code examples (không thay đổi)
- [ ] Verify Laravel terminology
- [ ] Cross-reference với official docs
- [ ] Check version-specific notes

### Final:
- [ ] Đọc lại toàn bộ bản dịch
- [ ] So sánh với bản gốc
- [ ] Đảm bảo không bỏ sót nội dung
- [ ] Update file `docs-vi/progress.md`
- [ ] Chuẩn bị PR description

## 🚀 Quick Reference

### Các term LUÔN LUÔN giữ nguyên tiếng Anh:
**Laravel Core:**
- Laravel, PHP, Composer, npm, yarn
- Artisan, Eloquent, Blade, Facade
- Route, Controller, Model, View
- Migration, Seeder, Factory
- Request, Response, Middleware
- Service Container, Service Provider

**Technical Terms:**
- HTTP, HTTPS, API, JSON, XML, SQL, CLI, SSH
- Database, Authentication, Authorization, Cache, Queue
- Event, Listener, Broadcast, Session, Cookie
- Git, GitHub, Docker, Redis, MySQL
- Configuration, Installation, Deployment, Documentation

**Programming Concepts:**
- Framework, Package, Library, Dependency
- Class, Object, Array, String, Integer, Boolean
- Function, Method, Property, Variable
- Interface, Abstract, Namespace

### Terms có thể dịch (chỉ khi nói về process):
- Documentation → Tài liệu (chỉ khi nói về việc tạo docs)
- Installation → Cài đặt (chỉ khi nói về process)
- Deployment → Triển khai (chỉ khi nói về process)
- Upgrade → Nâng cấp (chỉ khi nói về process)

**QUAN TRỌNG:** Hầu hết technical terms nên GIỮ NGUYÊN tiếng Anh!

### Format rules:
- GIỮ NGUYÊN tất cả code blocks (```php, ```bash)
- GIỮ NGUYÊN tất cả inline code (`code`)
- GIỮ NGUYÊN tất cả links
- GIỮ NGUYÊN cấu trúc headings (#, ##, ###)
- GIỮ NGUYÊN lists (-, 1., *)

### Common mistakes để tránh:
- ❌ Dịch từng từ (word-for-word)
- ❌ Thay đổi code examples
- ❌ Bỏ qua warnings và notes
- ❌ Thay đổi cấu trúc document
- ❌ Dịch tên functions/classes
- ❌ Thay đổi file paths
- ❌ Dịch machine translation trực tiếp

## 📞 Khi cần help

Nếu gặp vấn đề:
1. Check `TRANSLATION_RULES.md`
2. Check `TRANSLATION_PLAN.md`
3. Search trong GitHub issues
4. Hỏi trong GitHub discussions
5. Tag maintainer hoặc reviewer

## 🎯 Success criteria

Bản dịch được coi là hoàn thành khi:
- ✅ Tất cả nội dung đã dịch
- ✅ Format đúng với bản gốc
- ✅ Code blocks không bị thay đổi
- ✅ Term consistency được đảm bảo
- ✅ Ngữ pháp và spelling đúng
- ✅ Câu văn tự nhiên, dễ hiểu
- ✅ Đã qua review và được approve

---

**Template Version:** 1.1
**Last Updated:** 2026-06-11
**Changes:** Updated term list to keep most technical terms in English