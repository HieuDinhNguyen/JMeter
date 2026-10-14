# Báo Cáo Kiểm Thử Hiệu Năng Hệ Thống (Performance Testing Report)

> **Mục tiêu kiểm thử:** Đánh giá hiệu năng và độ ổn định của endpoint trang chủ dịch vụ [httpbin.org](https://httpbin.org) dưới tải mô phỏng bằng công cụ Apache JMeter.

---

## 1. Thông Tin Chung & Môi Trường Kiểm Thử

| Mục | Chi tiết |
| :--- | :--- |
| **Công cụ kiểm thử** | Apache JMeter (v5.6.3) |
| **Mục tiêu kiểm thử** | `https://httpbin.org/` |
| **Kịch bản (Test Plan)** | `Local Website Performance Test` |
| **Giao thức (Protocol)** | HTTPS |
| **Phương thức (Method)** | GET |
| **Thời gian thực hiện** | 14/10/2026 (ICT) |
| **Số lượng người dùng mô phỏng (Threads/Users)** | 20 Virtual Users (VU) |

---

## 2. Kịch Bản & Cấu Hình Kiểm Thử (Test Plan Setup)

Kịch bản được thiết lập trong Apache JMeter bao gồm các thành phần:
- **Thread Group:** Thiết lập nhóm luồng với 20 người dùng mô phỏng.
- **HTTP Request (`GET Home Page`):**
  - **Protocol:** `https`
  - **Server Name or IP:** `httpbin.org`
  - **Method:** `GET`
  - **Path:** `/`
  - **Tùy chọn:** Bật `Follow Redirects` và `Use KeepAlive`.
- **HTTP Header Manager:** Quản lý và cấu hình các HTTP header cho request.
- **Listeners:**
  - `View Results Tree`: Theo dõi chi tiết từng request gửi đi và phản hồi trả về từ máy chủ.
  - `Summary Report`: Thống kê tổng hợp các chỉ số hiệu năng (KPI).

### Hình ảnh cấu hình HTTP Request:
![Cấu hình HTTP Request - GET Home Page](images/01_http_request_config.png)

---

## 3. Kết Quả Kiểm Thử Tổng Hợp (Summary Report)

Sau khi hoàn tất quá trình chạy tải với tổng số **180 samples**, dữ liệu thu nhận được tại **Summary Report** như sau:

![Summary Report](images/02_summary_report.png)

### Bảng dữ liệu thống kê chi tiết:

| Chỉ số (Metric) | Giá trị ghi nhận | Đơn vị / Ý nghĩa |
| :--- | :---: | :--- |
| **Tổng số mẫu (# Samples)** | **180** | Tổng số request đã gửi |
| **Thời gian phản hồi TB (Average)** | **2,841 ms** | ~ 2.84 giây |
| **Thời gian phản hồi nhỏ nhất (Min)** | **175 ms** | Thời gian phản hồi nhanh nhất |
| **Thời gian phản hồi lớn nhất (Max)** | **21,072 ms** | ~ 21.07 giây (độ trễ đỉnh điểm) |
| **Độ lệch chuẩn (Std. Dev.)** | **6,450.02 ms** | Phản ánh độ dao động phản hồi cực lớn |
| **Tỷ lệ lỗi (Error %)** | **55.56%** | Có tới 100 / 180 request bị thất bại |
| **Thông lượng (Throughput)** | **9.7 / min** | ~ 0.16 requests/giây |
| **Băng thông nhận (Received)** | **4.76 KB/sec** | Tốc độ truyền tải dữ liệu nhận về |
| **Băng thông gửi (Sent)** | **0.03 KB/sec** | Tốc độ truyền tải dữ liệu gửi đi |
| **Kích thước trung bình (Avg. Bytes)** | **30,134.5 Bytes** | Dung lượng phản hồi trung bình (~29.4 KB) |

---

## 4. Chi Tiết Thực Thi Request (View Results Tree)

Kiểm tra một request thành công điển hình (`Thread Group 1-5`) ghi nhận các thông số kỹ thuật chi tiết:

![View Results Tree](images/03_view_results_tree.png)

### Thông số mẫu thành công (Sample Result Detail):
* **Thread Name:** `Thread Group 1-5`
* **Sample Start:** `2026-10-14 11:15:54 ICT`
* **Response Code:** `200`
* **Response Message:** `OK`
* **Load Time (Thời gian tải):** `902 ms`
* **Connect Time (Thời gian thiết lập kết nối SSL/TLS):** `672 ms` (chiếm 74.5% tổng thời gian request)
* **Latency:** `902 ms`
* **Kích thước dữ liệu nhận:** `9,832 bytes` (Body: 9,593 bytes; Header: 239 bytes)
* **Content-Type:** `text/html; charset=utf-8`

---

## 5. Phân Tích & Đánh Giá Chi Tiết

> [!WARNING]
> **Tỷ lệ lỗi báo động (55.56%):** Hơn một nửa số yêu cầu gửi tới máy chủ không thành công. Hệ thống không đáp ứng được mức tải 20 người dùng đồng thời trong kịch bản này.

1. **Về Thời gian phản hồi (Response Time):**
   - Thời gian phản hồi trung bình là **2.84 giây**, khá chậm đối với một trang tĩnh / landing page đơn giản.
   - Khoảng chênh lệch giữa **Min (175 ms)** và **Max (21,072 ms ~ 21 giây)** là cực lớn.
   - **Độ lệch chuẩn (Std. Dev.) lên tới 6,450 ms**, cho thấy hiệu năng máy chủ phản hồi rất thất thường và thiếu ổn định dưới tải.

2. **Về Tỷ lệ lỗi (Error Rate):**
   - Đạt mức **55.56%**. Do `httpbin.org` là dịch vụ công cộng miễn phí (public sandbox), nguyên nhân chính thường là:
     - Cơ chế giới hạn tốc độ truy cập (**Rate Limiting / HTTP 429 Too Many Requests**).
     - Máy chủ quá tải dẫn đến timeout kết nối (**Connection Timeout / HTTP 502/504**).
     - Lớp tường lửa ứng dụng (WAF / Cloudflare) ngắt kết nối do nhận diện hành vi gửi request đồng thời bất thường.

3. **Về Thông lượng (Throughput) & Kết nối mạng:**
   - Thông lượng chỉ đạt **9.7 requests/phút** (rất thấp), nguyên nhân do phần lớn các request phải chờ đợi timeout hoặc bị nghẽn ở khâu bắt tay thiết lập kết nối SSL/TLS (`Connect Time` ở mẫu test chiếm tới 672 ms trên tổng 902 ms).

---

## 6. Kết Luận & Khuyến Nghị

### Kết luận
- Endpoint `GET https://httpbin.org/` **không đạt tiêu chuẩn hiệu năng và độ ổn định** dưới mức tải đồng thời 20 luồng với tổng 180 request.
- Tỷ lệ lỗi **55.56%** và thời gian phản hồi tối đa vượt ngưỡng **21 giây** cho thấy hệ thống đã chạm ngưỡng bảo vệ hoặc kiệt quệ tài nguyên xử lý.

### Khuyến nghị
1. **Đối với môi trường kiểm thử:**
   - Không nên sử dụng server public của `httpbin.org` để chạy kiểm thử tải (load / stress test) vì sẽ kích hoạt cơ chế Rate Limiter của nhà cung cấp.
   - Khuyến nghị tự dựng một container httpbin độc lập bằng Docker (`docker run -p 80:80 kennethreitz/httpbin`) trên mạng local để có kết quả kiểm thử chính xác và khách quan.
2. **Đối với việc tinh chỉnh kịch bản JMeter:**
   - Thiết lập thời gian tăng tải (`Ramp-Up period`) hợp lý (ví dụ: tăng dần 20 thread trong vòng 30 - 60 giây thay vì gửi đồng loạt ngay lập tức).
   - Bổ sung `Timers` (như Constant Throughput Timer hoặc Gaussian Random Timer) để mô phỏng hành vi người dùng thực tế (*think time*), tránh gửi dồn dập khiến server kích hoạt bộ lọc bảo vệ.
