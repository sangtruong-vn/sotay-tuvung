# Ngữ cảnh dự án — Sổ tay Từ vựng Đa ngôn ngữ

Ghi chú dành cho AI đọc trước khi sửa. Cập nhật file này khi thay đổi kiến trúc.

## Bản chất dự án

Trang web học từ vựng 4 thứ tiếng (Việt – Anh – Nhật – Trung) cho một người học cá nhân.
**Toàn bộ dự án là MỘT file `index.html` tĩnh.** Không build step, không package manager,
không backend. Deploy bằng GitHub Pages từ nhánh `main` — push xong Pages tự build ~1–2 phút.

Phụ thuộc ngoài duy nhất: 2 thẻ CDN có sẵn từ đầu (Tailwind `cdn.tailwindcss.com`,
Font Awesome 6.0.0). **Không thêm dependency mới.** Mọi tính năng phải chạy được khi
mở thẳng file bằng trình duyệt.

## Ràng buộc bắt buộc giữ

1. **Giữ tính static.** Không thêm bundler, framework, file JS/CSS rời, hay bất cứ thứ gì
   cần cài đặt. Người dùng đã nói rõ: chạy GitHub Pages, chỉ HTML + JS.
2. **Không dùng API Node.** `node --check` chỉ là công cụ soi cú pháp trên máy dev,
   không phải runtime. Trong `index.html` không được có `require`/`import`/`process`.
3. **Ngôn ngữ giao tiếp: tiếng Việt** (có dấu đầy đủ). Comment trong code cũng tiếng Việt.
4. Commit message: tiếng Anh, ngắn gọn, theo phong cách lịch sử repo.

## Bản đồ code

`index.html` ~2200 dòng. Định vị bằng `grep -n` theo các mốc dưới đây thay vì số dòng
(số dòng sẽ trôi khi sửa):

| Mốc grep | Nội dung |
|---|---|
| `const database = [` | 18 chủ đề gốc: `{id, icon, title, vocab[], sentences[]}` |
| `const extraContent = {` | Mẫu câu bổ sung + `dialogues[]`, khóa theo `id` chủ đề |
| `// Gộp nội dung mở rộng vào database` | Vòng merge 2 nguồn trên |
| `// ===== HIỂN THỊ NGÔN NGỮ =====` | Ẩn/hiện EN/JA/ZH |
| `// ===== PHÁT ÂM (Web Speech API) =====` | TTS, chấm điểm giọng, panel chọn giọng |
| `// ===== TIẾN ĐỘ "ĐÃ THUỘC" =====` | Đánh dấu thuộc từ |
| `function startFlashcard` / `function renderFlashcard` | Flashcard theo bộ |
| `// ===== TÌM KIẾM =====` | `vocabIndex` / `lineIndex`, khớp bỏ dấu |
| `// ===== KIỂM TRA (QUIZ) =====` | Quiz trắc nghiệm 4 đáp án |
| `function vocabCells` / `function langLines` | Render dùng chung |

## Quy tắc khi sửa

**Thêm nội dung học (từ, câu, hội thoại):** sửa `extraContent`, KHÔNG đụng `database` gốc.
Thêm chủ đề mới thì phải thêm vào cả hai.

**Mọi markup đa ngôn ngữ phải đi qua `vocabCells()` hoặc `langLines()`.** Hai hàm này gắn
class `lang-en` / `lang-ja` / `lang-zh`; CSS `body.hide-*` dựa vào đó để ẩn. Viết markup
4 thứ tiếng thẳng tay ở chỗ khác sẽ làm tính năng ẩn/hiện ngôn ngữ hỏng âm thầm.

## localStorage — cẩn thận khi đổi

Ba khóa độc lập, đều bọc `try/catch` (phải giữ, vì chế độ ẩn danh có thể ném lỗi):

- `sotay-tuvung:learned` — mảng chuỗi `"vi|en"`, là tiến độ học thật của người dùng.
  **Đổi format khóa này = xóa sạch tiến độ.** Nếu buộc phải đổi, viết migrate.
- `sotay-tuvung:voice` — giọng đọc đã chọn + tốc độ.
- `sotay-tuvung:langs` — ngôn ngữ đang hiện.

## Bẫy đã gặp, đừng lặp lại

1. **Listener nút loa để ở pha capture** (`addEventListener(..., true)`) kèm
   `stopPropagation()`. Cố ý: để bấm nghe trên flashcard mà thẻ không bị lật.
   Đổi sang bubbling là hỏng.
2. **Class của quiz option gán tường minh, không dùng `className.replace()`.**
   Thứ tự CSS Tailwind không theo thứ tự class trong attribute, nên `bg-emerald-50`
   thêm sau vẫn có thể bị `bg-white` đè.
3. **Chọn giọng TTS phải chấm điểm, không lấy giọng đầu tiên khớp ngôn ngữ.**
   macOS mặc định trả về giọng *compact*, nghe rè như nghẹt mũi — người dùng đã phàn nàn.
   `voiceScore()` trừ 60 điểm cho compact/eloquence/espeak, cộng 50 cho giọng mạng.
4. Câu tiếng Nhật/Trung trong data có kèm phiên âm trong ngoặc. `stripPron()` cắt bỏ
   trước khi đọc, nếu không máy sẽ đọc cả romaji/pinyin.
5. Từ nhiều biến thể dạng `爷爷/外公` — dùng `firstAlt()` lấy phương án đầu khi đọc.

## Cách kiểm tra (không có test suite)

1. Trích phần trong `<script>` ra file tạm rồi `node --check` để soi cú pháp.
2. Đối chiếu mọi `getElementById('x')` trong JS với các `id="x"` trong HTML.
   Hiện có 87 ID, khớp hết. Đây là cách bắt lỗi rẻ và hiệu quả nhất cho file này.
3. Mở trình duyệt click thật: nút loa, flashcard, quiz, tìm kiếm.
   TTS chỉ chuẩn trên HTTPS/Pages, `file://` có thể không phát.

## Việc còn mở

- **Đồng bộ đa thiết bị.** `localStorage` tách theo trình duyệt nên máy tính và điện thoại
  có tiến độ riêng. Đã thống nhất với chủ dự án: khi cần sẽ làm nút xuất/nhập JSON,
  không dựng backend.
- Chưa có: quiz chiều ngược (nhìn ngoại ngữ chọn tiếng Việt), spaced repetition,
  chủ đề mới.

## Lịch sử quyết định

- Commit thẳng lên `main`, không tách nhánh — vì Pages serve từ `main`, tách nhánh thì
  trang không cập nhật. Chủ dự án đã đồng ý cách này.
- Chủ dự án muốn được xem và test trước khi commit; đừng tự commit khi chưa được bảo.
