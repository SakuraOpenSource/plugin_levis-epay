# Levis 易支付插件

对接易支付 V1，为 Levis 提供支付宝和微信支付。插件只接受 `alipay` 与 `wxpay` 两种支付方式。

## 功能

- 创建支付宝/微信支付订单（易支付 `mapi.php`）
- 查询订单状态
- 易支付 V1 MD5 签名
- 支付通知地址由 Levis 主程序提供

## 配置

| 配置项 | 必填 | 说明 |
| --- | --- | --- |
| `pid` | 是 | 易支付商户 ID |
| `key` | 是 | 易支付商户密钥，敏感字段 |
| `gateway_url` | 是 | 易支付网关地址 |
| `payment_type` | 是 | 只能是 `alipay`（支付宝）或 `wxpay`（微信支付） |

配置通过 Levis 管理后台的插件配置页面填写，无需本地配置文件。未填写支付方式时插件默认使用支付宝；其他值会被拒绝并保持未配置状态。

## 编译与安装

### GitHub Actions 发布

打开仓库的 `Actions → Release EPay Plugin → Run workflow`，在 `tag` 中输入版本号，例如 `v0.1.0`。Workflow 会检出同级的 Levis 与 `LiusCraft/epay-sdk-go` 本地依赖，运行测试，构建插件 ZIP、生成 SHA-256 校验文件，并根据 `feat`、`fix`、`chore`、`docs` 等 commit message 自动生成 Release Notes。

发布产物：

```text
levis-epay-v0.1.0.zip
levis-epay-v0.1.0.sha256
```

该插件仓库不要求提交任何商户配置、密钥或本地依赖目录。

在仓库根目录执行：

```bash
./build_epay.sh
```

构建产物为 `epay.zip`。完整插件 ZIP 应包含：

```text
epay/
├── plugin
└── frontend/
    └── index.html
```

将 ZIP 上传到 Levis 管理后台的「插件管理」，然后配置 `pid`、`key`、网关和支付方式，授予 `wallet:credit` 与 `order:read` 权限，最后显式启用插件。

## 支付通知

在易支付商户后台将异步通知地址配置为：

```text
https://your-domain.com/api/plugin/v1/payment-notify/epay
```

Levis 负责校验通知签名、核对订单金额并通过插件回调完成幂等入账。重复通知不会重复增加余额。

## 签名算法

易支付 V1 MD5 签名规则：

1. 按参数名 ASCII 升序排列。
2. 排除 `sign`、`sign_type` 和空值。
3. 拼接为 `key=value&key=value`，参数值不 URL 编码。
4. 末尾追加商户密钥。
5. 计算小写 MD5。

插件提交 HTTP 表单时会按 HTTP 标准进行 URL 编码；编码只用于传输，不改变签名原文。

## 安全提示

- 不要在日志、源码或公开仓库中提交商户密钥。
- 生产环境使用 HTTPS。
- 保证 Levis 的公网回调地址可被易支付访问。
- 支付到账以前必须核对签名、订单号和金额。
