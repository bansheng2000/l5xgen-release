# L5XGen 发布清单（公开仓库）

L5XGen 客户端从本仓库的 raw 地址联网读取两类清单。**这里只放公开内容。**

| 文件 | 用途 | 客户端读取地址 |
| --- | --- | --- |
| `crl.txt` | 吊销清单（作废的授权码指纹） | https://raw.githubusercontent.com/bansheng2000/l5xgen-release/main/crl.txt |
| `update.json` | 检查更新 | https://raw.githubusercontent.com/bansheng2000/l5xgen-release/main/update.json |

> raw 地址有 CDN 缓存（约 5 分钟），客户端请求时自带时间戳参数绕开。

## 为什么必须是公开仓库

私有仓库的 raw 地址需要 token，token 塞进客户端等于把凭据送出去。所以本仓库公开，
内容也必须经得起公开看：

- `crl.txt` **只放申请码指纹，不放客户名/单位名**；
- **签发私钥绝不能进本仓库**（私钥在本机 `C:\Users\12576\L5XGen-private\l5xgen-private.key`）。

## crl.txt

单行文本，可直接整行粘贴：

```
L5XCRL1.<base64url(正文)>.<base64url(RSA-2048/SHA-256 签名)>
```

正文（UTF-8、LF 换行，行序固定）：

```
L5XGen-crl-v1
seq=1
min-seq=0
revoke 3f9a1c22b7e04d15
```

- `seq` —— 清单版本号，**单调递增**。客户端只接受比本地缓存的更大的清单，防止用旧清单回退解锁。
- `min-seq` —— 低于此序号的授权码一律作废（0 = 不启用，正常只用 seq + revoke）。
- `revoke <16位hex>` —— 被吊销的 8 字节申请码指纹，可多行。

签发 / 校验（密钥与授权码同一对，客户端只内置公钥）：

```powershell
L5XGen.LicenseGen.exe crl --key <私钥> --seq 2 --min-seq 0 --revoke 3f9a1c22b7e04d15 --out crl.txt
L5XGen.LicenseGen.exe crl --verify crl.txt
```

**清单是全量不是增量**：每次新增吊销，都要把当前清单取回来、加一行、seq +1、重签，
旧的 revoke 行必须原样保留（否则被撤掉的人又活了）。

当前状态：`seq=1`、`min-seq=0`、无吊销项（签名有效的一次性占位清单）。

## update.json

```json
{"version": "1.0.0", "url": "https://github.com/bansheng2000/l5xgen-release/releases/download/v1.0.0/L5XGen-Setup-1.0.0.exe", "sha256": "CC3C576AE6CA3F902D10CCAC46EA3A8AC4C804E1838398BFDDEB645E0ECA635F", "notes": "首个版本", "mandatory": "false"}
```

客户端按字段名正则解析（`UpdateService`），`url` 支持 https 与 file://。

发版流程：

1. 把安装包作为 Release 资产上传（tag 如 `v1.1.0`）；
2. 改 `update.json` 的 `version` / `url` / `sha256` / `notes`，推上来。

注意：当前 `version` 与客户端内置版本同为 `1.0.0`，装了 1.0.0 的客户点「检查更新」只会得到
「已是最新」，**不会去下载**；上面那个 Release 资产还没上传，等真发 1.1.0 时再建。
