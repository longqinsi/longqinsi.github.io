# [技术备忘录](../README.md) | [Spring](README.md) | Spring Data Rest
## 目录
  1. [各种HTTP方法操作成功的响应的HTTP状态码是多少？](#success-response-status-code-of-http-methods)
  1. [PUT,PUT和PATCH操作的区别是什么？](#difference-between-put-and-patch)
  
## 问题
### 1.各种HTTP方法操作成功的响应的HTTP状态码是多少？<a name="success-response-status-code-of-http-methods"></a>[↑](#top)
HTTP方法 | 状态码 | 描述
------- | ----- | ----
GET | 200 | OK
POST | 201 | Created
PUT | 204 | No Content
PATCH | 204 | No Content
DELETE | 204 | No Content

备注：
1. 如果id对应的记录不存在，GET/PATCH/DELETE方法会返回404(Not Found)，而PUT方法则会创建一条新的记录（这种情况下等同于POST）。

### 2.PUT和PATCH操作的区别是什么？<a name="difference-between-put-and-patch"></a>[↑](#top)
PUT是更新实体的所有字段，所以在请求报文问题中需要包含实体的所有字段；而PATCH只需要提供需要更新的字段
* PUT的请求报文体
   ```json
   {
      "name": "C.F. Martin & Company",
      "foundedDate": "1833-01-01T07:00:00.000+0000",
      "averageYearlySales": 9000000,
      "active": true
   }
   ```

* PATCH的请求报文体（只更新averageYearlySales字段）
   ```json
   {
      "averageYearlySales": 9000000
   }
   ```
### 3.如何更新外键关系？<a name="update-foreign-key-association"></a>[↑](#top)
下面的json发送到http://localhost:8080/api/models/1/modelType，HTTP方法为PUT,Content-Type为application/json，用于改变model的modelType
   ```json
   {
      "_links":{
         "modelType": {
            "href":"http://localhost:8080/api/modeltypes/3", 
            "rel":"modelType"
         } 
      }
   }
   ```
### 4.如何删除外键关系？<a name="delete-foreign-key-assaciation></a>[↑](#top)
发送请求到http://localhost:8080/api/models/1/modelType，HTTP方法为DELETE，请求体为空，即可删除对应model的modelType