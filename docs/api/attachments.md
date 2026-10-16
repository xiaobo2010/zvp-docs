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
