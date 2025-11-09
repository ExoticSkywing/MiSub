# MiSub 多 API 地理信息查询说明

## 🎉 新特性

已实现智能多 API 轮询策略，按优先级自动切换：

1. **ipgeolocation.io** → 最精准（1000次/天）
2. **ipwhois.io** → 完全免费（10,000次/月）
3. **ipdata.co** → 准确（1500次/天）
4. **ip-api.com** → 兜底（45次/分钟）
5. **Cloudflare** → 最后降级

---

## ✅ 已完成的优化

### 1. **添加国旗 emoji** 🇨🇳
所有 API 现在都会显示国旗 emoji：
```
*国家:* 🇨🇳 `China`
```

### 2. **显示数据来源**
每次通知都会显示使用的 API：
```
*数据来源:* `ipgeolocation.io`
```

### 3. **街道信息**（ipgeolocation.io 独有）
```
*国家:* 🇨🇳 `China`
*城市:* `Wuhan`
*街道:* `Hongshan`  ← 只有 ipgeolocation.io 提供
```

### 4. **修复 ASN 显示**
优化了 ASN 字段的读取，支持多种 API 返回格式。

---

## ❓ 关于中文显示的说明

### 为什么 ipgeolocation.io 不是中文？

**实际情况：**
- ipgeolocation.io 返回：`China`, `Wuhan`, `Hongshan`（英文）
- ip-api.com 返回：`中国`, `武汉`（中文）

**原因：**
- ipgeolocation.io 的**免费版不支持中文字段**
- 即使传递 `lang=zh-CN` 参数，返回的仍然是英文字段名

**各 API 的语言支持：**

| API | 国家 | 城市 | 街道 | 说明 |
|-----|------|------|------|------|
| **ipgeolocation.io** | 🇨🇳 China | Wuhan | Hongshan | 英文（免费版限制） |
| **ipwhois.io** | 🇨🇳 中国 | 武汉 | - | 中文 ✅ |
| **ipdata.co** | 🇨🇳 China | Wuhan | - | 英文 |
| **ip-api.com** | 🇨🇳 中国 | 武汉 | - | 中文 ✅ |
| **Cloudflare** | 🇨🇳 CN | Wuhan | - | 英文 |

---

## 💡 如何获得中文显示？

### 方案1：不配置 ipgeolocation.io API Key（推荐）
如果你更看重中文显示，可以：
1. **不填写** `ipgeolocation.io API Key`
2. 系统会自动跳过它，直接使用 **ipwhois.io**（支持中文）

**效果：**
```
*国家:* 🇨🇳 `中国`
*城市:* `武汉`
*ISP:* `Chinanet`
*ASN:* `AS4134`
*数据来源:* `ipwhois.io`
```

### 方案2：接受英文（推荐）
- **ipgeolocation.io 最精准**，还提供街道信息
- 英文字段也能看懂（China, Wuhan）
- 有国旗 emoji 🇨🇳 辅助识别

---

## 🎯 轮询策略详解

### 完整流程

```
请求到达
   ↓
【1】ipgeolocation.io（如果配置了 API Key）
   ├─ 成功 → 返回英文 + 街道信息 ✅
   └─ 失败/未配置 → 下一步
      ↓
【2】ipwhois.io（无需配置，自动使用）
   ├─ 成功 → 返回中文 ✅
   └─ 失败 → 下一步
      ↓
【3】ipdata.co（如果配置了 API Key）
   ├─ 成功 → 返回英文 ✅
   └─ 失败/未配置 → 下一步
      ↓
【4】ip-api.com（无需配置，自动使用）
   ├─ 成功 → 返回中文 ✅
   └─ 失败 → 下一步
      ↓
【5】Cloudflare（最后降级）
   └─ 返回英文（城市可能不准） ⚠️
```

---

## 📊 Telegram 通知示例

### 情况1：ipgeolocation.io 成功（英文 + 街道）
```
🛰️ *订阅被访问*

*IP 地址:* `xxx.xxx.xxx.xxx`
*国家:* 🇨🇳 `China`
*城市:* `Wuhan`
*街道:* `Hongshan`
*ISP:* `CHINANET BACKBONE No.31,Jin rong Street`
*ASN:* `AS4134`
*数据来源:* `ipgeolocation.io`

*域名:* `subhub.tsmoe.com`
*客户端:* `Loon/904 CFNetwork/1402.0.8 Darwin/22.2.0`
*请求格式:* `base64`
*订阅组:* `publicdaily`
*到期时间:* `2026/7/17 23:59:59`

*时间:* `2025/11/9 14:32:33 (UTC+8)`
```

### 情况2：ipwhois.io 成功（中文）
```
*国家:* 🇨🇳 `中国`
*城市:* `武汉`
*ISP:* `Chinanet`
*ASN:* `AS4134`
*数据来源:* `ipwhois.io`
```

### 情况3：Cloudflare 降级（城市可能不准）
```
*国家:* 🇨🇳 `CN`
*城市:* `Shanghai` ⚠️
*ISP:* `CHINANET-BACKBONE`
*ASN:* `AS4134`
*数据来源:* `Cloudflare (城市可能不准)`
```

---

## 🔧 配置建议

### 追求最高准确性 + 街道信息
```
✅ 配置 ipgeolocation.io API Key
✅ 配置 ipdata.co API Key
💬 接受英文显示
```

### 追求中文显示
```
❌ 不配置 ipgeolocation.io API Key
❌ 不配置 ipdata.co API Key
💬 系统会自动使用 ipwhois.io（中文）
```

### 平衡方案（推荐）
```
✅ 配置 ipgeolocation.io API Key
✅ 配置 ipdata.co API Key
💬 英文显示 + 国旗 emoji + 街道信息
💬 达到上限后自动切换到 ipwhois.io（中文）
```

---

## 🆕 国旗 emoji 实现

使用 Unicode 区域指示符号（Regional Indicator Symbols）：
- CN → 🇨🇳
- US → 🇺🇸
- JP → 🇯🇵

**代码实现：**
```javascript
function getCountryEmoji(countryCode) {
  if (!countryCode || countryCode.length !== 2) return '';
  const codePoints = countryCode
    .toUpperCase()
    .split('')
    .map(char => 127397 + char.charCodeAt());
  return String.fromCodePoint(...codePoints);
}
```

**支持的 API：**
- ✅ ipgeolocation.io（直接提供 country_emoji 字段）
- ✅ ipwhois.io（通过 country_code 生成）
- ✅ ipdata.co（优先使用 emoji_flag，否则生成）
- ✅ ip-api.com（通过 countryCode 生成）
- ✅ Cloudflare（通过 country 生成）

---

## 📝 更新日志

- **2025-11-09**：
  - ✅ 添加国旗 emoji 支持
  - ✅ 修复 ASN 显示（支持多种字段格式）
  - ✅ 优化街道信息显示（仅在有数据时显示）
  - ✅ 显示数据来源（方便调试）
  - ✅ 实现多 API 智能轮询

---

## ❓ 常见问题

### Q1: 为什么看到的是英文而不是中文？
**A:** 因为当前使用的是 ipgeolocation.io，它的免费版只返回英文。如果想要中文，请不要配置 ipgeolocation.io API Key，系统会自动使用 ipwhois.io（中文）。

### Q2: 如何知道使用了哪个 API？
**A:** 每次 Telegram 通知都会显示"*数据来源:* `ipgeolocation.io`"。

### Q3: ipgeolocation.io 的街道信息准确吗？
**A:** 非常准确！这是它相比其他 API 的最大优势。

### Q4: 可以同时显示英文和中文吗？
**A:** 不可以，因为每个 API 只返回一种语言。但你可以通过国旗 emoji 快速识别国家。

### Q5: 如果所有 API 都失败了会怎样？
**A:** 最后会降级到 Cloudflare 原生数据。虽然城市可能不准，但国家和 ASN 是准确的。

---

## 🎉 总结

**推荐配置：**
- ✅ 配置两个 API Key（ipgeolocation.io + ipdata.co）
- ✅ 享受最高准确性 + 街道信息
- ✅ 接受英文显示（有国旗 emoji 辅助）
- ✅ 达到上限后自动降级到中文 API

**如果一定要中文：**
- ❌ 不配置任何 API Key
- ✅ 自动使用 ipwhois.io（中文 + 免费）
- ⚠️ 没有街道信息

**最佳体验：**
- 🇨🇳 国旗 emoji 一眼识别国家
- 📍 街道信息精准定位
- 🔄 智能轮询确保可用性
- 💰 每天 5500+ 次免费额度
