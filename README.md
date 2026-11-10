# Nam Vương Mega Media — Web app bình chọn

Web app gồm giao diện người chơi (QR, không nhập tên) và trang Admin điều khiển 10 câu hỏi. Dữ liệu được đồng bộ nhiều thiết bị qua Supabase Realtime.

## Có trong bản này
- 13 ứng viên giả định và 10 câu hỏi mẫu.
- Admin mở/khóa từng câu, NEXT theo thứ tự, kết thúc để công bố bảng xếp hạng.
- Người chơi dùng anonymous sign-in ẩn danh của Supabase; không nhập tên/email.
- Mỗi tài khoản ẩn danh chỉ gửi được 1 phiếu cho mỗi câu hỏi (ràng buộc unique ở database).
- QR tham gia tự sinh trong trang Admin.
- Giao diện responsive, phong cách hiện đại tối giản.

## Cần thiết lập một lần để chạy thật

### 1. Tạo Supabase project
1. Vào https://supabase.com và tạo project.
2. Mở **SQL Editor** → New query.
3. Dán toàn bộ nội dung `supabase/schema.sql` → Run.
4. Vào **Authentication → Providers** và bật **Anonymous Sign-Ins**.
5. Trong **Project Settings → API**, sao chép Project URL và anon/public key.

### 2. Tạo tài khoản Admin
1. Trong Supabase, vào **Authentication → Users** và tạo user bằng email + mật khẩu (hoặc đăng ký bằng trang auth của Supabase).
2. Sao chép UUID của user vừa tạo.
3. Chạy câu SQL sau trong SQL Editor, thay UUID:
   ```sql
   insert into public.admin_users(user_id)
   values ('UUID-TAI-KHOAN-ADMIN');
   ```
4. Đăng nhập trang Admin bằng email/mật khẩu đó.

### 3. Chạy ứng dụng
Cài Node.js phiên bản LTS, sau đó trong thư mục dự án:
```bash
npm install
```
Tạo file `.env` (copy từ `.env.example`) và điền:
```env
VITE_SUPABASE_URL=https://YOUR_PROJECT.supabase.co
VITE_SUPABASE_ANON_KEY=YOUR_SUPABASE_ANON_KEY
```
Chạy thử:
```bash
npm run dev
```
Mở URL Vite hiển thị. Trang người chơi là `/`; trang Admin là `/?admin=1`.

### 4. Đưa lên Internet để quét QR tại sự kiện
Có thể deploy thư mục này lên Vercel hoặc Netlify:
- Build command: `npm run build`
- Output directory: `dist`
- Thêm 2 biến môi trường `VITE_SUPABASE_URL` và `VITE_SUPABASE_ANON_KEY`.
- Sau khi deploy, mở `https://ten-mien-cua-ban/?admin=1`. QR sẽ trỏ về trang người chơi cùng domain.

## Tùy chỉnh tên ứng viên và câu hỏi
Sửa trực tiếp trong bảng `candidates` và `questions` trên Supabase. Giữ `position` từ 1–13 cho ứng viên và 1–10 cho câu hỏi. Nên hoàn tất trước sự kiện.

## Lưu ý quan trọng về giới hạn “mỗi người một phiếu”
Không đăng nhập/không nhập tên đồng nghĩa không thể xác minh danh tính người thật. Bản này cấp một phiên ẩn danh cho trình duyệt và khóa một phiếu mỗi câu theo phiên đó. Người dùng có thể tạo phiên mới nếu xóa dữ liệu trình duyệt hoặc đổi thiết bị. Nếu cần chống gian lận mạnh hơn, cần thêm mã vé/mã nhân viên dùng một lần hoặc xác thực qua danh sách nội bộ.

## Khởi động lại minigame
Để chạy một sự kiện mới, admin cần reset dữ liệu phiếu và trạng thái. Thực hiện trong SQL Editor sau khi xác nhận muốn xóa kết quả cũ:
```sql
delete from public.votes;
update public.game_state set status='waiting', current_question_id=null, updated_at=now() where id=1;
```
