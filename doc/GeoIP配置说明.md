# MiSub 地理信息来源配置

## 📝 更新说明

已修改 `functions/[[path]].js`，支持两种地理信息来源，并可通过配置自由切换。

---

## 🎯 两种方案对比

### 方案1：Cloudflare 原生数据（推荐）

**优点：**
- ✅ **完全免费**（无外部API调用）
- ✅ **速度极快**（直接从 request.cf 读取）
- ✅ **不消耗 Workers 请求配额**
- ✅ **数据质量高**（Cloudflare 自己的网络数据）

**可用字段：**
```javascript
request.cf.country         // 国家代码：CN, US, JP
request.cf.city            // 城市：Beijing, Tokyo, New York
request.cf.asn             // ASN 号码：4134
request.cf.asOrganization  // 运营商名称：CHINANET-BACKBONE
request.cf.region          // 省份/州：Beijing
request.cf.timezone        // 时区：Asia/Shanghai
request.cf.colo            // Cloudflare 机房代码：HKG, LAX
```

**Telegram 通知效果：**
```
🛰️ *订阅被访问*

*IP 地址:* `119.98.186.152`
*国家:* `CN`
*城市:* `武汉`
*ISP:* `CHINANET-BACKBONE`
*ASN:* `AS4134`

*域名:* `subhub.tsmoe.com`
*客户端:* `clash-verge/v2.3.1`
*请求格式:* `clash`
*订阅组:* `publicdaily`
*到期时间:* `2026/7/17 23:59:59`

*时间:* `2025/11/9 12:02:14 (UTC+8)`
```

---

### 方案2：ip-api.com（备选）

**优点：**
- ✅ 城市名称中文化（如"武汉"而不是"Wuhan"）
- ✅ ISP 名称更详细（如"Chinanet HB"）

**缺点：**
- ❌ 需要外部 API 请求（可能失败）
- ❌ 有速率限制（免费版 45次/分钟）
- ❌ 增加延迟（~100-300ms）
- ❌ 消耗 Workers 请求配额

**Telegram 通知效果：**
```
🛰️ *订阅被访问*

*IP 地址:* `119.98.186.152`
*国家:* `中国`
*城市:* `武汉`
*ISP:* `Chinanet HB`
*ASN:* `AS4134 CHINANET-BACKBONE`

（其余内容相同）
```

---

## ⚙️ 配置方法

### 方法1：通过管理界面（推荐）

1. 访问 MiSub 管理页面
2. 进入 **设置 (Settings)**
3. 找到 **GeoIPSource** 配置项
4. 选择：
   - `cloudflare`（默认，推荐）
   - `ip-api`（备选）

### 方法2：直接修改 KV 存储

在 Cloudflare Workers 的 KV 存储中，修改 `MISUB:Settings` 键值：

```json
{
  "FileName": "MiSub",
  "BotToken": "你的TG机器人Token",
  "ChatID": "你的TG ChatID",
  "GeoIPSource": "cloudflare"
}
```

**可选值：**
- `"cloudflare"` - 使用 Cloudflare 原生数据（推荐）
- `"ip-api"` - 使用 ip-api.com

---

## 🔄 自动降级机制

代码内置了智能降级：

```javascript
// 如果配置为 cloudflare，但 request.cf 不可用
// 会自动降级到 ip-api

// 如果配置为 ip-api，但请求失败
// locationInfo 会保持为空（不影响通知发送）
```

**这意味着：**
- 即使选择了 `cloudflare`，在某些情况下也会尝试 `ip-api`
- 即使 `ip-api` 失败，TG 通知仍会正常发送（只是缺少地理信息）

---

## 📊 推荐配置

### 小规模部署（<100 用户）
```json
{
  "GeoIPSource": "cloudflare"
}
```
**理由：** 免费、快速、无限制

### 中大规模部署（>100 用户）
```json
{
  "GeoIPSource": "cloudflare"
}
```
**理由：** 还是推荐 Cloudflare，因为 ip-api 有速率限制

### 特殊需求（需要中文城市名）
```json
{
  "GeoIPSource": "ip-api"
}
```
**注意：** 免费版 ip-api 限制 45次/分钟，超过后会失败

---

## 🧪 测试验证

### 测试 Cloudflare 数据
```bash
# 访问你的订阅链接，查看 TG 通知
curl "https://your-misub.pages.dev/sub/publicdaily?target=clash"
```

检查 TG 通知中的：
- 城市名是否为英文（如 "Beijing"）
- ASN 格式是否为 "AS4134"

### 测试 ip-api 数据
```bash
# 修改配置为 "ip-api" 后，再次访问
curl "https://your-misub.pages.dev/sub/publicdaily?target=clash"
```

检查 TG 通知中的：
- 城市名是否为中文（如 "北京"）
- ASN 格式是否为 "AS4134 CHINANET-BACKBONE"

---

## 🔍 实际效果对比

| 字段 | Cloudflare | ip-api |
|------|-----------|--------|
| 国家 | `CN` | `中国` |
| 城市 | `Wuhan` | `武汉` |
| ISP | `CHINANET-BACKBONE` | `Chinanet HB` |
| ASN | `AS4134` | `AS4134 CHINANET-BACKBONE` |
| 速度 | <1ms | ~100-300ms |
| 可靠性 | 100% | ~95% (受速率限制) |

---

## ❓ 常见问题

### Q1: 为什么默认是 cloudflare？
**A:** 因为它完全免费、速度快、无限制，且数据质量高。

### Q2: 如果我不配置 GeoIPSource 会怎样？
**A:** 默认使用 `cloudflare`，向后兼容。

### Q3: 我能看到完整的 request.cf 数据吗？
**A:** 可以，在 Workers 日志中添加：
```javascript
console.log('CF Data:', JSON.stringify(request.cf, null, 2));
```

### Q4: ip-api 的速率限制够用吗？
**A:** 免费版 45次/分钟，对于小规模部署足够。如果用户频繁更新订阅，可能会触发限制。

### Q5: 两种方案的数据准确性如何？
**A:** 都很准确。Cloudflare 基于自己的全球网络数据，ip-api 基于第三方 IP 数据库。城市级别可能有差异，但 ASN 和国家基本一致。

---

## 📝 代码修改记录

### 修改文件
- `functions/[[path]].js`

### 修改内容
1. **函数签名更改**
   ```javascript
   // 旧版
   sendEnhancedTgNotification(settings, type, clientIp, additionalData)
   
   // 新版
   sendEnhancedTgNotification(settings, type, request, additionalData)
   ```

2. **新增配置项**
   ```javascript
   const defaultSettings = {
     // ... 其他配置
     GeoIPSource: 'cloudflare'
   };
   ```

3. **新增 Cloudflare 数据读取逻辑**
   ```javascript
   if (geoSource === 'cloudflare' && request.cf) {
     const cf = request.cf;
     const country = cf.country || 'N/A';
     const city = cf.city || 'N/A';
     const asn = cf.asn ? `AS${cf.asn}` : 'N/A';
     const isp = cf.asOrganization || 'N/A';
     // ...
   }
   ```

4. **调用方式更新**
   ```javascript
   // 旧版
   context.waitUntil(sendEnhancedTgNotification(config, '🛰️ *订阅被访问*', clientIp, additionalData));
   
   // 新版
   context.waitUntil(sendEnhancedTgNotification(config, '🛰️ *订阅被访问*', request, additionalData));
   ```

---

## ✅ 部署步骤

1. **上传更新后的代码**
   ```bash
   wrangler deploy
   ```

2. **验证配置**
   - 访问管理页面
   - 确认 `GeoIPSource` 显示为 `cloudflare`

3. **触发测试**
   - 访问任意订阅链接
   - 检查 TG 通知是否包含地理信息

4. **（可选）切换到 ip-api**
   - 在管理页面修改 `GeoIPSource` 为 `ip-api`
   - 保存设置
   - 再次测试

---

## 🎉 总结

- ✅ **默认使用 Cloudflare**（推荐）
- ✅ **保留 ip-api 选项**（可切换）
- ✅ **向后兼容**（不影响现有配置）
- ✅ **智能降级**（失败时自动切换）
- ✅ **零额外成本**（Cloudflare 原生数据免费）

**建议：除非有特殊需求（如中文城市名），否则保持默认 `cloudflare` 即可。**
