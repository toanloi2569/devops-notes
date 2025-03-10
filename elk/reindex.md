# Hướng dẫn reindex một index trong Elasticsearch
Hướng dẫn sử dụng dev tool trong Kibana để reindex một index trong Elasticsearch

### Update template
1. Lấy index template hiện tại
```bash
# List all templates
GET _cat/templates 

# Get template
GET _template/<template_name>
```

2. Update template
   Thêm, sửa, xóa trường mapping

3. Apply template
```bash
PUT _template/<template_name>
<new config>
```

### Reindex
1. Reindex một index
```bash
POST _reindex
{
  "source": {
    "index": "index_name"
  },
  "dest": {
    "index": "index_name-v2"
  }
}
```

2. Count documents 
   Lệnh reindex sẽ có thời gian timeout. Sau khi timeout sẽ không nhận được kết quả từ task, nhưng task vẫn chạy ngầm.  
   Để biết khi nào task hoàn thành, cần kiểm tra số lượng document của index mới.

```bash
GET /index_name/_count?
GET /index_name-v2/_count?
```

3. Delete old index
```bash
DELETE /index_name
```

