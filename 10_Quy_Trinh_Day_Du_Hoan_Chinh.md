# TÀI LIỆU 10 QUY TRÌNH NGHIỆP VỤ HOÀN CHỈNH

Tài liệu này bao gồm mô tả chi tiết, phân tích kịch bản (Cases) và lưu đồ (BPMN Code) tối ưu dọc (Vertical Swimlane) cho 10 quy trình, phân nhóm chuẩn.

## 1. QUY TRÌNH CỐT LÕI

### 1.1 Quy trình đặt và giao hàng
**Actor (Tác nhân):** Người mua (Khách hàng), Người bán (Shop), Hệ thống Shopee, Đơn vị vận chuyển, Hệ thống thanh toán

**Phân tích các kịch bản (Cases):**
- Happy Case (Luồng lý tưởng): Khách hàng tìm thấy sản phẩm, đặt hàng với thông tin hợp lệ, thanh toán thành công. Người bán đóng gói và giao đúng hạn. Khách nhận hàng không có khiếu nại, tiền được chuyển cho người bán.
- Unhappy Case (Luồng thất bại): Khách hàng từ chối nhận hàng. Người bán không chuẩn bị hàng kịp. Giao hàng thất bại nhiều lần dẫn đến hoàn trả.
- Validation (Ràng buộc dữ liệu): Số lượng mua <= Tồn kho. Mã giảm giá hợp lệ. Thông tin giao hàng không để trống.
- Edge Case (Luồng ngoại lệ): Khách hủy đơn ngay lúc shipper quét mã lấy hàng. Kiện hàng thất lạc do thiên tai/hỏa hoạn.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    %% KHAI BÁO CÁC NODE TRONG POOL
    subgraph Pool_KhachHang [Khách Hàng]
        Start1((Bắt đầu))
        A(Truy cập<br>ứng dụng)
        B(Tìm kiếm &<br>tham khảo)
        C(Lựa chọn &<br>đặt mua)
        D(Nhập địa chỉ,<br>mã giảm giá)
        E(Xác nhận<br>đặt hàng)
        L(Nhận hàng &<br>kiểm tra)
        L1{Có khiếu nại?}
        L2(Yêu cầu<br>hoàn tiền)
        L3(Xác nhận<br>đã nhận)
    end
    
    subgraph Pool_Shopee [Shopee]
        subgraph Lane_TT [Thanh toán]
            F{Thanh toán<br>thành công?}
        end
        subgraph Lane_HT [Hệ thống]
            F1(Báo lỗi<br>thanh toán)
            G(Ghi nhận<br>đơn hàng)
            H(Gửi thông báo<br>đến Người bán)
            M(Hoàn tất<br>đơn hàng)
            N(Chuyển tiền<br>vào ví)
            End1(((Kết thúc)))
        end
    end
    
    subgraph Pool_NguoiBan [Người bán]
        I(Tiếp nhận &<br>kiểm tra)
        J(Đóng gói<br>sản phẩm)
        K(Bàn giao<br>cho ĐVVC)
    end
    
    subgraph Pool_DVVC [ĐVVC]
        K1(Tiếp nhận<br>kiện hàng)
        K2(Giao hàng<br>đến khách)
    end

    %% KẾT NỐI LUỒNG
    Start1 --> A --> B --> C --> D --> E --> F
    F -- Lỗi --> F1 --> D
    F -- Thành công --> G --> H --> I --> J --> K --> K1 --> K2 --> L
    L --> L1
    L1 -- Có --> L2
    L1 -- Không --> L3 --> M --> N --> End1

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_KhachHang fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_Shopee fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_TT fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_HT fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_NguoiBan fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_DVVC fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

### 1.2 Quy trình người bán đăng bán và xử lý đơn hàng
**Actor (Tác nhân):** Người bán, Hệ thống Shopee, Người mua, Đơn vị vận chuyển

**Phân tích các kịch bản (Cases):**
- Happy Case: Người bán tạo sản phẩm hợp lệ, được duyệt. Đóng gói đúng hạn, giao shipper thành công, nhận tiền.
- Unhappy Case: Sản phẩm bị từ chối do vi phạm. Người bán gửi nhầm hàng dẫn đến trả hàng. Shipper không đến lấy.
- Validation: Trọng lượng/kích thước trong giới hạn cho phép. Giá > 0.
- Edge Case: Nhấn "Chuẩn bị hàng" nhưng phát hiện kho trống, phải thỏa thuận khách hủy đơn.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_NguoiBan [Người bán]
        Start1((Bắt đầu))
        A(Đăng nhập<br>Seller Center)
        B(Tạo sản phẩm &<br>Nhập thông tin)
        F(Xác nhận<br>đơn hàng)
        G{Còn đủ<br>hàng tồn?}
        G1(Thỏa thuận<br>hủy đơn)
        H(Đóng gói &<br>In phiếu)
        I(Bàn giao<br>cho ĐVVC)
    end
    
    subgraph Pool_Shopee [Hệ thống Shopee]
        C{Dữ liệu<br>hợp lệ?}
        C1(Báo lỗi /<br>Từ chối)
        D(Hiển thị SP<br>trên sàn)
        E1(Khách<br>đặt hàng)
        E(Gửi thông báo<br>đơn mới)
        L(Đơn giao<br>thành công)
        M(Đối soát<br>doanh thu)
        N(Cộng tiền<br>vào ví)
        End1(((Kết thúc)))
    end
    
    subgraph Pool_DVVC [ĐVVC]
        J(Vận chuyển &<br>giao hàng)
    end

    %% KẾT NỐI
    Start1 --> A --> B --> C
    C -- Lỗi --> C1 --> B
    C -- Hợp lệ --> D
    E1 --> E --> F --> G
    G -- Không --> G1
    G -- Có --> H --> I --> J --> L --> M --> N --> End1

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_NguoiBan fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_Shopee fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_DVVC fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

### 1.3 Quy trình thanh toán và hoàn tiền
**Actor (Tác nhân):** Người mua, Người bán, Hệ thống Shopee, Cổng thanh toán, Ngân hàng hoặc ví điện tử, Đơn vị vận chuyển

**Phân tích các kịch bản (Cases):**
- Happy Case: Thanh toán online/COD thành công. Khách nhận hàng ưng ý hoặc hoàn tiền thành công khi có lỗi hợp lệ.
- Unhappy Case: Thẻ bị từ chối. Tranh chấp bị từ chối do người bán có bằng chứng đúng.
- Validation: Hạn mức thẻ >= Tổng đơn. Yêu cầu hoàn tiền trong thời gian quy định.
- Edge Case: Tranh chấp phức tạp, Shopee phải đền bù cho cả 2 bên.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_NguoiMua [Người mua]
        Start1((Bắt đầu))
        A(Chọn phương thức<br>thanh toán)
        B{Thanh toán trước<br>hay COD?}
        G(Gửi yêu cầu<br>hoàn tiền)
        G1(Cung cấp<br>bằng chứng)
    end
    
    subgraph Pool_DoiTac [Đối tác]
        subgraph Lane_NganHang [Ngân hàng]
            C(Xác thực<br>giao dịch)
            C1{Thành<br>công?}
            C2(Báo lỗi)
        end
        subgraph Lane_DVVC [Vận chuyển]
            D(Giao hàng)
        end
    end
    
    subgraph Pool_Shopee [Hệ thống Shopee]
        E{Phát hiện lỗi/<br>Tranh chấp?}
        F(Chuyển tiền<br>cho Người bán)
        H(Tiếp nhận &<br>kiểm tra)
        I{Chấp nhận<br>hoàn tiền?}
        J(Hoàn tiền<br>cho khách)
        K(Từ chối &<br>Hoàn tất)
        End1(((Kết thúc)))
        End2(((Kết thúc)))
    end
    
    subgraph Pool_NguoiBan [Người bán]
        G2(Gửi bằng chứng<br>đối chiếu)
    end

    %% KẾT NỐI
    Start1 --> A --> B
    B -- Trực tuyến --> C --> C1
    C1 -- Lỗi --> C2 --> A
    B -- COD --> D
    C1 -- Thành công --> D --> E
    E -- Có --> G --> G1 --> H
    G2 -.-> H
    E -- Không --> F --> End1
    H --> I
    I -- Có --> J --> End2
    I -- Không --> K --> F

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_NguoiMua fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_DoiTac fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_NganHang fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_DVVC fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_Shopee fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_NguoiBan fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

## 2. QUY TRÌNH QUẢN LÝ

### 2.1 QUY TRÌNH QUẢN LÝ KIỂM SOÁT BẢO MẬT/GIAN LẬN
**Actor (Tác nhân):** Bộ phận Quản lý rủi ro và gian lận, Hệ thống AI, Người mua/Người bán, Quản trị Shopee

**Phân tích các kịch bản (Cases):**
- Happy Case: Hệ thống AI quét trúng tài khoản lừa đảo. Bộ phận xác minh, khóa vĩnh viễn và bảo vệ nền tảng.
- Unhappy Case: AI nhận diện nhầm sinh viên chung wifi là tạo đơn ảo (False Positive). Bị khiếu nại ồ ạt.
- Validation: Risk Score > X. Device IP không trùng lặp đáng ngờ.
- Edge Case: Hacker dùng giả mạo IP cực tinh vi lọt qua bộ lọc, gây thất thoát mã giảm giá.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_Shopee [Shopee]
        subgraph Lane_AI [Hệ thống AI]
            Start1((Bắt đầu))
            A(Quét dữ liệu<br>định kỳ)
            B{Phát hiện<br>bất thường?}
            C(Cảnh báo<br>rủi ro)
        end
        subgraph Lane_Risk [Quản lý rủi ro]
            D(Phân tích &<br>xác minh)
            E{Là<br>gian lận?}
            E1(Bỏ qua<br>cảnh báo)
            F(Đưa ra<br>hình thức phạt)
            G{Có khiếu nại?}
            H(Thẩm định<br>lại)
            I{Hợp lệ?}
            I1(Gỡ bỏ<br>hình phạt)
            I2(Duy trì/Khóa<br>vĩnh viễn)
            End1(((Kết thúc)))
            End2(((Kết thúc)))
            End3(((Kết thúc)))
        end
    end
    
    subgraph Pool_NguoiDung [Người dùng]
        F1(Tài khoản<br>bị khóa)
    end

    %% KẾT NỐI
    Start1 --> A --> B
    B -- Không --> A
    B -- Có --> C --> D --> E
    E -- Không --> E1 --> End1
    E -- Có --> F --> F1 --> G
    G -- Không --> End2
    G -- Có --> H --> I
    I -- Hợp lệ --> I1 --> End3
    I -- Không --> I2 --> End2

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_Shopee fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_AI fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_Risk fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_NguoiDung fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

### 2.2 QUY TRÌNH QUẢN LÝ HIỆU SUẤT VÀ CHẤT LƯỢNG NGƯỜI BÁN
**Actor (Tác nhân):** Bộ phận quản lý đối tác/người bán, Người bán, Người mua, Hệ thống

**Phân tích các kịch bản (Cases):**
- Happy Case: Shop làm tốt được tăng hiển thị (Shopee Mall). Shop kém bị giảm hiển thị tự động.
- Unhappy Case: Thiên tai diện rộng làm nhiều shop không giao được hàng, hệ thống tự động trừ điểm nhầm.
- Validation: Tỷ lệ hủy đơn phải <= 10%. Điểm đánh giá (0-5 sao).
- Edge Case: Đối thủ tạo clone boom hàng để shop bị rớt hạng sao oan uổng.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_Shopee [Shopee]
        subgraph Lane_HT [Hệ thống]
            Start1((Bắt đầu))
            A(Thu thập<br>chỉ số)
            B(Tính toán<br>điểm xếp hạng)
            C{Đánh giá<br>kết quả}
            D(Gắn nhãn<br>Shop Tốt)
            E(Gửi cảnh báo &<br>Giảm hiển thị)
        end
        subgraph Lane_QL [Bộ phận quản lý]
            F(Duyệt quyền<br>Shopee Mall)
            G(Giải quyết<br>khiếu nại)
            End1(((Kết thúc)))
            End2(((Kết thúc)))
        end
    end
    
    subgraph Pool_NguoiBan [Người bán]
        H(Nhận đặc quyền<br>ưu đãi)
        I(Bị giới hạn<br>tính năng)
    end

    %% KẾT NỐI
    Start1 --> A --> B --> C
    C -- Tốt --> D --> H
    D --> F --> End1
    C -- Kém --> E --> I
    E --> G --> End2

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_Shopee fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_HT fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_QL fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_NguoiBan fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

### 2.3 QUY TRÌNH QUẢN LÝ VÀ ĐIỀU PHỐI CHIẾN DỊCH MARKTETING
**Actor (Tác nhân):** Bộ phận Marketing và quản lý chiến dịch, Bộ phận phân tích dữ liệu, Người bán, Hệ thống phân bổ voucher

**Phân tích các kịch bản (Cases):**
- Happy Case: Mã giảm giá tung ra, hệ thống chịu tải tốt, doanh thu đạt chỉ tiêu, báo cáo đẹp.
- Unhappy Case: Lượng truy cập vượt mức gây sập server. Người dùng không áp được mã, đánh giá app 1 sao.
- Validation: Ngân sách chiến dịch <= Ngân sách duyệt. Thời gian không chồng chéo phi logic.
- Edge Case: Setup nhầm giảm 50K thành giảm 500K không giới hạn, gây thiệt hại tài chính nghiêm trọng trước khi phát hiện.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_Shopee [Shopee]
        subgraph Lane_MKT [Bộ phận Marketing]
            Start1((Bắt đầu))
            A(Lập kế hoạch &<br>Ngân sách)
            B(Mở cổng<br>đăng ký)
            D(Theo dõi<br>chỉ số)
            E{Đạt kỳ vọng?}
            F(Điều chỉnh<br>ngân sách)
            G(Duy trì<br>chiến dịch)
            H(Tổng kết &<br>Báo cáo)
            End1(((Kết thúc)))
        end
        subgraph Lane_HT [Hệ thống phân bổ]
            C{Sản phẩm<br>đủ ĐK?}
            C2(Từ chối)
            C3(Phê duyệt &<br>Áp dụng)
        end
    end
    
    subgraph Pool_NguoiBan [Người bán]
        C1(Đăng ký<br>Sản phẩm)
    end

    %% KẾT NỐI
    Start1 --> A --> B --> C1 --> C
    C -- Không --> C2
    C -- Có --> C3 --> D --> E
    E -- Không --> F --> G
    E -- Có --> G --> H --> End1

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_Shopee fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_MKT fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_HT fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_NguoiBan fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

## 3. QUY TRÌNH HỖ TRỢ

### 3.1 Quy trình tuyển dụng và tiếp nhận nhân viên mới
**Actor (Tác nhân):** Phòng ban có nhu cầu, Trưởng bộ phận, Bộ phận Nhân sự, Ứng viên, Bộ phận IT, Hành chính, Nhân viên mới

**Phân tích các kịch bản (Cases):**
- Happy Case: Tìm được ứng viên giỏi, nhận việc đúng hạn. IT cấp quyền đầy đủ, thử việc thành công.
- Unhappy Case: Không tìm được ai phải đăng lại. Ứng viên đậu nhưng từ chối offer. Thử việc không đạt.
- Validation: Ngân sách lương <= Ngân sách năm của phòng ban. Hồ sơ nhân sự phải đầy đủ bằng cấp.
- Edge Case: Ứng viên nhận việc 1 ngày xong biến mất không báo trước, công ty đã sắm thiết bị và cấp quyền.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_CongTy [Công Ty]
        subgraph Lane_PhongBan [Phòng ban]
            Start1((Bắt đầu))
            A(Lập phiếu<br>yêu cầu)
            L(Đánh giá<br>thử việc)
            M{Đạt<br>yêu cầu?}
            N(Ký hợp đồng<br>chính thức)
            O(Chấm dứt<br>thử việc)
            End1(((Kết thúc)))
        end
        subgraph Lane_NhanSu [Nhân sự]
            B{Duyệt ngân sách &<br>Nhu cầu?}
            B1(Từ chối)
            C(Đăng tin &<br>Sàng lọc)
            D(Tổ chức<br>phỏng vấn)
            E{Ứng viên đạt?}
            F(Gửi Offer)
        end
        subgraph Lane_IT_HC [IT & Hành chính]
            J(Cấp tài khoản &<br>Trang thiết bị)
        end
    end
    
    subgraph Pool_UngVien [Ứng viên / Nhân viên]
        G{Đồng ý<br>offer?}
        H(Nộp hồ sơ<br>nhận việc)
        I(Tham gia<br>đào tạo)
    end

    %% KẾT NỐI
    Start1 --> A --> B
    B -- Không --> B1
    B -- Có --> C --> D --> E
    E -- Không --> C
    E -- Có --> F --> G
    G -- Không --> C
    G -- Có --> H --> J --> I --> L --> M
    M -- Có --> N --> End1
    M -- Không --> O --> End1

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_CongTy fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_PhongBan fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_NhanSu fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_IT_HC fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_UngVien fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

### 3.2 Quy trình hỗ trợ kỹ thuật và cấp quyền hệ thống nội bộ
**Actor (Tác nhân):** Nhân viên Shopee, Quản lý trực tiếp, IT Helpdesk, Quản trị viên hệ thống, An toàn thông tin

**Phân tích các kịch bản (Cases):**
- Happy Case: Nhân viên báo lỗi máy tính, IT tiếp nhận và sửa trong 30p, ticket đóng lại thành công.
- Unhappy Case: Sự cố phần cứng nặng phải chuyển bảo hành. Xin quyền bị ATTT từ chối do trái quy định.
- Validation: Quyền được xin phải khớp với chức danh. Ticket phải có ảnh đính kèm mô tả lỗi.
- Edge Case: Máy chủ bị ransomware tấn công diện rộng, hàng loạt ticket IT mở cùng lúc làm sập hệ thống Helpdesk.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_CongTy [Công Ty]
        subgraph Lane_NhanVien [Nhân viên]
            Start1((Bắt đầu))
            A(Tạo Ticket<br>Helpdesk)
            C1{Yêu cầu<br>cấp quyền?}
            D(Quản lý<br>phê duyệt)
            H(Xác nhận<br>đã xử lý)
            End1(((Kết thúc)))
        end
        subgraph Lane_IT [IT & ATTT]
            B(Tiếp nhận &<br>Phân loại)
            E{Đạt chuẩn<br>ATTT?}
            E1(Từ chối<br>cấp quyền)
            F(Xử lý sự cố /<br>Cấp quyền)
            G(Kiểm tra lại<br>hoạt động)
        end
    end

    %% KẾT NỐI
    Start1 --> A --> B --> C1
    C1 -- Có --> D --> E
    E -- Không --> E1 --> H
    C1 -- Không --> F
    E -- Có --> F --> G --> H --> End1

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_CongTy fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_NhanVien fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_IT fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

### 3.3 Quy trình mua sắm thiết bị và dịch vụ phục vụ hoạt động
**Actor (Tác nhân):** Phòng ban đề xuất, Trưởng bộ phận, Bộ phận Mua sắm, Bộ phận Tài chính, Ban lãnh đạo, Nhà cung cấp, Hành chính

**Phân tích các kịch bản (Cases):**
- Happy Case: Tìm được NCC rẻ, hàng chuẩn, giao đúng hạn. Nghiệm thu suôn sẻ, bàn giao đủ.
- Unhappy Case: Hết ngân sách. NCC giao trễ hẹn. Hàng lỗi phải yêu cầu đổi trả.
- Validation: Giá mua <= Ngân sách duyệt. Phải có tối thiểu 3 báo giá cạnh tranh.
- Edge Case: NCC phá sản ngay sau khi nhận tiền cọc, công ty phải truy thu pháp lý.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_CongTy [Công Ty]
        subgraph Lane_PhongBan [Phòng ban]
            Start1((Bắt đầu))
            A(Lập phiếu<br>đề nghị)
            K(Nghiệm thu<br>thiết bị)
            L{Đạt<br>chất lượng?}
            M(Đưa vào<br>sử dụng)
            N(Yêu cầu<br>đổi trả)
            End1(((Kết thúc)))
        end
        subgraph Lane_MuaSam [Mua sắm & Tài chính]
            B{Duyệt<br>ngân sách?}
            B1(Từ chối mua)
            C(Xin báo giá)
            D(So sánh &<br>Chọn NCC)
            E(Lãnh đạo<br>phê duyệt PO)
        end
    end
    
    subgraph Pool_NCC [Nhà cung cấp]
        C1(Gửi báo giá)
        F(Giao hàng)
    end

    %% KẾT NỐI
    Start1 --> A --> B
    B -- Không --> B1
    B -- Có --> C -.-> C1 -.-> D --> E --> F --> K --> L
    L -- Không --> N --> F
    L -- Có --> M --> End1

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_CongTy fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_PhongBan fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_MuaSam fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_NCC fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

### 3.4 Quy trình đối soát và thanh toán cho nhà cung cấp
**Actor (Tác nhân):** Nhà cung cấp, Mua sắm, Bộ phận sử dụng, Hành chính, Kế toán, Tài chính, Ngân hàng

**Phân tích các kịch bản (Cases):**
- Happy Case: Hồ sơ khớp 100%, sếp duyệt nhanh, ngân hàng chuyển tiền trong ngày, NCC hài lòng.
- Unhappy Case: Hóa đơn sai thông tin mã số thuế công ty. Ngân hàng lỗi bảo trì làm chậm giao dịch.
- Validation: Số tiền hóa đơn = Số tiền nghiệm thu. Hóa đơn phải hợp lệ (e-invoice).
- Edge Case: Bị hacker đánh tráo email hóa đơn của NCC sang tài khoản lừa đảo, kế toán chuyển nhầm tiền tỷ.

**Lưu đồ BPMN (Mermaid Code):**
```mermaid
flowchart TD
    classDef default fill:#ffffff,stroke:#000000,color:#000000;
    subgraph Pool_CongTy [Công Ty]
        subgraph Lane_MuaSam [Nhận hàng & Mua sắm]
            Start1((Bắt đầu))
            A(Hoàn thành<br>nghiệm thu)
            B(Chuyển biên bản<br>cho Kế toán)
        end
        subgraph Lane_KeToan [Kế toán & Tài chính]
            D(Tiếp nhận hồ sơ)
            E{Đối chiếu<br>có khớp?}
            F(Yêu cầu NCC<br>xuất lại HĐ)
            G(Lập ủy nhiệm chi)
            H(Lãnh đạo<br>duyệt chi)
            K(Ghi nhận<br>sổ sách)
            End1(((Kết thúc)))
        end
    end
    
    subgraph Pool_DoiTac [Nhà Cung Cấp & Ngân Hàng]
        C(Gửi hóa đơn)
        I(Ngân hàng<br>chuyển khoản)
        J(Gửi thông báo<br>số dư)
    end

    %% KẾT NỐI
    Start1 --> A --> B --> D
    A --> C --> D
    D --> E
    E -- Lỗi --> F --> C
    E -- Hợp lệ --> G --> H --> I --> J --> K --> End1

    %% Ép nền trắng viền đen cho các Pool/Lane (Bỏ nền vàng)
    style Pool_CongTy fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_MuaSam fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Lane_KeToan fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
    style Pool_DoiTac fill:#ffffff,stroke:#000000,color:#000000,stroke-width:2px;
```

---

