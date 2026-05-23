<<<<<<< HEAD
# 🚗 Skill: Ghép Ảnh Giám Định Xe Ô Tô PTI – Xuất PDF

> Skill dành cho **Claude (Anthropic)** – tự động ghép ảnh giám định xe ô tô thành bộ PDF chuẩn định dạng PTI SOS.

---

## 📌 Mục đích

Skill nhận ảnh giám định xe và ảnh báo giá / bảng kê, tự động:

- Đọc **biển số xe** và **tên gara/địa điểm** từ ảnh báo giá hoặc bảng kê (Claude Vision)
- Nhận dạng **vị trí hư hỏng** trên từng ảnh hiện trạng xe (`0_*`)
- Gán **caption thông minh** theo tên file và nội dung báo giá
- Ghép ảnh vào khung **A4 dọc (6 ảnh/trang)** hoặc **A4 ngang (4 ảnh/trang)**
- Xuất file **`.pdf`** chuẩn định dạng PTI SOS có logo, header, caption, footer

---

## 🖼️ Kết quả đầu ra

| Thông tin | Chi tiết |
|---|---|
| **Định dạng** | PDF (JPEG quality 92%) |
| **Kích thước trang** | A4 @ 150 DPI |
| **Bố cục dọc** | 2 cột × 3 hàng = 6 ảnh/trang |
| **Bố cục ngang** | 2 cột × 2 hàng = 4 ảnh/trang |
| **Header** | Logo PTI SOS + biển số + tên gara + GĐV |
| **Caption** | Nền navy, chữ trắng, tên hạng mục |
| **Footer** | Tên phòng ban + số trang |

---

## 💬 Cách kích hoạt Skill

Gõ một trong các lệnh sau kèm ảnh upload:

```
tạo ảnh giám định dọc – GĐV Nguyễn Văn A
tạo ảnh giám định ngang – GĐV Trần Thị B
ghép ảnh giám định
tạo bản ảnh giám định xe
tạo ảnh gara
tạo ảnh giám định PDF
```

> **Mặc định**: Nếu không chỉ định hướng → tự động chọn **dọc 6 ảnh/trang**

---

## 📂 Yêu cầu file đầu vào

### 1. Ảnh báo giá / bảng kê (bắt buộc)
- Tên file gợi ý: `baogia.jpg`, `bangke.jpg`, `Baogia_14K240.jpg`
- Dùng để trích xuất: biển số xe, tên gara, danh sách hạng mục (STT → tên)

### 2. Logo PTI SOS (bắt buộc)
- Tên file bắt buộc: **`logo-ptisos.png`**
- Đặt cùng thư mục với các ảnh upload

### 3. Ảnh giám định xe (bắt buộc)
Đặt tên file theo quy tắc:

| Tên file | Caption tự động |
|---|---|
| `0_01.jpg` hoặc `0.01.jpg` | Ảnh đăng kiểm |
| `0_02.jpg` hoặc `0.02.jpg` | Ảnh tem đăng kiểm |
| `0_*.jpg` (các file hiện trạng khác) | Hiện trạng xe {vị trí} – nhận diện bằng AI |
| `1_*.jpg`, `2_*.jpg`, ... | Hạng mục STT 1, 2, ... (lấy tên từ báo giá) |
| `sk.jpg` | Số khung xe |
| `tdk.jpg` | Tem đăng kiểm |

### Thứ tự sắp xếp ảnh trong PDF:
```
0_01 → 0_02 → 0_* (hiện trạng) → 1_* → 2_* → ... → sk → tdk
```

---

## ⚙️ Thư viện Python cần thiết

```bash
pip install Pillow reportlab numpy --break-system-packages
```

| Thư viện | Mục đích |
|---|---|
| `Pillow` | Xử lý ảnh, bo góc, ghép layout |
| `reportlab` | Xuất file PDF |
| `numpy` | Xử lý pixel logo (xóa nền, đổi màu) |

---

## 🎨 Thông số thiết kế

### Màu sắc
| Thành phần | Màu |
|---|---|
| Header & Caption | Navy `(0, 32, 96)` |
| Viền ngoài | Xám `(180, 180, 180)` – 2px |
| Viền ô ảnh | Xám `(200, 200, 200)` – 1px |
| Footer text | Xám `(128, 128, 128)` |

### Kích thước (A4 dọc @ 150 DPI)
```
Canvas  : 1240 × 1753 px
Header  : cao 150 px (~3cm)
Footer  : cao 50 px
Caption : cao 46 px
Logo    : 89 × 89 px (1.5cm)
Bo góc  : radius 12 px
```

### Xử lý logo PTI SOS
- Pixel tối `R<80, G<80, B<80` → **trong suốt** (xóa nền đen)
- Pixel navy `R<60, G<60, B>100` → **trắng** (logo hiển thị trên nền navy)
- Màu cam/vàng → giữ nguyên

---

## 📐 Nguyên tắc ghép ảnh

- **Ảnh dọc** bị rotate 90° thành nằm ngang trước khi ghép
- Ảnh co/dãn **giữ tỷ lệ gốc**, căn giữa trong ô
- Ô thiếu ảnh → để trắng, không dịch chuyển ảnh trang sau
- Ghép liên tục từ trên-trái xuống dưới-phải

---

## 📝 Cấu trúc Header (3 dòng)

| Dòng | Nội dung | Font |
|---|---|---|
| Dòng 1 (đậm) | `Giám định chi tiết xe ô tô biển kiểm soát: {BKS}` | Arial Narrow Bold 18pt |
| Dòng 2 | `Địa điểm tại {Tên gara}` | Arial Narrow 18pt |
| Dòng 3 | `{Tên giám định viên}` | Arial Narrow 16pt, căn giữa toàn header |

> Với bảng kê: Dòng 1 thay bằng `Giám định hiện trường xe ô tô biển kiểm soát: {BKS}`

---

## 📁 Cấu trúc thư mục Skill

```
ghep-anh-gara/
├── SKILL.md          ← Hướng dẫn đầy đủ cho Claude
├── README.md         ← File này
└── example/
    ├── input/
    │   ├── baogia_sample.jpg
    │   ├── 0_1.jpg
    │   ├── 1_1.jpg
    │   └── logo-ptisos.png
    └── output/
        └── Giam_Dinh_Anh_14K24048.pdf
```

---

## 🔒 Lưu ý bảo mật & bản quyền

- **Logo PTI SOS** là tài sản nội bộ – không upload lên repository public
- **Ảnh giám định thật** của khách hàng chứa dữ liệu cá nhân – không commit vào git
- Thêm vào `.gitignore`:

```gitignore
# Ảnh thật và dữ liệu nhạy cảm
*.jpg
*.JPG
*.jpeg
*.JPEG
*.png
*.PNG
*.pdf

# Logo nội bộ
assets/logo-ptisos.png
example/input/
example/output/
```

---

## 🚀 Ví dụ sử dụng

**Kịch bản**: Giám định xe `14K-240.48` tại gara `Tùng Anh`, GĐV `Nguyễn Văn Hướng`

1. Upload các file: `baogia.jpg`, `logo-ptisos.png`, `0_1.jpg`, `0_2.jpg`, `1_1.jpg`, `2_1.jpg`
2. Gõ lệnh:
   ```
   tạo ảnh giám định dọc – GĐV Nguyễn Văn Hướng
   ```
3. Claude tự động:
   - Đọc biển số `14K-240.48` và tên gara `Tùng Anh` từ `baogia.jpg`
   - Nhận diện `0_1.jpg` → `Hiện trạng xe trước phải`
   - Ghép thành PDF 2 trang, 6 ảnh/trang
4. Tải về file: `Giam_Dinh_Anh_14K24048.pdf`

---

## 🛠️ Phát triển & đóng góp

- Skill này được thiết kế cho hệ thống **Claude Cowork / Claude.ai**
- Để cải tiến skill, chỉnh sửa file `SKILL.md` theo đúng cú pháp skill format
- Issues và Pull Requests được chào đón

---

## 📞 Liên hệ

> Phòng Giám định và Cứu hộ tại Quảng Ninh – PTI SOS
=======
# ghep-anh-gara
Skill ghép ảnh giám định xe ô tô PTI, xuất PDF chuẩn định dạng
>>>>>>> d5eda66a624e212429787497c023a492355e6b30
