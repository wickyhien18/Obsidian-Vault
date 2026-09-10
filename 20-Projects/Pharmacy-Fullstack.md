---
Hub: "[[MOC - Web & Node]]"
created: 2026-09-07
status: active
repo: https://github.com/wickyhien18/FullStack_Pharmacy
---

# Ý tưởng dự án

Xây dựng nền tảng thương mại điện tử bán thuốc/dược phẩm (Pharmacy Wicky) — full-stack, deploy production thật, dùng làm portfolio chính khi xin việc Backend Engineer / hướng tới AI Engineer. Mục tiêu không chỉ chạy được mà phải xử lý đúng các vấn đề thực tế của một hệ thống production: schema chuẩn, race condition khi đặt hàng, auth an toàn đa thiết bị, cache, performance, và CI/CD (kể cả bài học đắt giá khi làm sai).

Live: [Pharmacy-Fullstack](https://full-stack-pharmacy.vercel.app/)

# Tech Stack

| Thành phần      | Công nghệ                       | Lý do chọn                                                                         |
| --------------- | ------------------------------- | ---------------------------------------------------------------------------------- |
| Backend runtime | Node.js + Express               | Layered architecture: routes → controllers → services → repositories               |
| ORM             | Prisma                          | Type-safe, kết hợp `select` để tối ưu payload, `$transaction` cho atomic operation |
| Database        | PostgreSQL (Supabase)           | Quan hệ rõ ràng, hỗ trợ `SELECT FOR UPDATE` cho race condition                     |
| Cache           | Redis                           | Cache GET endpoint ít đổi (products, categories), invalidate khi admin ghi         |
| Auth            | JWT (access + refresh)          | Refresh token rotation theo device, fingerprint qua `sec-ch-ua` headers            |
| Email           | Nodemailer + Gmail App Password | Không cần verify domain riêng như các dịch vụ email-as-a-service                   |
| Frontend        | React + Vite                    | Build nhanh, HMR tốt                                                               |
| Data fetching   | TanStack Query (React Query)    | `placeholderData` giữ data cũ khi refetch, tránh blank screen                      |
| Frontend deploy | Vercel                          | CDN global, `vercel.json` rewrite cho React Router                                 |
| Backend deploy  | Render (free tier)              | UptimeRobot + keep-alive ping bù nhược điểm cold start                             |
| CI/CD           | GitHub Actions                  | Test backend + build frontend tự động khi push `main`                              |
| Docs            | Swagger UI                      | Document API trực tiếp trong code                                                  |

# Nhật ký kỹ thuật

## Giai đoạn 1 — Thiết kế database schema & migration

**Mục tiêu giai đoạn:** Xây dựng schema chuẩn cho hệ thống pharmacy e-commerce, đảm bảo đủ các bảng cho toàn bộ nghiệp vụ đặt hàng, kho, thanh toán, giao hàng.

**Đã làm:**

- Thiết kế schema PostgreSQL gồm các bảng: `users`, `roles`, `refresh_tokens`, `products`, `product_images`, `categories`, `manufacturers`, `inventory`, `inventory_logs`, `carts`, `cart_items`, `orders`, `order_items`, `payments`, `shipments`, `otp_verifications`
- Định nghĩa enum types: `product_status`, `change_type_enum`, `order_status_enum`, `payment_status_enum`, `payment_method_enum`, `shipment_status_enum`
- Tạo index đầy đủ cho các cột hay dùng trong WHERE/ORDER BY
- Viết migration script SQL + Prisma schema song song
- Seed dữ liệu mẫu: 20+ sản phẩm thuốc, 3 danh mục, 4 nhà sản xuất
- Tạo bảng `product_images` tách ảnh ra khỏi bảng `products` (1 sản phẩm nhiều ảnh)

**Quyết định kỹ thuật:**

- **Vấn đề:** `order_status` và `payment_status` nên tách riêng hay gộp chung?
- **Đã chọn:** Tách riêng thành 2 field độc lập
- **Vì sao:** COD và VNPAY có vòng đời payment khác nhau hoàn toàn — gộp chung sẽ không thể model đúng. Shopify, Magento đều dùng cách tách này.
- **Vấn đề:** Soft delete hay hard delete cho products?
- **Đã chọn:** Soft delete với field `deleted_at`
- **Vì sao:** Cần giữ lịch sử order items tham chiếu đến product đã xoá, không bị FK violation.

**Bug gặp phải:**

- **Bug 1 — Rename `medicine` → `product` bị thiếu chỗ:** Khi đổi tên toàn bộ `medicine` → `product` trong DB, bỏ sót bảng `medicine_images` mới thêm → phải thêm migration script riêng cho bảng này. Fix bằng cách liệt kê đầy đủ: table, column, index, trigger, enum, FK constraint — rename không được làm theo cảm tính, phải có checklist.

**Việc còn tồn đọng từ giai đoạn này:**

- Chưa có seed script riêng biệt cho môi trường test — đây chính là lỗ hổng dẫn đến sự cố ở Giai đoạn 6 (CI dùng chung DB với production vì không có bộ seed/schema test độc lập để dựng nhanh).
- Chưa viết down-migration (rollback script) — nếu 1 migration lỗi giữa chừng, hiện tại không có cách revert sạch mà không thao tác tay.

---

## Giai đoạn 2 — Backend API (Node.js + Express + Prisma)

**Mục tiêu giai đoạn:** Xây dựng REST API đầy đủ cho toàn bộ nghiệp vụ, đảm bảo auth, phân quyền, validation, error handling chuẩn.

**Đã làm:**

- Cấu trúc project theo layered architecture: routes → controllers → services → repositories
- Auth: đăng ký, đăng nhập, refresh token, logout, profile
- Refresh token rotation theo device (mỗi browser/device có token riêng)
- Product CRUD + filter + phân trang + full-text search
- Category, Manufacturer CRUD
- Cart: thêm, xoá, cập nhật số lượng, clear cart
- Order: tạo đơn (atomic transaction + `SELECT FOR UPDATE`), xem lịch sử
- Admin: quản lý sản phẩm, đơn hàng, kho, xét duyệt huỷ/hoàn hàng
- Inventory log tự động khi stock thay đổi
- Upload ảnh lên Supabase Storage
- OTP email qua Nodemailer + Gmail SMTP
- Redis cache cho GET products và categories
- Swagger UI documentation
- GitHub Actions CI/CD

**Quyết định kỹ thuật:**

- **Vấn đề:** Dùng `omit` hay `select` trong Prisma để bỏ field thừa?
- **Đã chọn:** `select` thay vì `include` + `omit`
- **Vì sao:** Prisma không cho phép dùng `omit` và `include` cùng lúc — xung đột. `select` linh hoạt hơn, chỉ lấy đúng field cần dùng.
- **Vấn đề:** Xử lý race condition khi nhiều user đặt hàng cùng lúc?
- **Đã chọn:** `SELECT FOR UPDATE` trong `$transaction` với `isolationLevel: Serializable`
- **Vì sao:** Đảm bảo chỉ 1 transaction được lock và đọc inventory tại một thời điểm, transaction sau phải chờ và đọc lại giá trị mới — loại bỏ hoàn toàn oversell.
- **Vấn đề:** Device fingerprint để phân biệt browser khi login?
- **Đã chọn:** Hash SHA-256 từ `user-agent` + `sec-ch-ua` + `sec-ch-ua-mobile` + `sec-ch-ua-platform`
- **Vì sao:** `sec-ch-ua` phân biệt được Brave vs Chrome (cùng Chromium engine nhưng header khác nhau). Không dùng `origin` vì bị Vite proxy strip mất khi dev local.

**Bug gặp phải:**

- **Bug 2 — `categoryId` filter không hoạt động:** Query `/products?categoryId=5` không filter đúng → trong `buildWhere`, có `if (categoryId)` nhưng bên trong không thêm vào `where` object — chỉ check điều kiện rồi bỏ qua; `price` filter lại nằm lồng trong `if (categoryId)` nên cũng sai logic → tách `categoryId` và `price` thành 2 block độc lập, thêm `where.categoryId = Number(categoryId)`.
- **Bug 3 — Transaction timeout 5000ms:** Lỗi `Transaction already closed` khi tạo đơn hàng → Supabase free tier + Render free tier có độ trễ network cao, transaction mặc định 5s không đủ → tăng `timeout: 30000`, `maxWait: 10000` trong tất cả `$transaction` và thêm `transactionOptions` global trong `PrismaClient`.
- **Bug 4 — Refresh token bị ghi đè khi login nhiều browser:** Brave và Chrome trên Linux bị nhận là cùng 1 device vì `deviceInfo` chỉ lưu `"Chrome on Linux"` → thêm hash fingerprint từ `sec-ch-ua` headers để phân biệt (Brave có `"Brave";v="..."` trong `sec-ch-ua` nên hash ra khác Chrome).
- **Bug 5 — CORS block khi deploy:** Frontend Vercel (`vercel.app`) gọi Backend Render (`onrender.com`) bị CORS block → `sameSite: 'lax'` không cho phép cross-site cookie → đổi thành `sameSite: 'none'`, `secure: true` cho production, thêm danh sách allowed origins động thay vì 1 giá trị cố định.
- **Bug 6 — Gửi email thất bại với Resend:** Lỗi `gmail.com domain is not verified` — Resend không cho phép dùng `@gmail.com` làm sender vì không phải domain của mình → chuyển sang Nodemailer + Gmail App Password, không cần domain riêng.

**Việc còn tồn đọng từ giai đoạn này:**

- `DATABASE_URL` dùng trong GitHub Actions CI hiện là cùng 1 giá trị với production — chưa tách DB test riêng biệt (rủi ro trực tiếp dẫn đến sự cố ở Giai đoạn 6, cần xử lý trước khi bật lại CI).
- Nodemailer + Gmail SMTP cần theo dõi thêm độ ổn định trên Render — một số nhà cung cấp PaaS free tier chặn outbound port SMTP (587/465), cần có phương án dự phòng (email-as-a-service) nếu Gmail SMTP bị chặn trong tương lai.
- Google OAuth và VNPAY: code đã viết nhưng chưa test thật — thiếu Google Cloud Client ID/Secret và VNPAY sandbox TmnCode/HashSecret.
- Swagger UI mới cover một phần endpoint, cần rà soát bổ sung cho đủ toàn bộ API.

---

## Giai đoạn 3 — Frontend (React + Vite + TanStack Query)

**Mục tiêu giai đoạn:** Xây dựng UI người dùng và admin với đầy đủ tính năng, UX tốt.

**Đã làm:**

- Layout chung: Header (search, cart badge, auth), Footer
- `AuthInitializer`: khôi phục session từ cookie khi reload, prefetch categories + products
- Trang chủ: banner, danh mục icon grid, sản phẩm nổi bật
- ProductListPage: filter danh mục, search, sort, phân trang, `isFetching` overlay
- SearchPage: debounce 400ms, `isFetching` overlay
- ProductDetailPage: ảnh, thông tin, thêm vào giỏ
- CartPage: danh sách items, cập nhật số lượng, checkout
- AccountPage: thông tin cá nhân, lịch sử đơn hàng, đổi mật khẩu
- Admin: quản lý sản phẩm (CRUD + upload ảnh), đơn hàng, kho, yêu cầu huỷ hàng
- Skeleton loading: ProductCard, CategorySidebar, CartItem, ProductDetail, OrderCard, Banner
- Custom hooks: `useProducts`, `useProduct`, `useCategories`, `useCategoriesWithCount`
- Axios interceptor: auto refresh token khi 401, retry 2 lần khi 502/503
- `ErrorBoundary`: bắt crash toàn app, hiện thông báo thân thiện
- `placeholderData` trong React Query: giữ data cũ khi refetch (sort/filter không bị blank)

**Quyết định kỹ thuật:**

- **Vấn đề:** Auto refresh token không cần reload trang?
- **Đã chọn:** Axios response interceptor với queue pattern
- **Vì sao:** Khi accessToken hết hạn (15 phút), interceptor bắt 401, gọi refresh, retry request gốc. Nếu nhiều request cùng lúc bị 401, chỉ gọi refresh 1 lần rồi xử lý toàn bộ queue với token mới.
- **Vấn đề:** Sort/filter chậm gây UX xấu (reload cả trang)?
- **Đã chọn:** `placeholderData: (prev) => prev` + `isFetching` overlay
- **Vì sao:** Data cũ vẫn hiển thị trong khi fetch data mới, chỉ show overlay mờ nhỏ. User không thấy blank screen.
- **Vấn đề:** Refresh data khi chuyển trang?
- **Đã chọn:** `staleTime: 0` cho orders/cart, `staleTime: 5 phút` cho products/categories
- **Vì sao:** Orders/cart cần real-time, refetch mỗi khi component mount. Products/categories ít thay đổi, cache giảm số request.

**Bug gặp phải:**

- **Bug 7 — Filter category không hoạt động ở frontend:** Bấm vào danh mục không filter đúng → `categoriesArray` đã là mảng nhưng vẫn gọi `.items` lần nữa → `matchedCategory` luôn `undefined` → `categoryId` luôn rỗng → bỏ `.items` thừa: `categoriesArray.find(c => c.slug === categorySlug)`.
- **Bug 8 — User bị đăng xuất mỗi lần deploy:** Sau mỗi lần deploy Render, user phải đăng nhập lại → cookie `refreshToken` có `sameSite: 'lax'` → browser không gửi cookie cross-domain → server không nhận được token → 401 → logout → fix `sameSite: 'none'`, `secure: true` cho production.
- **Bug 9 — Request URL thiếu `/api`:** Frontend local gọi `render.com/categories` thay vì `render.com/api/categories` → `VITE_API_URL` được set là URL gốc nhưng axios `baseURL` không tự thêm `/api` → sửa `baseURL: ${VITE_API_URL}/api` cho production, `/api` cho local (qua Vite proxy).
- **Bug 10 — Vite proxy không forward origin header:** Device fingerprint trên local không có hash, trên Vercel có hash → Vite proxy strip origin header khi forward request → `origin` là `undefined` → hash ra chuỗi khác → không dùng `origin` trong fingerprint, chỉ dùng `sec-ch-ua` headers (không bị proxy ảnh hưởng).

**Việc còn tồn đọng từ giai đoạn này:**

- Chưa có TypeScript — toàn bộ frontend vẫn JavaScript thuần, cần migrate dần bắt đầu từ types cho API response.
- Chưa có test cho component React (Testing Library) — hiện chỉ có test backend.
- Chưa audit lại toàn bộ nơi dùng `staleTime: 0` xem có đang gây quá nhiều request thừa không, đặc biệt sau khi thêm skeleton + `isFetching` overlay.

---

## Giai đoạn 4 — Deploy & DevOps

**Mục tiêu giai đoạn:** Deploy toàn bộ hệ thống lên production, đảm bảo ổn định, không bị sleep, CI/CD tự động.

**Đã làm:**

- Deploy database lên Supabase (PostgreSQL managed)
- Deploy backend lên Render (free tier Node.js web service)
- Deploy frontend lên Vercel (free tier, CDN global)
- Cấu hình environment variables trên Render và Vercel
- Setup UptimeRobot ping `/health` mỗi 5 phút để tránh Render sleep
- Thêm `keepDbAlive` ping `SELECT 1` mỗi 4 phút để tránh Supabase drop idle connection
- `uncaughtException` + `unhandledRejection` handler để tránh server crash
- DB connection retry 3 lần khi khởi động
- Axios retry tự động 2 lần khi gặp 502/503
- GitHub Actions CI: tự động test backend + build frontend khi push lên `main`
- `vercel.json` rewrite rule để React Router hoạt động đúng khi F5

**Quyết định kỹ thuật:**

- **Vấn đề:** Monorepo (BackEnd + FrontEnd cùng repo) → deploy thế nào?
- **Đã chọn:** Render dùng Root Directory: `BackEnd`, Vercel dùng Root Directory: `FrontEnd`
- **Vì sao:** Cả 2 platform đều hỗ trợ chỉ định subfolder để build, không cần tách repo.
- **Vấn đề:** Render free tier sleep sau 15 phút → 502/503 khi wake up?
- **Đã chọn:** UptimeRobot ping 5 phút + Axios retry 2 lần khi 502/503
- **Vì sao:** UptimeRobot giữ server luôn awake (miễn phí). Axios retry làm fallback nếu vẫn gặp lỗi tạm thời.

**Bug gặp phải:**

- **Bug 11 — Build Render lâu (~3-4 phút):** `npm install` tải lại toàn bộ package mỗi lần deploy → chuyển `prisma generate` vào `postinstall` script trong `package.json`, đổi Build Command thành `npm ci` (nhanh hơn `npm install` vì không resolve dependencies, chỉ cài đúng theo `package-lock.json`).
- **Bug 12 — 502/503 liên tục sau khi fix cold start:** Server crash do unhandled promise rejection trong một số edge case → thêm `process.on('uncaughtException')` và `process.on('unhandledRejection')` log lỗi nhưng không exit process, kết hợp Axios retry 2 lần + delay tăng dần (1s, 2s).
- **Bug 13 — Supabase relation `schema_migrations` does not exist:** Log lỗi xuất hiện trong Supabase dashboard → đây là lỗi nội bộ của Supabase khi tự chạy maintenance task, không ảnh hưởng đến app → bỏ qua, không cần xử lý.

**Việc còn tồn đọng từ giai đoạn này:**

- GitHub Actions CI chạy thẳng vào `DATABASE_URL` production — đây chính là rủi ro dẫn tới sự cố nghiêm trọng ở Giai đoạn 6, phải tách DB test trước khi bật lại CI.
- Chưa cấu hình branch protection rule trên GitHub để bắt buộc pull request + review trước khi merge vào `main`, hiện tại push thẳng vẫn được.

---

## Giai đoạn 5 — Tối ưu hiệu suất

**Mục tiêu giai đoạn:** Cải thiện tốc độ đọc/ghi DB, giảm response time, cải thiện UX khi chờ data.

**Đã làm:**

- Thêm composite index: `(status, deleted_at)`, `(category_id, status)`, `(created_at DESC)`
- Thêm partial index chỉ index row active (`WHERE status = 'ACTIVE' AND deleted_at IS NULL`)
- Chuyển từ `include: true` (lấy hết field) sang `select` (chỉ lấy field cần dùng)
- Dùng `Promise.all` thay vì `await` tuần tự cho `findMany` + `count`
- Bật Redis cache cho 4 endpoint: `GET /products`, `GET /products/:slug`, `GET /categories`, `GET /categories/count`
- Invalidate cache tự động khi admin tạo/sửa/xoá sản phẩm
- Global slow query logger: tự động log query chậm hơn ngưỡng mà không cần thêm code thủ công
- Skeleton loading: ProductCard grid, CategorySidebar, CartItem, ProductDetail, OrderCard, Banner, CategoryIconGrid
- `placeholderData` + `isFetching` overlay cho sort/filter trên ProductListPage và SearchPage
- Optimistic update cho thêm vào giỏ hàng

**Quyết định kỹ thuật:**

- **Vấn đề:** Redis giúp được gì, không giúp được gì?
- **Đã chọn:** Chỉ cache GET endpoint ít thay đổi (products, categories), không cache write hay user-specific data
- **Vì sao:** Cache write không có tác dụng. Cart/orders là data cá nhân, cache sai user là bug nghiêm trọng. Products/categories đọc nhiều, ít thay đổi → cache hit rate cao → giảm 90%+ DB query cho các endpoint này.
- **Vấn đề:** Skeleton loading hay spinner toàn trang?
- **Đã chọn:** Skeleton cho lần đầu `isLoading`, overlay mờ cho `isFetching` (refetch)
- **Vì sao:** Skeleton giữ layout không bị "nhảy", user biết nội dung sắp hiện. Overlay mờ khi refetch giữ data cũ hiển thị thay vì blank screen.

**Việc còn tồn đọng từ giai đoạn này:**

- Chưa đo lại Lighthouse sau khi thêm composite index + partial index mới nhất để xác nhận cải thiện thực tế bằng số liệu.
- Global slow query logger hiện chỉ `console.log` — log biến mất khi server restart, chưa có nơi lưu trữ lâu dài (cần structured logging + file/service log riêng nếu muốn phân tích xu hướng).

### Nghiệp vụ quan trọng — Order Flow

**Order Status**

```
PENDING → CONFIRMED → SHIPPING → DELIVERED
    │          │           │
    ▼          ▼           ▼
CANCELLED  CANCEL_    RETURN_
           REQUESTED  REQUESTED
               │           │
               ▼           ▼
           CANCELLED   RETURNED
```

**Payment Status**

```
COD:   PENDING → PAID (khi DELIVERED) / FAILED / REFUNDED
VNPAY: PENDING → PAID (verify callback) / FAILED / REFUNDED
```

**Actors trong flow**

```
Customer   → Đặt hàng, huỷ PENDING, yêu cầu huỷ/hoàn
Pharmacist → Duyệt đơn (PENDING → CONFIRMED hoặc CANCELLED)
Admin      → Duyệt huỷ/hoàn, quản lý shipment
Shipper    → Cập nhật SHIPPING → DELIVERED / FAILED
```

**Race condition prevention**

sql

```sql
-- Trong $transaction với isolationLevel: Serializable
SELECT product_id, quantity
FROM inventory
WHERE product_id = ?
FOR UPDATE  -- lock row, transaction khác phải chờ
```

---

## Giai đoạn 6 — Sự cố mất dữ liệu Production (CI/CD)

**Mục tiêu giai đoạn:** Bổ sung GitHub Actions để tự động chạy test suite (Jest + Supertest, integration test thật vào DB) trước khi merge — nhưng đây là giai đoạn xảy ra sự cố nghiêm trọng nhất toàn dự án.

**Đã làm:**

- Viết bộ integration test đầy đủ: `auth.test.js`, `cart.test.js`, `category.test.js`, `product.test.js`, `notification.test.js`, `prisma.test.js`, `security.test.js`, `order.test.js`, `admin.test.js` — 55 test case
- Workflow GitHub Actions chạy `npm test` mỗi khi push lên `main`
- Set `DATABASE_URL` trong GitHub Secrets để CI kết nối DB khi test

**Quyết định kỹ thuật (sai lầm, ghi lại để nhớ):**

- **Vấn đề:** Dùng DB nào cho CI chạy test?
- **Đã chọn (sai):** Dùng luôn `DATABASE_URL` của production, không tạo project Supabase riêng cho môi trường test
- **Vì sao đây là sai lầm:** Không có ranh giới an toàn nào giữa môi trường test và production — bất kỳ lỗi nhỏ nào trong test cleanup (`afterAll`/`afterEach`) đều có khả năng phá huỷ dữ liệu thật, không có lớp phòng thủ nào chặn lại.

**Bug gặp phải:**

- **Bug 14 — [NGHIÊM TRỌNG] `DELETE FROM users WHERE 1=1` xoá sạch bảng `users` production:** Trong `order.test.js`, dòng cleanup viết `where: { email: orderUser.emai }` — lỗi gõ thiếu chữ `l` (`emai` thay vì `email`) → property `orderUser.emai` không tồn tại trên object → giá trị `undefined` → Prisma xử lý `where: { email: undefined }` bằng cách **bỏ qua hoàn toàn điều kiện đó** (khác với `null` — coi như không lọc theo field này) → `deleteMany({ where: {} })` → xoá toàn bộ bảng `users`. Vì `DATABASE_URL` trỏ vào production thật, mỗi lần CI chạy (mỗi lần push lên `main`) đều xoá sạch bảng này. Nhờ `orders`/`carts`/`refresh_tokens` dùng `ON DELETE SET NULL` (không phải CASCADE) nên các bảng này chỉ mất liên kết `user_id` (thành `NULL`) chứ không bị xoá theo.

**Cách sửa (nhiều lớp phòng thủ):**

1. Sửa lỗi gõ: `orderUser.emai` → `orderUser.email`
2. Soát toàn bộ 9 file test tìm lỗi tương tự — chỉ duy nhất 1 chỗ bị lỗi, các chỗ khác đều dùng property đúng hoặc hardcode string trực tiếp
3. Tạm vô hiệu hoá hoàn toàn workflow CI/CD (đổi tên file `.yml` → `.yml.disabled`)
4. Viết `jest.setup.js` làm lưới an toàn thứ 2: kiểm tra `DATABASE_URL` có chứa marker nhận diện DB test không, nếu không thì `process.exit(1)` ngay trước khi bất kỳ test nào chạy
5. Khôi phục: đăng ký lại tài khoản qua flow thật (để có password hash bcrypt đúng chuẩn), nâng quyền `ROLE_ADMIN` qua `UPDATE` SQL trực tiếp
6. Dọn dẹp order mồ côi: xoá `order_items`/`payments`/`shipments` có `order_id` thuộc các order có `user_id IS NULL`, rồi mới xoá các `orders` đó (đúng thứ tự để không vi phạm foreign key)

**Việc còn tồn đọng từ giai đoạn này:**

- **CI/CD tạm gác lại hoàn toàn** — quyết định có chủ đích, không bật lại cho đến khi tạo được Supabase project test riêng biệt và verify `jest.setup.js` hoạt động đúng.
- Supabase Free tier không có automated backup — rủi ro tương tự vẫn tồn tại cho bất kỳ thao tác DB thủ công nào khác (không chỉ riêng CI/CD). Cần xây thói quen `pg_dump` định kỳ thủ công dù không có CI.
- Chưa cấu hình branch protection + required review cho `main` (liên quan trực tiếp Bug 14 — nếu có review, có thể bắt được lỗi gõ trước khi merge).
- Chưa quyết định lại chiến lược test: tiếp tục dùng integration test thật (cần DB test riêng, tốn công setup) hay chuyển hẳn sang unit test mock repository cho phần nhạy cảm (nhanh, an toàn tuyệt đối, nhưng không bắt được lỗi tích hợp thật).
# Kiến thức đã áp dụng
- [[]]