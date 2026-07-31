# NovaChat AI SaaS

Bộ mã nguồn website SaaS bán quyền truy cập chatbot AI. Người dùng có thể đăng ký, mua hoặc gia hạn gói, sau đó truy cập khu vực chat theo quyền và hạn mức đã cấu hình.

## Chức năng có sẵn

- Landing page hiện đại, responsive cho máy tính và điện thoại.
- Đăng ký/đăng nhập bằng email và Google OAuth qua Supabase.
- Thanh toán bằng **payOS/VietQR** hoặc **Stripe Subscription**.
- Webhook kích hoạt gói tự động, chống kích hoạt trùng đơn hàng.
- Chặn truy cập chatbot khi chưa có gói hoặc gói đã hết hạn.
- Hạn mức tin nhắn theo tháng, thao tác trừ lượt nguyên tử trong PostgreSQL.
- Chatbot dùng OpenAI Responses API, tiếp tục ngữ cảnh theo hội thoại.
- Tùy chọn File Search bằng OpenAI Vector Store để đưa tài liệu riêng vào chatbot.
- Lưu lịch sử chat theo từng tài khoản.
- Trang quản trị theo dõi người dùng, thuê bao và lượng tin nhắn.
- Trang điều khoản, chính sách bảo mật và trạng thái thanh toán.

## Công nghệ

- Next.js 16 App Router + TypeScript
- Supabase Auth + PostgreSQL + Row Level Security
- OpenAI Responses API
- payOS Node SDK hoặc Stripe Checkout/Billing Portal/Webhook
- CSS thuần, không phụ thuộc UI framework

## 1. Chạy trên máy tính

Yêu cầu Node.js `20.9` trở lên.

```bash
npm install
cp .env.example .env.local
npm run dev
```

Mở `http://localhost:3000`.

## 2. Tạo Supabase

1. Tạo một project tại Supabase.
2. Mở **SQL Editor** và chạy toàn bộ file `supabase/schema.sql`.
3. Trong **Authentication > Providers**, bật Email/Password; bật Google nếu cần.
4. Trong cấu hình URL của Auth, thêm:
   - `http://localhost:3000/auth/callback`
   - `https://TEN-MIEN-CUA-BAN/auth/callback`
5. Điền URL, anon key và service role key vào `.env.local`.
6. Không đưa `SUPABASE_SERVICE_ROLE_KEY` vào mã phía client hoặc GitHub công khai.

Tạo tài khoản quản trị bằng SQL:

```sql
update public.profiles
set role = 'admin'
where id = 'USER_UUID';
```

Sau đó truy cập `/admin`.

## 3. Chọn cổng thanh toán

Trong `.env.local`:

```env
PAYMENT_PROVIDER=payos
```

Hoặc:

```env
PAYMENT_PROVIDER=stripe
```

### Phương án A — payOS/VietQR

Đây là chế độ mặc định của bộ mã. Mỗi lần thanh toán sẽ cộng thêm số ngày của gói vào thời hạn đang còn hiệu lực.

1. Tạo kênh thanh toán trong payOS.
2. Điền `PAYOS_CLIENT_ID`, `PAYOS_API_KEY`, `PAYOS_CHECKSUM_KEY`.
3. Cấu hình webhook production:

```text
https://TEN-MIEN-CUA-BAN/api/payos/webhook
```

4. Giá tiền và số ngày của gói nằm trong `lib/plans.ts`.
5. Dùng giao dịch thử nhỏ trước khi mở bán thật.

### Phương án B — Stripe Subscription

1. Tạo 3 Product và Recurring Price cho Starter, Pro, Business.
2. Điền các Price ID vào `STRIPE_PRICE_STARTER`, `STRIPE_PRICE_PRO`, `STRIPE_PRICE_BUSINESS`.
3. Tạo webhook:

```text
https://TEN-MIEN-CUA-BAN/api/stripe/webhook
```

4. Chọn các sự kiện:
   - `checkout.session.completed`
   - `customer.subscription.created`
   - `customer.subscription.updated`
   - `customer.subscription.deleted`
5. Điền signing secret vào `STRIPE_WEBHOOK_SECRET`.
6. Bật Stripe Customer Portal trong Dashboard.

Test local với Stripe CLI:

```bash
stripe listen --forward-to localhost:3000/api/stripe/webhook
```

## 4. Cấu hình chatbot OpenAI

```env
OPENAI_API_KEY=sk-...
OPENAI_MODEL=gpt-5.6-terra
CHATBOT_SYSTEM_PROMPT="Hướng dẫn đầy đủ của chatbot của bạn"
```

Custom GPT tạo trong giao diện ChatGPT không thể được nhúng trực tiếp vào website. Hãy chuyển:

- Instructions → `CHATBOT_SYSTEM_PROMPT`.
- Knowledge files → OpenAI Vector Store, sau đó điền `OPENAI_VECTOR_STORE_ID`.
- Actions hoặc quy trình riêng → bổ sung thành API/tool server-side theo nhu cầu.

Khóa OpenAI chỉ được đọc trong Route Handler phía máy chủ.

## 5. Chỉnh gói, giá và hạn mức

Sửa `lib/plans.ts`:

```ts
{
  id: "starter",
  name: "Starter",
  priceLabel: "199.000đ/30 ngày",
  amountVnd: 199000,
  durationDays: 30,
  messageLimit: 500,
}
```

- Với payOS, `amountVnd` là số tiền thực thu.
- Với Stripe, giá thực thu nằm trong Stripe Price ID.
- `messageLimit: null` nghĩa là không giới hạn theo gói.

## 6. Đổi thương hiệu

- Tên ứng dụng: `NEXT_PUBLIC_APP_NAME`.
- Domain: `NEXT_PUBLIC_APP_URL`.
- Màu sắc và giao diện: `app/globals.css`.
- Nội dung landing: `app/page.tsx`.
- Điều khoản và bảo mật: `app/terms/page.tsx`, `app/privacy/page.tsx`.

## 7. Triển khai lên Vercel

1. Đẩy thư mục lên GitHub.
2. Import repository vào Vercel.
3. Thêm toàn bộ biến môi trường.
4. Đổi `NEXT_PUBLIC_APP_URL` thành domain thật, không có dấu `/` cuối.
5. Cập nhật callback URL trong Supabase.
6. Cấu hình webhook payOS hoặc Stripe với domain production.
7. Chạy một giao dịch thật giá trị nhỏ và kiểm tra:
   - Bản ghi `payment_orders` hoặc `subscriptions`.
   - Trang `/chat` được mở khóa.
   - Tin nhắn được lưu vào `chat_messages`.

## Cấu trúc chính

```text
app/
  api/chat                 API chatbot
  api/checkout             Tạo phiên/link thanh toán
  api/payos/webhook        Kích hoạt gói payOS
  api/stripe/webhook       Đồng bộ thuê bao Stripe
  admin                    Trang quản trị
  chat                     Khu vực người dùng trả phí
components/                Thành phần giao diện
lib/                       OpenAI, thanh toán, Supabase, gói
supabase/schema.sql        Database, RLS và RPC
proxy.ts                   Làm mới phiên Supabase Auth
```

## Checklist trước khi mở bán

- Thay tên thương hiệu, logo và email hỗ trợ.
- Nhờ người có chuyên môn rà soát điều khoản, chính sách bảo mật và quy trình hoàn tiền.
- Bật giới hạn chi phí và cảnh báo ngân sách OpenAI.
- Kiểm thử webhook test/live và xử lý trường hợp thanh toán đến chậm.
- Bổ sung rate limiting theo IP, moderation và hệ thống log nếu lượng truy cập lớn.
- Thiết lập email giao dịch, hỗ trợ khách hàng, sao lưu và giám sát lỗi.
- Không commit `.env.local` hoặc bất kỳ secret key nào.
