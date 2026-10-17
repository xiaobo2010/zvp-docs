# 附件（`/api/attachments`）

> 来源：`routers/attachments_router.py`　·　挂载前缀：`/api/attachments`

!!! note
    该 router 在构造时挂了 router 级依赖 `dependencies=[Depends(reject_xuehai_device)]`，所有端点均受「学海设备禁止访问媒体库」门禁约束。

| 方法 | 路径 | 处理函数 | 鉴权 / 门禁 | 请求参数 / 请求体 | 用途 |
|---|---|---|---|---|---|
| `POST` | `/api/attachments` | `upload_attachment` | 学海设备门禁（router 级）；登录态、记日志；手动: 封禁检查、配额、entity 绑定需 manager | multipart form (`file,purpose,entity_type,entity_oid,original_name`) | 上传附件 |
| `GET` | `/api/attachments` | `list_attachments` | 学海设备门禁（router 级）；登录态、记日志；手动: `scope=all` 需 manager | query `scope,purpose,status,entity_type,entity_oid,page,size` | 附件列表 |
| `GET` | `/api/attachments/{oid}` | `get_attachment_bytes` | 学海设备门禁（router 级）；可选登录态 | query `exp,purpose,sig` | 读取附件字节；签名校验或读取权限 |
| `GET` | `/api/attachments/{oid}/thumb` | `get_attachment_thumb` | 学海设备门禁（router 级）；可选登录态 | query `exp,purpose,sig` | 读取缩略图；同字节鉴权 |
| `GET` | `/api/attachments/{oid}/meta` | `get_attachment_meta` | 学海设备门禁（router 级）；登录态、记日志 | path `oid` | 附件元数据；_can_read 门禁，sha256 仅 owner 可见 |
| `POST` | `/api/attachments/{oid}/link` | `create_attachment_link` | 学海设备门禁（router 级）；登录态、记日志 | query `ttl` | 签发外链；_require_owner_or_manager |
| `POST` | `/api/attachments/{oid}/ocr` | `request_attachment_ocr` | 学海设备门禁（router 级）；登录态、记日志 | path `oid` (BackgroundTasks) | 触发 OCR；_require_manager |
| `GET` | `/api/attachments/{oid}/ocr` | `get_attachment_ocr` | 学海设备门禁（router 级）；登录态、记日志 | path `oid` | 读取 OCR 结果；_require_owner_or_manager |
| `POST` | `/api/attachments/{oid}/verify` | `verify_attachment` | 学海设备门禁（router 级）；登录态、记日志 | body `VerifyRequest{decision,note}` | 查证附件；_require_manager |
| `POST` | `/api/attachments/{oid}/bind` | `bind_attachment` | 学海设备门禁（router 级）；登录态、记日志 | body `BindRequest{entity_type,entity_oid}` | 绑定附件实体；_require_manager |
| `DELETE` | `/api/attachments/{oid}` | `delete_attachment` | 学海设备门禁（router 级）；登录态、记日志 | path `oid` | 软删除附件；owner/manager，已查证仅 admin 可删 |

## 上传契约

`POST /api/attachments` 的请求体为 `multipart/form-data`：

| 字段 | 说明 |
|---|---|
| `file` | 单文件；同名多份时**只取最后一份** |
| `purpose` | 附件用途，默认 `volunteer_order`，须在允许集合内 |
| `entity_type` / `entity_oid` | 绑定的实体类型 / OID；需 manager |
| `original_name` | 原始文件名，缺省用上传文件名 |

**成功响应为 HTTP 200 + body `code: 201`**——路由未设 `status_code=`，HTTP 状态码仍是 `200`：

```json
{
  "status": "ok",
  "code": 201,
  "data": {
    "_id": "…",
    "kind": "original",
    "purpose": "volunteer_order",
    "entity_type": null,
    "entity_oid": null,
    "status": "stored"
  }
}
```

响应 `data` 的主键是 **`_id`**（不是 `id`）。同 owner 命中 sha256 去重时，不新建记录，直接返回既有记录（`code: 200`）。

## 限制与门禁

- **学海设备门禁**：请求头 `Xuehai-Id` 解析为 **>0 的整数**时，**整个 `/api/attachments`（读写）** 一律返回 `403 {"detail": "学海设备不支持访问媒体库"}`。空值 / 非正整数放行。
- **单文件上限 15 MB**：`Content-Length` 声明超限或实际读取字节超限，均返回 `413`。
- 上传需 manager 权限；`entity_type` / `entity_oid` 绑定同样需 manager。

## 易错点

判定「文件 part」必须用 **`starlette.datastructures.UploadFile`**（基类）。手工 `await request.form()` 取到的 file part **永远是 Starlette 基类实例**，而 `fastapi.UploadFile` 是它的**子类**、且只在声明式 `File(...)` 注入路径才被包装——若用 `fastapi.UploadFile` 判定会**恒为 False**，导致所有带文件的请求被误拒为 `400 {"detail": "Missing file part"}`。修复后统一按基类判定（可同时接受两者）。

## 已知限制

以下缺口需 manager 身份 + 手工构造畸形 multipart 才能触发，属输入校验不足而非越权：

- `original_name` 传成文件 part 时，`sanitize_filename` 迭代 `UploadFile` 触发未捕获 `TypeError` → `500`。
- `entity_type` / `entity_oid` 传成文件 part 时，真值为真，会把 `"UploadFile(filename=…)"` 当作实体 OID 写入数据库。
