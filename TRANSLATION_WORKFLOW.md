# Workflow Dịch Thuật Laravel Documentation

## 🔄 Quy trình tổng quát

```
┌─────────────────┐
│  1. Select File │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  2. Create Issue │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  3. Assign Task  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  4. Translate    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  5. Create PR    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  6. Review       │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  7. Tech Review  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  8. Approve      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  9. Merge        │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ 10. Update Progress│
└─────────────────┘
```

## 📋 Chi tiết từng bước

### Bước 1: Chọn file để dịch

1. Xem `TRANSLATION_PLAN.md` để biết priority
2. Check `docs-vi/progress.md` để xem file nào chưa dịch
3. Chọn file theo priority và khả năng của bạn
4. Comment trong issue hoặc discussion để claim

### Bước 2: Tạo Issue

**Title format:**
```
[Translate] <file-name.md> - Priority <X>
```

**Template:**
```markdown
## File cần dịch
- **File:** `filename.md`
- **Priority:** 1/2/3/4/5/6/7/8
- **Ước tính độ khó:** Dễ/Trung bình/Khó

## Thông tin translator
- **Name:** @username
- **Thời gian dự kiến:** X ngày/tuần

## Notes
- Có term đặc biệt cần chú ý không?
- Có phần nào cần research thêm không?

## Checklist
- [ ] Đọc TRANSLATION_RULES.md
- [ ] Đọc file gốc để hiểu nội dung
- [ ] Xác định các term cần giữ nguyên
- [ ] Dịch nội dung
- [ ] Review format và structure
- [ ] Check links và references
- [ ] Spell check
- [ ] Update progress.md
```

### Bước 3: Assign task

- Maintainer sẽ assign issue cho translator
- Set due date dựa trên ước tính
- Label: `translation`, `priority-X`, `in-progress`

### Bước 4: Dịch thuật

1. **Chuẩn bị:**
   ```bash
   # Copy file từ docs-src sang docs-vi
   cp docs-src/filename.md docs-vi/filename.md
   ```

2. **Dịch:**
   - Mở file `docs-vi/filename.md`
   - Dịch theo quy tắc trong `TRANSLATION_RULES.md`
   - Giữ nguyên code blocks, links, format
   - Sử dụng checklist trong `TRANSLATION_RULES.md`

3. **Test local (nếu có thể):**
   ```bash
   # Test MkDocs build
   cp mkdocs.yml mkdocs-vi.yml
   # Edit mkdocs-vi.yml để trỏ docs_dir đến docs-vi
   mkdocs build -f mkdocs-vi.yml
   ```

### Bước 5: Tạo Pull Request

**Title format:**
```
[Translation] vi: <file-name.md>
```

**Template:**
```markdown
## Thông tin dịch thuật
- **File:** `filename.md`
- **Original:** `docs-src/filename.md`
- **Translation:** `docs-vi/filename.md`
- **Priority:** X
- **Translator:** @username

## Changes
- Đã dịch full file sang tiếng Việt
- Giữ nguyên tất cả code blocks
- Update links (nếu cần)
- Giữ nguyên format và structure

## Checklist
- [ ] Đã đọc và follow TRANSLATION_RULES.md
- [ ] Giữ nguyên tất cả code blocks
- [ ] Giữ nguyên links và references
- [ ] Term list đã được áp dụng đúng
- [ ] Spell check tiếng Việt
- [ ] Format và structure đúng
- [ ] Đã test build (nếu có thể)
- [ ] Đã update docs-vi/progress.md

## Reviewers
@reviewer1 @reviewer2

## Related Issue
Closes #X
```

### Bước 6: Review (Language Review)

**Reviewer responsibilities:**
- Check ngữ pháp và spelling tiếng Việt
- Đảm bảo câu văn tự nhiên
- Verify term consistency
- Check format và structure
- Test build (nếu có thể)

**Review checklist:**
- [ ] Ngữ pháp tiếng Việt đúng
- [ ] Câu văn tự nhiên, dễ hiểu
- [ ] Term được dùng consistent
- [ ] Không có lỗi spelling
- [ ] Format đúng với bản gốc
- [ ] Code blocks không bị thay đổi
- [ ] Links hoạt động tốt

**Feedback format:**
```markdown
## Review Comments

### Issues Found:
1. **Line X:** "Câu này hơi gượng" → Suggested: "..."
2. **Term:** "X" nên dịch là "Y"
3. **Link:** Link này đã die

### General Feedback:
- Tổng thể tốt, chỉ cần sửa vài chỗ
- Cần review lại section X
```

### Bước 7: Technical Review (Optional nhưng recommended)

**Technical reviewer responsibilities:**
- Verify technical accuracy
- Check code examples
- Verify Laravel terminology
- Cross-reference với official docs

**Review checklist:**
- [ ] Technical content chính xác
- [ ] Code examples đúng
- [ ] Laravel terms được dùng đúng
- [ ] Không có thông tin sai lệch

### Bước 8: Approve

- Sau khi review xong và các issue đã được fix
- Reviewer approve PR
- Label: `translation`, `approved`

### Bước 9: Merge

- Maintainer merge PR vào main branch
- Delete branch sau khi merge
- Auto deploy sẽ update site

### Bước 10: Update Progress

1. Update file `docs-vi/progress.md`:
   ```markdown
   | File | Trạng thái | Translator | Reviewer | Notes |
   |------|------------|------------|----------|-------|
   | filename.md | ✅ Hoàn thành | @translator | @reviewer | - |
   ```

2. Update tổng progress:
   - Tăng số files hoàn thành
   - Update percentage
   - Cập nhật thống kê theo tuần

3. Close issue liên quan

## 🏷️ Labels

Sử dụng các labels sau:

**Status:**
- `translation` - Dịch thuật
- `in-progress` - Đang thực hiện
- `review` - Đang review
- `approved` - Đã approved
- `merged` - Đã merge

**Priority:**
- `priority-1` - Cốt lõi
- `priority-2` - Database & ORM
- `priority-3` - Security
- `priority-4` - Testing
- `priority-5` - Advanced
- `priority-6` - Packages
- `priority-7` - AI
- `priority-8` - Tools
- `priority-9` - Dev Tools
- `priority-10` - Miscellaneous

**Difficulty:**
- `difficulty-easy` - Dễ
- `difficulty-medium` - Trung bình
- `difficulty-hard` - Khó

**Special:**
- `good-first-issue` - Dành cho người mới
- `help-wanted` - Cần trợ giúp
- `blocked` - Bị block

## 👥 Roles và Responsibilities

### Translator:
- Dịch file theo quy tắc
- Tạo PR với đầy đủ thông tin
- Fix các issue từ reviewer
- Update progress khi hoàn thành

### Language Reviewer:
- Review ngữ pháp và spelling
- Check term consistency
- Đảm bảo câu văn tự nhiên
- Provide feedback constructively

### Technical Reviewer:
- Verify technical accuracy
- Check code examples
- Verify Laravel terminology
- Cross-reference với official docs

### Maintainer:
- Assign tasks
- Approve và merge PRs
- Manage labels và milestones
- Update progress tracking
- Handle conflicts và issues

## 🔧 Tools và Automation

### GitHub Actions (Future):

Có thể tạo automated checks:
- Spell check tiếng Việt
- Term consistency check
- Format validation
- Link checker

### Scripts (Future):

```bash
# Script để tạo template file dịch
./scripts/create-translation.sh filename.md

# Script để validate translation
./scripts/validate-translation.sh docs-vi/filename.md

# Script để update progress
./scripts/update-progress.sh
```

## 📞 Communication

### Channels:
- **GitHub Issues** - Để claim và track tasks
- **GitHub PRs** - Để review và discuss
- **GitHub Discussions** - Để discuss general topics
- **Communication Channel** (Slack/Discord) - Real-time discussion

### Etiquette:
- Respectful và constructive feedback
- Clear và specific comments
- Responsive to reviews
- Ask for help khi needed
- Share knowledge và resources

---

**Version:** 1.0
**Last Updated:** 2026-06-11
**Maintainer:** thienbd203