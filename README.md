```markdown
# Data Warehouse Project – DataCo Smart Supply Chain
## Kiến trúc Hệ thống

CSV → **Bronze Layer** → Silver Layer → Gold Layer → SSAS → BI/Reporting

### Trạng thái hiện tại

| Layer   | Trạng thái          | Người thực hiện | Ghi chú |
|---------|---------------------|------------------|--------|
| Bronze  | **Hoàn thành**      | Member 1        | Dữ liệu đã load đầy đủ |
| Silver  | Chưa thực hiện      | Member 2        | Sẽ bắt đầu từ Bronze |
| Gold    | Chưa thực hiện      | Member 3        | Sẽ bắt đầu từ Silver |
| BI      | Chưa thực hiện      | Member 4        | Sẽ bắt đầu từ Gold |

---

## Cấu trúc Repository

```plaintext
DataWarehouseProject/
├── DWH_Project/
│   ├── Bronze.dtsx          # Member 1 - ĐÃ HOÀN THÀNH
│   ├── Silver.dtsx          # Member 2 sẽ chỉnh sửa
│   ├── Gold.dtsx            # Member 3 sẽ chỉnh sửa
│   └── Main.dtsx            # Master Package (chạy toàn bộ pipeline)
├── README.md

## Bronze Layer
Thực hiện toàn bộ phần **Ingestion** dữ liệu từ file CSV vào Bronze Layer theo các bước sau:

### 1. Cài đặt và chuẩn bị môi trường
- Cài đặt Visual Studio 2019 + SSIS Extension v4.6
- Kết nối SQL Server instance `QUYNHNHI\MSSQL2025` bằng Windows Authentication
- Tạo database `DataSupplyChain`
- Bật SQL Server and Windows Authentication mode
- Tạo Login và User cho Member 2, 3, 4 (với password `DWH2025`)

### 2. Cấu hình kết nối từ xa 
- Bật Remote Access trên SQL Server
- Enable TCP/IP và mở Port 1433
- Cài đặt ZeroTier VPN với Network ID: `2873fd00f27fef1d`
- ZeroTier IP của server: `10.159.203.9`
- Cung cấp thông tin kết nối cho các thành viên còn lại:
  - Server: `10.159.203.9\MSSQL2025,1433`
  - Authentication: SQL Server Authentication
  - Login: `member2` / `member3` / `member4`
  - Password: `DWH2025`

### 3. Thiết kế và tạo bảng Bronze
- Phân tích file CSV (53 cột) và chia thành 3 bảng:
  - `bronze_order` (36 cột)
  - `bronze_shipping` (9 cột)
  - `bronze_customer` (11 cột)
- Chạy script SQL để tạo 3 bảng trong database `DataSupplyChain`

### 4. Thiết kế SSIS Package Bronze
- Tạo Connection Manager:
  - Flat File Source kết nối đến file `DataCoSupplyChainDataset.csv`
  - OLE DB Destination kết nối đến SQL Server
- Điều chỉnh OutputColumnWidth cho các cột dài (Customer Street, Product Description, Product Image…)
- Thiết kế Control Flow: 3 Data Flow Task chạy tuần tự  
  **Data Flow Task - Order → Shipping → Customer**
- Mapping đầy đủ cột từ CSV sang 3 bảng Bronze tương ứng

### 5. Chạy và kiểm tra kết quả
- TRUNCATE bảng trước khi chạy lại (nếu cần)
- Chạy package `Bronze.dtsx`
- Kiểm tra số dòng dữ liệu (mong đợi **180.519** dòng mỗi bảng)

**Kết quả đã đạt được:**
- Bronze Layer hoàn thành
- Dữ liệu được load đầy đủ và chính xác
- Package `Bronze.dtsx` chạy ổn định
- Kết nối từ xa đã được thiết lập sẵn cho các thành viên khác

---

## Hướng dẫn cho Member 2, 3, 4 (Làm tiếp) - sau này xoá 

**Member 2 (Silver Layer):**
- Kết nối SSMS / Visual Studio bằng thông tin kết nối từ xa ở trên
- Mở package `Silver.dtsx`
- Lấy dữ liệu từ 3 bảng Bronze
- Thực hiện Cleaning (xử lý NULL, duplicate), chuẩn hóa datatype, transform
- Hoàn thiện và chạy `Silver.dtsx`

**Member 3 (Gold Layer):**
- Phụ thuộc vào kết quả của Member 2
- Thiết kế Star Schema, tạo Fact & Dimension tables
- Load dữ liệu từ Silver sang Gold
- Hoàn thiện package `Gold.dtsx`

**Member 4 (SSAS + BI):**
- Phụ thuộc vào Gold Layer
- Xây dựng SSAS Model, tạo relationship, thiết kế báo cáo Power BI

**Chạy toàn bộ pipeline:**
- Mở `Main.dtsx` → Nhấn Start (sẽ chạy Bronze → Silver → Gold theo thứ tự)

---

## Kiểm tra Bronze Layer

```sql
USE DataSupplyChain;
GO

SELECT 'bronze_order'    AS TableName, COUNT(*) AS TotalRows FROM [dbo].[bronze_order]
UNION ALL
SELECT 'bronze_shipping', COUNT(*) FROM [dbo].[bronze_shipping]
UNION ALL
SELECT 'bronze_customer', COUNT(*) FROM [dbo].[bronze_customer];
```

**Kết quả mong đợi:** Mỗi bảng có **180.519** dòng.

---

## Lưu ý quan trọng
- Luôn **TRUNCATE** bảng trước khi chạy lại package để tránh dữ liệu bị trùng.
- Các Member sau vui lòng **không sửa** package `Bronze.dtsx` trừ khi có thông báo từ Member 1.
- Nếu gặp vấn đề về kết nối từ xa, kiểm tra ZeroTier VPN và Port 1433.

**Hoàn thành bởi Member 1**  
**Cập nhật lần cuối:** 14/04/2026

```
