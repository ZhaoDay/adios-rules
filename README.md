# Adios 规则 · iOS 去开屏广告

不越狱的 iPhone 去开屏方案：把应用的**开屏广告域名**在本地拦掉，广告请求发不出去，应用自然没有广告可播。

规则来自安卓版 [Adios](https://github.com/ZhaoDay/adios) 的适配经验——每个应用用的是哪家广告 SDK、走哪些域名，都已经在安卓端验证过。

> 这些规则**不能**替代安卓版 Adios：安卓在视图层把广告藏掉，连本地缓存的广告都不显示；iOS 只能在网络层拦，**已经缓存到本地的广告要等过期或卸载重装**。

---

## 已知支持

| 应用 | 规则文件 | 状态 |
|---|---|---|
| 波点音乐 | [`bodian.conf`](bodian.conf) | 已在 iPhone 实测生效 |

> 「生效」指：开启规则后冷启动不再出现开屏广告。首次使用如果之前缓存过广告，需要等缓存过期（约 12–24 小时）或卸载重装一次。

---

## 在 Shadowrocket 里使用

1. 打开「配置」页 → 右上角 **+** → 粘贴配置地址：
   ```
   https://raw.githubusercontent.com/ZhaoDay/adios-rules/main/bodian.conf
   ```
2. 下载完成后点选这份配置
3. 回到首页，**全局路由**选「配置」，打开顶部开关
4. 彻底关掉目标应用再打开，观察是否还有开屏广告

**更新规则**：配置列表里对这份配置执行「更新」，或者删掉重新下载。规则有变化时我会在这里更新。

这份配置只做了两件事：命中广告域名就 `REJECT`，其余流量 `DIRECT`。所以**不需要节点、不需要证书、不需要开 HTTPS 解密**，也不会有流量绕路。

---

## 在其他客户端里使用

### Quantumult X

`[filter_local]` 段追加：

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

### Loon

`[Rule]` 段追加：

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

### Surge

`[Rule]` 段追加：

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

注意：规则要放在 `FINAL` / 最后的 `DIRECT` **之前**，否则不会命中。

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
