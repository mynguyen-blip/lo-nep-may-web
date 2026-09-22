# Lưu đơn hàng vào Google Sheets

Mỗi khi khách hàng bấm **"Đã thanh toán, xác nhận đơn hàng"**, website sẽ gửi thông tin đơn hàng đến một Google Apps Script,
script này ghi thành 1 dòng mới vào Google Sheet của bạn. Bạn xem/quản lý đơn hàng trực tiếp trong Sheet như Excel bình thường.

## Bước 1 — Tạo Google Sheet

1. Vào [sheets.google.com](https://sheets.google.com) → tạo file mới, đặt tên ví dụ **"Lò Nếp Mây — Đơn hàng"**.
2. (Tùy chọn) Import sẵn tiêu đề cột: File > Import > Upload > chọn file `google-apps-script/orders-template.csv`
   trong gói code này → chọn "Insert new sheet". Sheet mới sẽ có tên "orders-template" — đổi tên sheet đó thành **`Orders`**
   (đúng chính tả, chữ hoa O). Nếu bỏ qua bước import này cũng không sao — script ở Bước 2 sẽ tự tạo sheet `Orders` với đúng
   tiêu đề cột trong lần ghi đầu tiên.

## Bước 2 — Cài Apps Script

1. Trong Google Sheet vừa tạo: **Tiện ích mở rộng (Extensions) → Apps Script**.
2. Xóa hết nội dung mặc định, dán toàn bộ nội dung file `google-apps-script/Code.gs` (trong gói code này) vào.
3. (Khuyến nghị) Chống spam: bấm biểu tượng bánh răng **Project Settings** → **Script Properties** → **Add script property**:
   - Property: `ORDER_WEBHOOK_TOKEN`
   - Value: một chuỗi bí mật bạn tự đặt, ví dụ `lnm-2026-bimat`
   - Việc này đảm bảo chỉ website của bạn (biết đúng mã) mới ghi được vào Sheet.
4. Bấm **Deploy → New deployment**:
   - Chọn loại: **Web app**
   - Execute as: **Me**
   - Who has access: **Anyone**
   - Bấm **Deploy**, cấp quyền truy cập khi Google hỏi (chọn tài khoản Google của bạn → Advanced → Go to (tên project) → Allow).
5. Copy URL dạng `https://script.google.com/macros/s/AKfycb.../exec` — đây là **VITE_ORDER_WEBHOOK_URL**.

> Lưu ý: mỗi lần bạn sửa code trong Apps Script, phải tạo **deployment mới** (Deploy → Manage deployments → Edit → New version)
> để thay đổi có hiệu lực trên URL đang dùng.

## Bước 3 — Cấu hình website

### Chạy local để test
Tạo file `web/.env` (copy từ `.env.example`):
```
VITE_ORDER_WEBHOOK_URL=https://script.google.com/macros/s/XXXXXXXX/exec
VITE_ORDER_WEBHOOK_TOKEN=lnm-2026-bimat
```
(Bỏ trống `VITE_ORDER_WEBHOOK_TOKEN` nếu bạn không đặt token ở Bước 2.3.)

### Deploy trên Vercel
Vào **Project Settings → Environment Variables**, thêm 2 biến trên với cùng giá trị, rồi **Redeploy**
(biến môi trường chỉ được đọc lúc build, đổi giá trị xong phải deploy lại mới có hiệu lực).

## Kiểm tra hoạt động

1. Mở URL Apps Script (`.../exec`) trực tiếp trên trình duyệt — phải thấy `{"ok":true,"message":"..."}`. Nếu lỗi, kiểm tra
   lại bước Deploy (Who has access phải là "Anyone").
2. Đặt thử 1 đơn trên website, đi hết luồng đến bước "Đã thanh toán, xác nhận đơn hàng".
3. Mở lại Google Sheet — sẽ thấy 1 dòng mới xuất hiện trong sheet `Orders`.

## Giới hạn cần biết

- Đây là giải pháp đơn giản, phù hợp quy mô 1 cửa hàng, lượng đơn không quá lớn (Google Sheets/Apps Script có giới hạn
  request/ngày, nhưng dư sức cho một tiệm bánh nhỏ).
- Ghi dữ liệu là **one-way** (website → Sheet), sửa trạng thái đơn hàng (đã giao, đã hủy...) làm trực tiếp trong Sheet,
  không đồng bộ ngược lại website.
- Nếu Google Sheets/Apps Script tạm thời lỗi, website vẫn cho khách đặt hàng bình thường — chỉ là đơn đó sẽ không tự ghi vào
  Sheet, và khách sẽ thấy thông báo nhắc liên hệ cửa hàng để đối chiếu.
