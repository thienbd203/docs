# Laravel Documentation - MkDocs Setup

Tài liệu Laravel được hiển thị sử dụng MkDocs với Material theme.

## 🚀 Local Development

### Cài đặt dependencies

```bash
pip install -r requirements.txt
```

### Chạy local server

```bash
mkdocs serve
```

Mở browser tại http://127.0.0.1:8000

### Build site

```bash
mkdocs build
```

Site sẽ được build vào thư mục `site/`

## 📦 CI/CD Setup

### GitHub Actions

Project đã được cấu hình với GitHub Actions để auto deploy lên GitHub Pages.

**Workflow sẽ chạy khi:**
- Push vào branch `13.x`, `main`, hoặc `master`
- Pull request vào các branch trên
- Trigger thủ công qua GitHub UI

### Cấu hình GitHub Pages

1. Vào repository Settings → Pages
2. Chọn Source là **GitHub Actions**
3. Workflow sẽ tự động deploy khi push code

**URL của site sẽ là:**
```
https://[username].github.io/docs/
```

Ví dụ: https://thienbd203.github.io/docs/

## 🎨 Tùy chỉnh Theme

Theme Material có thể tùy chỉnh trong file `mkdocs.yml`:

- Màu sắc (palette)
- Navigation features
- Search functionality
- Code highlighting
- v.v.

## 📝 Thêm/Sửa tài liệu

- Các file markdown nằm ở root directory
- Thêm file mới vào section nav trong `mkdocs.yml`
- Format filename: `ten-file.md`
- Sử dụng GitHub Flavored Markdown

## 🔍 Tính năng

- ✅ Dark/Light mode toggle
- ✅ Search functionality
- ✅ Responsive design
- ✅ Code syntax highlighting
- ✅ Navigation tabs & sections
- ✅ Auto deploy trên push
- ✅ Fast loading (static site)

## 📚 Tham khảo

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Material Theme Documentation](https://squidfunk.github.io/mkdocs-material/)
- [Laravel Official Docs](https://laravel.com/docs)