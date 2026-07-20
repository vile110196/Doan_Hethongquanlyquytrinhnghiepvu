# CODE MERMAID (MÔ PHỎNG BPMN 2.0 CHUẨN KÝ HIỆU) CHO 10 QUY TRÌNH

Mã nguồn đã được rà soát và chỉnh sửa chuẩn xác từng ký hiệu theo chuẩn BPMN 2.0:
- **Start Event (Bắt đầu)**: Ký hiệu hình tròn nét đơn `(( ))`.
- **End Event (Kết thúc)**: Ký hiệu hình tròn nét đôi `((( )))`.
- **Task (Hoạt động/Nhiệm vụ)**: Ký hiệu hình chữ nhật bo góc `( )`.
- **Gateway (Cổng rẽ nhánh)**: Ký hiệu hình thoi `{ }`.

---

## 1. QUY TRÌNH CỐT LÕI

### 1.1 Quy trình đặt và giao hàng
```mermaid
flowchart LR
    subgraph KhachHang [Người mua]
        Start1((Bắt đầu)) --> A(Truy cập ứng dụng/website)
        A --> B(Tìm kiếm & tham khảo sản phẩm)
        B --> C(Lựa chọn SP & tiến hành đặt mua)
        C --> D(Nhập địa chỉ, mã giảm giá, chọn ĐVVC)
        D --> E(Xác nhận đặt hàng và Thanh toán)
        L(Nhận hàng và kiểm tra) --> L1{Có khiếu nại?}
        L1 -- Có --> L2(Yêu cầu hoàn tiền)
        L1 -- Không --> L3(Xác nhận đã nhận hàng)
    end
    subgraph HeThongThanhToan [Hệ thống thanh toán]
        E --> F{Thanh toán thành công?}
    end
    subgraph HeThongShopee [Hệ thống Shopee]
        F -- Lỗi --> F1(Báo lỗi thanh toán)
        F1 --> D
        F -- Thành công --> G(Ghi nhận đơn hàng)
        G --> H(Gửi thông báo đến Người bán)
        L3 --> M(Hoàn tất đơn hàng)
        M --> N(Chuyển tiền vào ví Người bán)
        N --> End1(((Kết thúc)))
    end
    subgraph NguoiBan [Người bán]
        H --> I(Tiếp nhận đơn, kiểm tra hàng hóa)
        I --> J(Đóng gói sản phẩm)
        J --> K(Bàn giao cho ĐVVC)
    end
    subgraph DVVC [Đơn vị vận chuyển]
        K --> K1(Tiếp nhận kiện hàng)
        K1 --> K2(Giao hàng đến địa chỉ khách)
        K2 --> L
    end
```

### 1.2 Quy trình người bán đăng bán và xử lý đơn hàng
```mermaid
flowchart LR
    subgraph NguoiBan [Người bán]
        Start1((Bắt đầu)) --> A(Đăng nhập Seller Center)
        A --> B(Tạo sản phẩm & Nhập thông tin)
        F(Xác nhận đơn hàng) --> G{Còn đủ hàng tồn?}
        G -- Không --> G1(Thỏa thuận hủy đơn với khách)
        G -- Có --> H(Đóng gói đúng quy cách & In phiếu)
        H --> I(Bàn giao cho ĐVVC)
    end
    subgraph HeThongShopee [Hệ thống Shopee]
        B --> C{Dữ liệu hợp lệ?}
        C -- Lỗi --> C1(Báo lỗi/Từ chối duyệt)
        C1 --> B
        C -- Hợp lệ --> D(Hiển thị sản phẩm trên sàn)
        E1(Khách đặt hàng) --> E(Gửi thông báo có đơn mới)
        E --> F
        L(Đơn giao thành công) --> M(Đối soát doanh thu)
        M --> N(Cộng tiền vào ví Người bán)
        N --> End1(((Kết thúc)))
    end
    subgraph DVVC [Đơn vị vận chuyển]
        I --> J(Vận chuyển và giao hàng)
        J --> L
    end
```

### 1.3 Quy trình thanh toán và hoàn tiền
```mermaid
flowchart LR
    subgraph NguoiMua [Người mua]
        Start1((Bắt đầu)) --> A(Lựa chọn phương thức thanh toán)
        A --> B{Thanh toán trước hay COD?}
        G(Gửi yêu cầu trả hàng / hoàn tiền) --> G1(Cung cấp bằng chứng)
    end
    subgraph NganHang [Cổng thanh toán / Ngân hàng]
        B -- Thanh toán trực tuyến --> C(Xác thực giao dịch)
        C --> C1{Thành công?}
        C1 -- Lỗi --> C2(Báo lỗi)
        C2 --> A
    end
    subgraph DVVC [Đơn vị vận chuyển]
        B -- COD --> D(Giao hàng)
        C1 -- Thành công --> D
    end
    subgraph HeThongShopee [Hệ thống Shopee]
        D --> E{Phát hiện lỗi/Tranh chấp?}
        E -- Có --> G
        E -- Không --> F(Chuyển tiền cho Người bán)
        G1 --> H(Tiếp nhận, kiểm tra bằng chứng)
        H --> I{Chấp nhận hoàn tiền?}
        I -- Có --> J(Hoàn tiền cho khách hàng)
        I -- Không --> K(Từ chối, hoàn tất giao dịch)
        K --> F
        F --> End1(((Kết thúc)))
        J --> End2(((Kết thúc)))
    end
    subgraph NguoiBan [Người bán]
        G -. Gửi bằng chứng đối chiếu .-> H
    end
```

---

## 2. QUY TRÌNH QUẢN LÝ

### 2.1 QUY TRÌNH QUẢN LÝ KIỂM SOÁT BẢO MẬT/GIAN LẬN (QT1)
```mermaid
flowchart LR
    subgraph HeThongAI [Hệ thống AI]
        Start1((Bắt đầu)) --> A(Quét dữ liệu hệ thống định kỳ)
        A --> B{Phát hiện bất thường?}
        B -- Không --> A
        B -- Có --> C(Phát tín hiệu cảnh báo rủi ro)
    end
    subgraph QuanLyRuiRo [Bộ phận Quản lý rủi ro]
        C --> D(Phân tích log và xác minh bằng chứng)
        D --> E{Là gian lận?}
        E -- Không --> E1(Bỏ qua cảnh báo)
        E -- Có --> F(Đưa ra hình thức phạt)
        G{Có khiếu nại?} -- Có --> H(Thẩm định lại bằng chứng)
        H --> I{Hợp lệ?}
        I -- Hợp lệ --> I1(Gỡ bỏ hình phạt)
        I -- Không hợp lệ --> I2(Duy trì hình phạt / Khóa vĩnh viễn)
        G -- Không --> End1(((Kết thúc)))
        E1 --> End2(((Kết thúc)))
        I1 --> End3(((Kết thúc)))
        I2 --> End4(((Kết thúc)))
    end
    subgraph NguoiDung [Người mua/Người bán]
        F --> F1(Tài khoản bị giới hạn/Khóa)
        F1 --> G
    end
```

### 2.2 QUY TRÌNH QUẢN LÝ HIỆU SUẤT VÀ CHẤT LƯỢNG NGƯỜI BÁN (QT2)
```mermaid
flowchart LR
    subgraph HeThong [Hệ thống]
        Start1((Bắt đầu)) --> A(Định kỳ thu thập chỉ số hiệu suất)
        A --> B(Tính toán điểm xếp hạng)
        B --> C{Đánh giá kết quả}
        C -- Tốt --> D(Gắn nhãn Shop Tốt / Đề xuất hỗ trợ)
        C -- Kém/Vi phạm --> E(Gửi cảnh báo & Giảm hiển thị)
    end
    subgraph BoPhanQuanLy [Bộ phận quản lý]
        D --> F(Duyệt cấp quyền Shopee Mall/Yêu thích)
        E --> G(Xem xét giải quyết khiếu nại nếu có)
        F --> End1(((Kết thúc)))
        G --> End2(((Kết thúc)))
    end
    subgraph NguoiBan [Người bán]
        D --> H(Nhận đặc quyền ưu đãi)
        E --> I(Bị giới hạn tính năng)
    end
```

### 2.3 QUY TRÌNH QUẢN LÝ VÀ ĐIỀU PHỐI CHIẾN DỊCH MARKETING (QT3)
```mermaid
flowchart LR
    subgraph BPMarketing [Bộ phận Marketing]
        Start1((Bắt đầu)) --> A(Lập kế hoạch chiến dịch & Ngân sách)
        A --> B(Mở cổng đăng ký chương trình)
        D(Theo dõi chỉ số thời gian thực) --> E{Đạt kỳ vọng?}
        E -- Không/Hết voucher sớm --> F(Điều chỉnh ngân sách/Giao diện linh hoạt)
        E -- Có/Ổn định --> G(Duy trì đến hết chiến dịch)
        F --> G
        G --> H(Tổng kết, xuất báo cáo)
        H --> End1(((Kết thúc)))
    end
    subgraph NguoiBan [Người bán]
        B --> C1(Đăng ký Sản phẩm và giảm giá)
    end
    subgraph HeThong [Hệ thống phân bổ]
        C1 --> C{Sản phẩm đủ điều kiện?}
        C -- Không --> C2(Từ chối)
        C -- Có --> C3(Phê duyệt & Áp dụng mã giảm)
        C3 --> D
    end
```

---

## 3. QUY TRÌNH HỖ TRỢ

### 3.1 Quy trình tuyển dụng và tiếp nhận nhân viên mới
```mermaid
flowchart LR
    subgraph PhongBan [Phòng ban & Trưởng BP]
        Start1((Bắt đầu)) --> A(Lập phiếu yêu cầu tuyển dụng)
        L(Đánh giá sau thử việc) --> M{Đạt yêu cầu?}
        M -- Có --> N(Ký hợp đồng chính thức)
        M -- Không --> O(Chấm dứt thử việc)
        N --> End1(((Kết thúc)))
        O --> End2(((Kết thúc)))
    end
    subgraph NhanSu [Bộ phận Nhân sự]
        A --> B{Duyệt ngân sách & Nhu cầu?}
        B -- Không --> B1(Từ chối tuyển)
        B -- Có --> C(Đăng tin & Sàng lọc hồ sơ)
        C --> D(Tổ chức phỏng vấn)
        D --> E{Ứng viên đạt?}
        E -- Không --> C
        E -- Có --> F(Gửi Offer Letter)
    end
    subgraph UngVien [Ứng viên / Nhân viên]
        F --> G{Đồng ý offer?}
        G -- Không --> C
        G -- Có --> H(Nộp hồ sơ nhận việc)
        H --> I(Tham gia đào tạo hội nhập)
        I --> L
    end
    subgraph ITHanhChinh [IT & Hành chính]
        H --> J(Cấp tài khoản & Trang thiết bị)
        J --> I
    end
```

### 3.2 Quy trình hỗ trợ kỹ thuật và cấp quyền hệ thống nội bộ
```mermaid
flowchart LR
    subgraph NhanVien [Nhân viên & Quản lý]
        Start1((Bắt đầu)) --> A(Tạo Ticket IT Helpdesk)
        C1{Yêu cầu cấp quyền mới?} -- Có --> D(Quản lý phê duyệt)
        H(Xác nhận đã xử lý) --> End1(((Kết thúc)))
    end
    subgraph IT_ATTT [IT & An Toàn Thông Tin]
        A --> B(Tiếp nhận & Phân loại Ticket)
        B --> C1
        D --> E{Đạt chuẩn ATTT?}
        E -- Không --> E1(Từ chối cấp quyền)
        E1 --> H
        C1 -- Không --> F(Quản trị viên xử lý sự cố)
        E -- Có --> F(Cấp quyền hệ thống)
        F --> G(Kiểm tra lại hoạt động)
        G --> H
    end
```

### 3.3 Quy trình mua sắm thiết bị và dịch vụ phục vụ hoạt động
```mermaid
flowchart LR
    subgraph PhongBan [Phòng ban]
        Start1((Bắt đầu)) --> A(Lập phiếu đề nghị mua sắm)
        K(Nghiệm thu thiết bị/dịch vụ) --> L{Đạt chất lượng?}
        L -- Có --> M(Đưa vào sử dụng & Báo thanh toán)
        L -- Không --> N(Yêu cầu NCC đổi trả)
        M --> End1(((Kết thúc)))
    end
    subgraph MuaSamTaiChinh [Mua sắm, Tài chính & Lãnh đạo]
        A --> B{Duyệt ngân sách?}
        B -- Không --> B1(Từ chối mua)
        B -- Có --> C(Xin báo giá từ NCC)
        C --> D(So sánh & Chọn NCC tốt nhất)
        D --> E(Lãnh đạo phê duyệt Hợp đồng/PO)
    end
    subgraph NhaCungCap [Nhà cung cấp]
        C -.-> C1(Gửi báo giá)
        C1 -.-> D
        E --> F(Giao hàng/Cung cấp dịch vụ)
        F --> K
        N --> F
    end
```

### 3.4 Quy trình đối soát và thanh toán cho nhà cung cấp
```mermaid
flowchart LR
    subgraph BoPhanSuDung [Bộ phận nhận hàng & Mua sắm]
        Start1((Bắt đầu)) --> A(Hoàn thành nghiệm thu hàng hóa)
        A --> B(Chuyển biên bản nghiệm thu cho Kế toán)
    end
    subgraph NCC_NganHang [Nhà Cung Cấp & Ngân Hàng]
        A --> C(Gửi hóa đơn VAT / Hồ sơ)
        I(Ngân hàng thực hiện chuyển khoản) --> J(Gửi thông báo biến động số dư)
    end
    subgraph KeToanTaiChinh [Kế toán & Tài chính]
        B --> D(Tiếp nhận hồ sơ thanh toán)
        C --> D
        D --> E{Đối chiếu có khớp không?}
        E -- Lỗi/Lệch --> F(Yêu cầu NCC xuất lại hóa đơn)
        F --> C
        E -- Hợp lệ --> G(Lập ủy nhiệm chi / Đề nghị thanh toán)
        G --> H(Ban lãnh đạo duyệt chi)
        H --> I
        J --> K(Ghi nhận sổ sách kế toán)
        K --> End1(((Kết thúc)))
    end
```
