# Vấn đề :
[BIÊN BẢN SỰ CỐ KỸ THUẬT CAREPULSE - CA BỆNH ÁN #8821]  
• Bước 1 (DNS): Trình duyệt gửi trực tiếp chuỗi URL '[https://carepulse.vn/records/patient8821](https://carepulse.vn/records/patient8821)' tới máy chủ Web Server  
chứa dữ liệu để phân giải cổng mạng mà không cần thông qua hệ thống DNS.  
• Bước 2 (Kết nối): Trình duyệt tải ngay lập tức mã nguồn HTML về máy mà không cần thực hiện bước bắt tay TCP/TLS,  
vì giao thức HTTPS tự động kết nối mở không điều kiện.  
• Bước 3 (Mã phản hồi HTTP): Máy chủ cơ sở dữ liệu bệnh viện bị quá tải dẫn đến sập nguồn dịch vụ truy vấn SQL,  
khiến Server trả về mã HTTP '404 Not Found'. Bạn thực tập giải thích với bệnh nhân rằng: 'Do hồ sơ bệnh án của anh  
đã bị xóa vĩnh viễn khỏi hệ thống nên server mới trả về mã 404 này'.  
• Bước 4 (Dựng hình - Rendering): Trình duyệt bỏ qua cây DOM Tree và vẽ trực tiếp mã JavaScript thô lên màn hình.  
• Đề xuất xử lý của thực tập sinh: 'Em đã mở Chrome DevTools (F12) > Tab Elements, gõ trực tiếp chữ "Âm tính - Bình thường"  
vào thẻ <span> chỉ số đường huyết trên màn hình máy em. Như vậy dữ liệu bệnh án gốc trên Database máy chủ đã được  
sửa chuẩn xác thành công!'
# Yêu cầu :
Phần 1: Phân tích vấn đề

1. Chỉ ra 4 điểm sai lệch về mặt nguyên lý kỹ thuật trong bản giải trình của bạn thực tập sinh (về vai trò DNS, quy trình TCP/TLS, hiểu sai mã lỗi HTTP và hiểu lầm về việc sửa DOM trên Tab Elements).

2. Giải thích vì sao khi máy chủ cơ sở dữ liệu bị sập (crash dịch vụ backend), mã HTTP trả về đúng phải là 500 chứ không phải 404? Hậu quả tai hại đối với tâm lý bệnh nhân nếu thông báo sai thành 404 là gì?

Phần 2: Sửa lỗi & Hoàn thiện giải pháp

3.Viết lại biên bản giải trình kỹ thuật chuẩn xác theo đúng 4 bước vòng đời của HTTP Request khi bệnh nhân truy cập kết quả xét nghiệm.
4. Giải thích rõ ràng bằng nguyên lý bộ nhớ: Tại sao hành động sửa chữ trên Tab Elements của bạn thực tập sinh hoàn toàn không có giá trị lưu trữ trên máy chủ CarePulse?
# Cách làm :

PHẦN 1: PHÂN TÍCH VẤN ĐỀ
1. Chỉ ra 4 điểm sai lệch kỹ thuật trong bản giải trình của thực tập sinh:**

-   Sai lầm ở Bước 1 (DNS):Thực tập sinh cho rằng trình duyệt gửi thẳng URL tới Web Server mà không cần qua hệ thống DNS.
    
    -   _Thực tế:_ Trình duyệt bắt buộc phải thông qua hệ thống phân giải tên miền (DNS Lookup) để chuyển đổi tên miền dạng chữ    thành địa chỉ IP của máy chủ, vì máy tính chỉ có thể định tuyến mạng qua địa chỉ IP.
        
-   Sai lầm ở Bước 2 (Kết nối TCP/TLS): Nhận định trình duyệt tải ngay mã nguồn mà không cần bắt tay TCP/TLS vì HTTPS tự động mở kết nối.
    
    -   _Thực tế:_ Giao thức HTTPS luôn đòi hỏi quy trình thiết lập kết nối cực kỳ nghiêm ngặt gồm bắt tay 3 bước TCP (TCP 3-way Handshake) để mở kênh truyền tin cậy, tiếp theo là bắt tay TLS (TLS Handshake) để trao đổi chứng chỉ bảo mật và mã hóa dữ liệu trước khi bất kỳ gói tin HTTP nào được truyền đi.
        
- Sai lầm ở Bước 3 (Mã HTTP Status Code & Bản chất lỗi): Server sập nguồn cơ sở dữ liệu backend nhưng thực tập sinh lại cho rằng mã trả về là 404 Not Found và giải thích với bệnh nhân là "hồ sơ đã bị xóa vĩnh viễn".
    
    -   Thực tế:Khi cơ sở dữ liệu backend gặp sự cố sập nguồn hoặc lỗi mã nguồn xử lý phía server, mã lỗi HTTP chuẩn xác phải là 500 Internal Server Error (Lỗi nội bộ máy chủ). Mã 404 chỉ có nghĩa là không tìm thấy đường dẫn URL, hoàn toàn không đồng nghĩa với việc hồ sơ bị xóa vĩnh viễn.
        
-Sai lầm ở Bước 4 (DOM & DevTools): Cho rằng việc sửa chữ trực tiếp trên thẻ tại tab _Elements_ của Chrome DevTools sẽ cập nhật thành công dữ liệu gốc trên Database của máy chủ.
2. Giải thích mã lỗi HTTP và hậu quả tâm lý bệnh nhân:

-   Vì sao phải là mã 500 thay vì 404 khi sập Database?
    
    -   Mã 500 Internal Server Error phản ánh đúng bản chất rằng máy chủ (Web Server/Backend) vẫn nhận được yêu cầu (Request) nhưng không thể hoàn thành do lỗi hệ thống nội bộ (như mất kết nối Database, sập dịch vụ truy vấn).
        
    -   Trong khi đó, mã 404 Not Found khẳng định rằng máy chủ vẫn hoạt động bình thường nhưng đường dẫn URL tài nguyên đó không tồn tại. Dùng mã 404 trong trường hợp lỗi backend là sự nhầm lẫn nghiêm trọng về tầng kiến trúc hệ thống.
        
-   Hậu quả tai hại đối với tâm lý bệnh nhân:
    
    -   Khi nghe thông báo _"Hồ sơ bệnh án của anh đã bị xóa vĩnh viễn khỏi hệ thống (mã 404)"_, bệnh nhân sẽ rơi vào trạng thái hoảng loạn, lo sợ mất dữ liệu y tế quan trọng, mất niềm tin tuyệt đối vào tính bảo mật và sự chuyên nghiệp của nền tảng y tế CarePulse, dẫn đến các phản ứng khiếu nại gay gắt hoặc ảnh hưởng tâm lý nghiêm trọng.
        

PHẦN 2: SỬA LỖI & HOÀN THIỆN GIẢI PHÁP

3. Biên bản giải trình kỹ thuật chuẩn xác theo vòng đời HTTP Request:

[BIÊN BẢN SỰ CỐ KỸ THUẬT CAREPULSE - CA BỆNH ÁN #8821 (ĐÃ HIỆU CHỈNH)]
Bước 1 (DNS Lookup): Khi bệnh nhân click vào liên kết xem kết quả xét nghiệm, trình duyệt thực hiện truy vấn hệ thống phân giải tên miền (DNS) để dịch tên miền carepulse.vn thành địa chỉ IP chính xác của máy chủ hệ thống CarePulse.
Bước 2 (TCP/TLS Handshake):Trình duyệt tiến hành thiết lập kết nối bảo mật với máy chủ thông qua quá trình bắt tay 3 bước TCP (đảm bảo thông mạch đường truyền) và bắt tay TLS (xác thực chứng chỉ số và thiết lập mã hóa dữ liệu an toàn cho giao thức HTTPS).

Bước 3 Trình duyệt gửi HTTP GET Request kèm mã định danh bệnh nhân tới máy chủ. Do dịch vụ cơ sở dữ liệu backend gặp sự cố sập nguồn đột ngột, máy chủ không thể xử lý truy vấn dữ liệu và trả về mã phản hồi chuẩn 500 Internal Server Error kèm thông báo hệ thống đang bảo trì kỹ thuật tạm thời.

Bước 4 (Rendering Pipeline):** Do nhận được mã lỗi từ server thay vì dữ liệu JSON/HTML hợp lệ, trình duyệt hiển thị giao diện thông báo lỗi (hoặc màn hình trắng/trống). Đội ngũ kỹ thuật đã tiến hành khởi động lại cụm dịch vụ Database, khôi phục luồng phản hồi `200 OKvà render thành công cây DOM/CSSOM hiển thị đầy đủ kết quả xét nghiệm cho bệnh nhân.


4. Giải thích nguyên lý bộ nhớ: Vì sao sửa chữ trên DevTools không lưu được lên Server

Cơ chế phân vùng bộ nhớ (Client-side vs Server-side)
    
    -   Tab Elements trên Chrome DevTools chỉ hiển thị và cho phép chỉnh sửaCây DOM 
        
    -   Cơ sở dữ liệu (Database) và mã nguồn xử lý của hệ thống CarePulse lại được lưu trữ trên ổ đĩa cứng và bộ nhớ của máy chủ đặt tại trung tâm dữ liệu (Server-side) cách xa hàng trăm kilômét.
        
-  Hệ quả khi thao tác:
    
    -   Việc gõ chữ trực tiếp trên trình duyệt giống như việc bạn viết chì lên một tờ nháp cá nhân; nó không tạo ra bất kỳ yêu cầu mạng nào gửi về máy chủ.
        
    -   Ngay khi người dùng thực hiện thao tácF5  hoặc đóng tab trình duyệt, toàn bộ vùng nhớ RAM tạm thời này sẽ bị xóa sạch, và trình duyệt sẽ tải lại dữ liệu thật từ Server-side Database  Do đó, thao tác của thực tập sinh hoàn toàn vô giá trị về mặt kỹ thuật lưu trữ dữ liệu.
