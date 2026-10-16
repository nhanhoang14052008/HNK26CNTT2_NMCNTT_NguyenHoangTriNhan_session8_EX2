# 1. Phân tích Input / Output

### Input — Văn bản thô

Nội dung Markdown chưa được render, bao gồm:

-   Tiêu đề và các đoạn văn bản giải thích.
    
-   Câu lệnh Terminal và mã Python.
    
-   Dữ liệu thông số cảm biến để tạo bảng.
    
-   Các công việc kiểm tra thiết bị dưới dạng Task List.
    
-   Ký tự Escape `\` để hiển thị ký tự đặc biệt theo đúng yêu cầu.
    

### Output — Giao diện đã render

Tài liệu hiển thị rõ ràng với:

-   Tiêu đề phân cấp và nội dung dễ đọc.
    
-   Câu lệnh ngắn như `python3 main.py` được định dạng Inline Code.
    
-   Mã Python có tô màu cú pháp.
    
-   Bảng so sánh cảm biến cũ và mới.
    
-   Danh sách công việc có ô đánh dấu hoàn thành.
# 2. Source code

# HƯỚNG DẪN CÀI ĐẶT CẢM BIẾN ĐỘ ẨM CHO TRANG TRẠI LAN

## 1. Mục tiêu

Lắp đặt cảm biến độ ẩm đất để theo dõi tình trạng đất trồng và tự động điều khiển máy bơm nước khi đất quá khô.

Hệ thống gồm:

-   Cảm biến độ ẩm đất.
-   Bộ điều khiển Raspberry Pi.
-   Module relay điều khiển máy bơm.
-   Nguồn điện phù hợp với từng thiết bị.

> **Lưu ý an toàn:** Ngắt nguồn điện trước khi đấu nối. Không kết nối máy bơm điện áp cao trực tiếp với chân GPIO của Raspberry Pi.

## 2. Chuẩn bị môi trường

### Bước 1: Kiểm tra Python

Mở Terminal và nhập lệnh sau để kiểm tra phiên bản Python:

`python3 --version` 

Nếu máy chưa có Python 3, hãy cài đặt theo hướng dẫn dành cho hệ điều hành đang sử dụng.

### Bước 2: Tạo thư mục dự án

Nhập lần lượt các câu lệnh vào Terminal:

`mkdir -p ~/trang-trai-lan
cd ~/trang-trai-lan
python3 -m venv .venv
source .venv/bin/activate` 

Các lệnh trên tạo thư mục dự án, tạo môi trường Python riêng và kích hoạt môi trường đó.

### Bước 3: Tạo tệp chương trình

Tạo tệp `main.py` để lưu chương trình điều khiển máy bơm:

`nano main.py` 

Tên tệp `main.py` và lệnh `python3 main.py` là các ví dụ về Inline Code trong tài liệu hướng dẫn.

## 3. Bảng so sánh cảm biến

Bảng dưới đây minh họa sự khác biệt giữa cảm biến độ ẩm loại cũ và loại mới. Các thông số là ví dụ tham khảo, cần kiểm tra datasheet của thiết bị thực tế trước khi lắp đặt.

Tiêu chí

Cảm biến loại cũ

Cảm biến loại mới

Công nghệ

Điện trở tiếp xúc

Điện dung

Khả năng chống ăn mòn

Thấp hơn

Tốt hơn

Độ ổn định lâu dài

Có thể suy giảm

Thường ổn định hơn

Ngõ ra

Analog hoặc digital

Thường là analog

Bảo trì

Có thể cần thay thế thường xuyên hơn

Thường ít bảo trì hơn

Chi phí

Thường thấp

Thường cao hơn

Khuyến nghị

Phù hợp thử nghiệm cơ bản

Phù hợp theo dõi đất lâu dài

**Đề xuất:** Trang trại Lan có thể cân nhắc cảm biến điện dung để giảm ảnh hưởng của hiện tượng ăn mòn điện cực. Tuy nhiên, cần hiệu chuẩn theo loại đất và điều kiện thực tế.

## 4. Mã Python điều khiển máy bơm

Chương trình dưới đây minh họa logic bật máy bơm khi độ ẩm thấp hơn ngưỡng cài đặt.

Lưu ý: Hàm `read_moisture_percent()` là hàm mẫu. Cần thay bằng mã đọc cảm biến thực tế, bao gồm ADC nếu cảm biến xuất tín hiệu analog. Ví dụ sử dụng thư viện `gpiozero` để điều khiển relay trên Raspberry Pi.

`from time import sleep
from gpiozero import OutputDevice

# Chan GPIO ket noi relay (vi du)
RELAY_PIN = 17

# Nguong do am (%)
MOISTURE_THRESHOLD = 35

# Relay active_high=True la cau hinh vi du.
# Kiem tra relay thuc te truoc khi cap nguon may bom.
pump = OutputDevice(
    RELAY_PIN,
    active_high=True,
    initial_value=False
)


def read_moisture_percent():
    """
    Thay bang ham doc cam bien thuc te.
    Gia tri tra ve phai nam trong khoang 0-100%.
    """
    raise NotImplementedError(
        "Hay ket noi va hieu chuan cam bien truoc khi chay."
    )


def control_pump(moisture):
    """Dieu khien may bom theo do am dat."""
    if moisture < MOISTURE_THRESHOLD:
        pump.on()
        print(f"Dat kho: {moisture:.1f}%. Bat may bom.")
    else:
        pump.off()
        print(f"Do am du: {moisture:.1f}%. Tat may bom.")


try:
    while True:
        moisture = read_moisture_percent()
        control_pump(moisture)
        sleep(5)

except KeyboardInterrupt:
    print("Dung chuong trinh.")

finally:
    pump.off()
    pump.close()` 

### Giải thích chương trình

-   `RELAY_PIN`: chân GPIO điều khiển relay.
-   `MOISTURE_THRESHOLD`: ngưỡng độ ẩm để quyết định bật máy bơm.
-   `read_moisture_percent()`: hàm cần được hoàn thiện theo loại cảm biến thực tế.
-   `control_pump()`: bật máy bơm khi đất khô và tắt khi độ ẩm đạt ngưỡng.
-   `sleep(5)`: chờ 5 giây trước khi đọc tiếp.

**Quan trọng:** Chương trình chưa thể điều khiển hệ thống thực tế cho đến khi hoàn thiện hàm đọc cảm biến. Cần kiểm tra logic relay, trạng thái an toàn khi khởi động và nguồn điện của máy bơm trước khi vận hành.

## 5. Kiểm tra và chạy thử thiết bị

Thực hiện lần lượt các công việc sau:

-   Kiểm tra nguồn cấp cho Raspberry Pi.
-   Kiểm tra dây tín hiệu và dây nguồn của cảm biến.
-   Xác nhận cảm biến tương thích với bộ chuyển đổi ADC nếu sử dụng ngõ ra analog.
-   Kiểm tra chân GPIO kết nối với relay.
-   Hiệu chuẩn cảm biến trong điều kiện đất khô và đất ẩm.
-   Hoàn thiện hàm `read_moisture_percent()`.
-   Kiểm tra relay khi chưa kết nối máy bơm.
-   Chạy thử chương trình trong điều kiện an toàn.
-   Xác nhận máy bơm tự tắt khi độ ẩm đạt ngưỡng.
-   Kiểm tra cơ chế tắt máy bơm khi xảy ra lỗi cảm biến.

Sau khi hoàn tất một công việc, có thể đổi `- [ ]` thành `- [x]` trong tệp Markdown để đánh dấu đã hoàn thành.

## 6. Lệnh chạy chương trình

Mở Terminal tại thư mục dự án và kích hoạt môi trường Python nếu chưa kích hoạt:

`cd ~/trang-trai-lan
source .venv/bin/activate
python3 main.py` 

Để dừng chương trình, nhấn `Ctrl+C`.

Lưu ý rằng chương trình mẫu sẽ báo lỗi `NotImplementedError` cho đến khi hàm đọc cảm biến được triển khai. Đây là chủ đích nhằm tránh bật máy bơm dựa trên dữ liệu chưa có.

## 7. Quy tắc hiển thị Markdown và Escape

-   Dùng Inline Code cho tên biến như `MOISTURE_THRESHOLD`, tên tệp như `main.py` và câu lệnh như `python3 main.py`.
-   Dùng Code Block có chỉ định ngôn ngữ `python` hoặc `bash` để trình bày mã nguồn và câu lệnh Terminal.
-   Dùng Table để so sánh thông số cảm biến.
-   Dùng Task List để theo dõi tiến độ kiểm tra thiết bị.

Khi cần hiển thị ký tự đặc biệt trong văn bản Markdown, sử dụng dấu gạch chéo ngược `\` để Escape. Ví dụ, nhập `\*` để hiển thị dấu sao * và nhập `\#` để hiển thị dấu thăng # thay vì tạo định dạng Markdown.

## 8. Kết luận

Tài liệu đã minh họa cách tổ chức hướng dẫn kỹ thuật bằng Markdown, giúp người đọc phân biệt nội dung giải thích, câu lệnh Terminal, mã Python, dữ liệu dạng bảng và danh sách kiểm tra.

Trước khi triển khai tại trang trại Lan, cần hoàn thiện phần đọc cảm biến, hiệu chuẩn ngưỡng độ ẩm và kiểm tra an toàn điện để bảo đảm hệ thống vận hành đúng thực tế.


