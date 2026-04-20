# Data Warehouse Project – DataCo Smart Supply Chain
## Kiến trúc Hệ thống

CSV → **Bronze Layer** → Silver Layer → Gold Layer → SSAS → BI/Reporting

## Cấu trúc Repository

```plaintext
DataWarehouseProject/
├── DWH_Project/
│   ├── Bronze.dtsx          # Member 1 
│   ├── Silver.dtsx          # Member 2 sẽ chỉnh sửa
│   ├── Gold.dtsx            # Member 3 sẽ chỉnh sửa
│   └── Main.dtsx            # Master Package (chạy toàn bộ pipeline)
├── README.md
```
### Cài đặt và chuẩn bị môi trường
- Cài đặt Visual Studio 2019 + SSIS Extension v4.6
- Kết nối SQL Server instance `QUYNHNHI\MSSQL2025` bằng Windows Authentication
- Tạo database `DataSupplyChain`
- Bật SQL Server and Windows Authentication mode
- Tạo Login và User cho Member 2, 3, 4 (với password `DWH2025`)

**Chạy toàn bộ pipeline:**
- Mở `Main.dtsx` → Nhấn Start (sẽ chạy Bronze → Silver → Gold theo thứ tự)

---

## Lưu ý quan trọng
- Luôn **TRUNCATE** bảng trước khi chạy lại package để tránh dữ liệu bị trùng.
- Các Member sau vui lòng **không sửa** package `Bronze.dtsx` trừ khi có thông báo từ Member 1.
- Nếu gặp vấn đề về kết nối từ xa, kiểm tra ZeroTier VPN và Port 1433.

```
