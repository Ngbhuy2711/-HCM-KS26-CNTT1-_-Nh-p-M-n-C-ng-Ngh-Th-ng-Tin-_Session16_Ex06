Phần 1. Phân tích nút thắt cổ chai
Em mở portal.greenlog.vn, nhấn F12 → Network, tick Disable cache và F5 để ghi một lượt tải trang từ đầu. Sau đó xem các cột Name, Status, Type, Size, Time và Waterfall. Em lọc Img để tìm ảnh lớn, lọc CSS/JS để xem file bắt buộc cho giao diện, và tìm các request màu đỏ để kiểm tra nguyên nhân thất bại.
Đề chỉ cho biết trang tải khoảng 20 giây chứ không đưa file log, nên chưa thể kết luận chính xác nút thắt. Bảng dưới đây là số liệu giả định để minh họa cách đọc Waterfall:
| Request |	Status |	size |	Nhận xét |
| :--- | :---: | :---: | ---: |
| main.css |	200 |	65 KB |	Tải nhanh; phục vụ dựng giao diện.|
| app.js |	200 |	420 KB |	Nên xem có chặn thời điểm hiển thị không. |
| map-route.png |	404 |	0 B |	Sai đường dẫn, bản đồ không thể hiện. |
| banner-HD.jpg |	200 |	14 MB |	Tải gần 16 giây, là nút thắt giả định. |
| /api/routes | 200 |	120 KB |	Dữ liệu lộ trình phản hồi bình thường trong ví dụ.|

Trên Waterfall, thanh kéo dài do Waiting/TTFB lớn gợi ý máy chủ/API phản hồi chậm; thanh Content Download dài đi kèm Size lớn gợi ý tải nhiều dữ liệu. Cũng cần nhìn thứ tự request và quan hệ phụ thuộc giữa CSS, JavaScript, ảnh để phân biệt thời gian tải với thời gian chờ.
 
Hình minh họa mô phỏng F12/Chrome, không phải ảnh chụp thực tế.
Phần 2. Quy trình 3 bước tối ưu
1. Sửa 404: mở request lỗi → Headers để đọc Request URL, kiểm tra đường dẫn file ảnh bản đồ trên server hoặc CDN và phân biệt lỗi thật với URL gõ sai. Sửa đường dẫn/cấu hình ở bản triển khai rồi tải lại xác nhận Status 200.
2. Nén ảnh: dùng WebP/AVIF cho ảnh phù hợp, giảm độ phân giải về kích thước thực tế, áp dụng responsive images và lazy loading cho ảnh bên dưới màn hình đầu tiên. Với dữ liệu bản đồ quan trọng cần tải đúng ảnh/tile cần thiết, không tải cả tập ảnh HD khi chưa dùng.
3. Bật Browser Cache: cho CSS/JS/ảnh tĩnh có tên file gắn hash một thời gian cache dài, còn dữ liệu lộ trình thay đổi thường xuyên dùng cache phù hợp hoặc không cache nếu cần cập nhật ngay. Kiểm tra lại ở Network cả lần tải mới và lần reload bình thường.
CSS/JavaScript thiết yếu phải tải thành công để trang render đúng. 200 OK là phổ biến ở lần tải mới; khi dùng cache hợp lệ, trình duyệt có thể dùng bản đã lưu hoặc nhận 304 Not Modified thay vì bắt buộc mọi lần đều phải là 200.
Phần 3. Kiểm thử mã 304 Not Modified
304 Not Modified cho biết trình duyệt đã hỏi lại bằng yêu cầu có điều kiện (ví dụ If-None-Match với ETag), và máy chủ xác nhận tài nguyên không thay đổi. Trình duyệt được dùng lại bản trong Cache mà không phải tải lại toàn bộ nội dung. Nhờ vậy dung lượng truyền tải giảm, nhất là với file CSS, JavaScript và ảnh lớn. Muốn quan sát cơ chế này, em bỏ chọn Disable cache sau khi đã kiểm thử lần tải mới.

