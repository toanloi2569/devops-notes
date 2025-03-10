# Identifying Issues 

### Đếm số lượng row từng bảng
Ước tính số lượng rows trong từng bảng (không tuyệt đối chính xác)
```sql
SELECT Table_name,SUM(table_rows) As RowCount 
     FROM information_schema.tables 
     WHERE TABLE_SCHEMA = 'YourDBName';
```

Để xem số lượng row chính xác, sử dụng lệnh `SELECT COUNT(*) FROM table_name;`
Sử dụng lệnh sau để generate lệnh `SELECT COUNT(*) FROM table_name;` cho tất cả các bảng
```sql
select Concat('Select "',table_name,
'" as tablename,count(*) from ',table_name,' Union ') as Query from 
 INFORMATION_SCHEMA.TABLES 
WHERE table_schema = 'YourDatabaseName';
```
