# Adios 规则 · iOS 去开屏广告

不越狱的 iPhone 去开屏方案：把应用的**开屏广告域名**在本地拦掉，广告请求发不出去，应用自然没有广告可播。

规则来自安卓版 [Adios](https://github.com/ZhaoDay/adios) 的适配经验——每个应用用的是哪家广告 SDK、走哪些域名，都已经在安卓端验证过。

> 这些规则**不能**替代安卓版 Adios：安卓在视图层把广告藏掉，连本地缓存的广告都不显示；iOS 只能在网络层拦，**已经缓存到本地的广告要等过期或卸载重装**。

---

## 怎么用（只做一次）

在 Shadowrocket 里添加**一份**配置，以后所有应用共用这一份：

```
https://raw.githubusercontent.com/ZhaoDay/adios-rules/main/adios.conf
```

1. 「配置」页 → 右上角 **+** → 粘贴上面的地址 → 下载
2. 点选这份配置
3. 首页「全局路由」选「配置」，打开顶部开关

**以后规则更新**：配置列表里对这份配置执行「更新」即可，不用重新添加。所有应用的规则都在 [`adios.conf`](adios.conf) 里，按应用分段，新应用会直接加进去。

这份配置只做两件事：命中广告域名就 `REJECT`，其余流量 `DIRECT`。所以**不需要节点、不需要证书、不需要开 HTTPS 解密**，也不会有流量绕路。

---

## 已支持

| 应用 | 状态 | 说明 |
|---|---|---|
| 波点音乐 | ✅ 已在 iPhone 实测生效 | TME 自有广告 + 腾讯优量汇 |

> 「生效」指：开启规则后冷启动不再出现开屏广告。首次使用如果之前缓存过广告，需要等缓存过期（约 12–24 小时）或卸载重装一次。

---

## 其他客户端的规则片段

不想用整份配置、只想往自己的配置里加几条的话，按客户端取用。规则要放在 `FINAL` / 最后的 `DIRECT` **之前**。

### 波点音乐

Quantumult X（`[filter_local]`）：

```
host, ad.tencentmusic.com, reject
host, adtest.tencentmusic.com, reject
host, tmead.y.qq.com, reject
host, tmeadcomm.y.qq.com, reject
host-suffix, gdt.qq.com, reject
host, sdk.e.qq.com, reject
host, sdkreport.e.qq.com, reject
host, tangram.e.qq.com, reject
host-suffix, gdtimg.com, reject
```

Loon（`[Rule]`）：

```
HOST,ad.tencentmusic.com,REJECT
HOST,adtest.tencentmusic.com,REJECT
HOST,tmead.y.qq.com,REJECT
HOST,tmeadcomm.y.qq.com,REJECT
HOST-SUFFIX,gdt.qq.com,REJECT
HOST,sdk.e.qq.com,REJECT
HOST,sdkreport.e.qq.com,REJECT
HOST,tangram.e.qq.com,REJECT
HOST-SUFFIX,gdtimg.com,REJECT
```

Surge（`[Rule]`）：

```
DOMAIN,ad.tencentmusic.com,REJECT
DOMAIN,adtest.tencentmusic.com,REJECT
DOMAIN,tmead.y.qq.com,REJECT
DOMAIN,tmeadcomm.y.qq.com,REJECT
DOMAIN-SUFFIX,gdt.qq.com,REJECT
DOMAIN,sdk.e.qq.com,REJECT
DOMAIN,sdkreport.e.qq.com,REJECT
DOMAIN,tangram.e.qq.com,REJECT
DOMAIN-SUFFIX,gdtimg.com,REJECT
```

---

## 原理与限制

### 为什么能拦

开屏广告素材是通过网络下载的。把广告 SDK 的接口域名拒掉，应用拿不到广告填充，只能显示自己的品牌启动页或直接进首页。

### 为什么有时候还看得到

- **本地缓存**：广告 SDK 会缓存素材，过期时间通常 12–24 小时。已有缓存不会被域名规则清掉——这是这套方案最大的短板。彻底清理：保持规则开启的状态下卸载重装应用。
- **HTTPDNS / IP 直连**：少数应用绕过系统 DNS 直接连 IP，按域名的规则可能漏掉。
- **全局生效**：小火箭 / Loon 这类工具没有「只对某个应用生效」的能力，规则开了对全部应用生效。目前拦截的都是纯广告域名，正常使用不受影响。

### 为什么不越狱也有代价

iOS 不允许第三方应用修改其他应用的行为，所以任何不越狱方案都必须有一个常驻的系统级过滤层（VPN 或 DNS 描述文件）。建议在小火箭里开启「按需连接 → 总是连接」，开机自动连上，就不用每次记着开。

---

## 免责声明

- 规则仅供个人学习与自用，不保证对所有版本、所有地区生效
- 应用更新后广告 SDK 可能更换域名，届时规则会失效，欢迎反馈
- 拦截基于域名判断，如果某条规则误伤了正常功能，请先关掉对应配置再反馈
