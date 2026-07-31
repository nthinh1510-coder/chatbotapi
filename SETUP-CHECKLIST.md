# Checklist triển khai nhanh

## Bắt buộc

- [ ] Cài Node.js 20.9+
- [ ] Chạy `npm install`
- [ ] Sao chép `.env.example` thành `.env.local`
- [ ] Tạo Supabase project
- [ ] Chạy `supabase/schema.sql`
- [ ] Điền khóa Supabase
- [ ] Điền OpenAI API key và system prompt
- [ ] Chọn `PAYMENT_PROVIDER=payos` hoặc `stripe`
- [ ] Cấu hình khóa và webhook của cổng thanh toán
- [ ] Chạy `npm run dev` và thử đăng ký
- [ ] Thử thanh toán, kiểm tra gói được kích hoạt
- [ ] Thử gửi chat, kiểm tra lịch sử và hạn mức

## Trước production

- [ ] Đổi tên/logo/nội dung thương hiệu
- [ ] Thay domain production trong biến môi trường
- [ ] Thêm callback production vào Supabase
- [ ] Đổi webhook sang domain production
- [ ] Bật cảnh báo chi phí OpenAI
- [ ] Rà soát pháp lý và chính sách dữ liệu
- [ ] Kiểm thử trên điện thoại
