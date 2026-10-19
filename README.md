# demoXXE

Lab thực hành lỗ hổng **XXE (XML External Entity)**. Ứng dụng web Java (Jakarta EE Servlet + MySQL) mô phỏng API quản lý tồn kho, cố tình cấu hình parser XML không an toàn để học về XXE.

> ⚠️ Code cố ý chứa lỗ hổng. Chỉ chạy trong lab/localhost, không deploy ra production.

## Công nghệ
Java 21 · Jakarta Servlet 6 · MySQL · đóng gói WAR (Maven) · chạy trên Tomcat 10+.

## Endpoint
Context path: `/demo`. Các request XML cần header `Content-Type: application/xml`.

| Method | Path | Profile parser | Chức năng |
|---|---|---|---|
| GET | `/demo/product` | — | Liệt kê sản phẩm |
| POST | `/demo/product/stock` | `DEFAULT` | Lấy số lượng tồn (dễ tấn công nhất) |
| PUT | `/demo/product/stock` | `OOB_PARAMETER_ONLY` | Cập nhật số lượng |
| DELETE | `/demo/product/stock` | `OOB_GENERAL` | Xoá tồn kho |

`XmlParseProfile` quyết định mức an toàn của parser. Mỗi method dùng một profile khác nhau để minh hoạ từng biến thể XXE. Profile `SECURE` là cấu hình an toàn (chặn DOCTYPE + tắt external entity).

## Cài & chạy
1. Tạo MySQL schema `xxe` với user `xxe` (xem `DBConnection.java`), gồm bảng `products`, `stocks`, `stores`.
2. Build:
   ```bash
   mvn clean package
   ```
3. Deploy `target/demo.war` vào Tomcat, truy cập `http://localhost:8080/demo/`.

## Thử XXE (chỉ trong lab)
Request hợp lệ:
```bash
curl -X POST http://localhost:8080/demo/product/stock \
  -H "Content-Type: application/xml" \
  --data '<stock><productId>1</productId><storeId>1</storeId></stock>'
```
Payload XXE đọc file (endpoint POST):
```xml
<?xml version="1.0"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/hosts"> ]>
<stock><productId>&xxe;</productId><storeId>1</storeId></stock>
```

## Cách vá
Đổi controller sang dùng `XmlParseProfile.SECURE` để tắt DOCTYPE và external entity (theo OWASP XXE Prevention).
