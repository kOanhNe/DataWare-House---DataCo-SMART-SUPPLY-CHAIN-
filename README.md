# Data Warehouse Project – DataCo Smart Supply Chain
## Kiến trúc Hệ thống

CSV → **Bronze Layer** → Silver Layer → Gold Layer → SSAS → BI/Reporting

## Cấu trúc Repository

```plaintext
DataWarehouseProject/
├── DWH_Project/
│   ├── Bronze.dtsx          # Member 1 
│   ├── Silver.dtsx          # Member 2 
│   ├── Gold.dtsx            # Member 3 
│   └── Main.dtsx            # Master Package (chạy toàn bộ pipeline)
├── README.md
```
### Cài đặt và chuẩn bị môi trường
- Cài đặt Visual Studio 2019 + SSIS Extension v4.6
- Kết nối SQL Server instance `QUYNHNHI\MSSQL2025` bằng Windows Authentication
- Tạo database `DataSupplyChain`
- Bật SQL Server and Windows Authentication mode

**Chạy toàn bộ pipeline:**
- Mở `Main.dtsx` → Nhấn Start (sẽ chạy Bronze → Silver → Gold theo thứ tự)

---

## Lưu ý quan trọng
- Luôn **TRUNCATE** bảng trước khi chạy lại package để tránh dữ liệu bị trùng.

```
