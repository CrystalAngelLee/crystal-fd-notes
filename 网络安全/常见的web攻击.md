# 常见web攻击

## * XSS

> XSS (Cross-Site Scripting)，跨站脚本攻击，因为缩写和CSS重叠，所以只能叫XSS。

**定义**

跨站脚本攻击是指通过存在安全漏洞的Web⽹站注册⽤户的浏览器内运⾏⾮法的非本站点HTML标签或JavaScript进⾏的⼀种攻击。

**跨站脚本攻击有可能造成以下影响:**

- 利⽤虚假输⼊表单骗取⽤户个⼈信息。
- 利⽤脚本窃取⽤户的Cookie值，被害者在不知情的情况下，帮助攻击者发送恶意请求。
- 显示伪造的⽂章或图⽚。

### XSS攻击分类

#### 反射型- url参数直接注⼊

```js
//普通
http://localhost:3000/?from=china

// alert尝试
http://localhost:3000/?from=<script>alert(3)</script>

// 获取Cookie
http://localhost:3000/?from=<script src="http://localhost:4000/hack.js">
</script>

// 短域名伪造 https://dwz.cn/

// 伪造 cookie ⼊侵 chrome
document.cookie="kaikeba:sess=eyJ1c2VybmFtZSI6Imxhb3dhbmciLCJfZXhwaXJlIjox
NTUzNTY1MDAxODYxLCJfbWF4QWdlIjo4NjQwMDAwMH0="
```

#### 存储型-存储到DB后读取时注⼊

```js
// 评论
<script>alert(1)</script>
// 跨站脚本注⼊
我来了<script src="http://localhost:4000/hack.js"></script>
```

### XSS攻击的危害

> Scripting 能⼲啥就能⼲啥

- 获取⻚⾯数据
- 获取Cookies
- 劫持前端逻辑
- 发送请求
- 偷取⽹站的任意数据
- 偷取⽤户的资料
- 偷取⽤户的秘密和登录态
- 欺骗⽤户

### 防范⼿段

#### **1. ejs转义**

> ⼩知识
>
> ```js
> <% code %> ⽤于执⾏其中javascript代码；
> <%= code %> 会对code进⾏html转义；
> <%- code %> 将不会进⾏转义
> ```

#### **2. HEAD**

```js
ctx.set('X-XSS-Protection', 0) // 禁⽌XSS 过滤
// http://localhost:3000/?from=<script>alert(3)</script> 可以拦截但伪装⼀下就不⾏了
```

> 0禁⽌XSS过滤。
>
> 1启⽤XSS过滤（通常浏览器是默认的）。如果检测到跨站脚本攻击，浏览器将清除⻚⾯（删除不安全的部分）。
>
> 1;mode=block启⽤XSS过滤。如果检测到攻击，浏览器将不会清除⻚⾯，⽽是阻⽌⻚⾯加载。
>
> 1; report=(Chromium only)
>
> 启⽤XSS过滤。如果检测到跨站脚本攻击，浏览器将清除⻚⾯并使⽤CSP report-uri指令的功能发送违规报告。

#### **3. CSP**

> 内容安全策略(CSP, Content Security Policy)是⼀个附加的安全层，⽤于帮助检测和缓解某些类型的攻击，包括跨站脚本(XSS)和数据注⼊等攻击。这些攻击可⽤于实现从数据窃取到⽹站破坏或作为恶意软件分发版本等⽤途。
>
> CSP本质上就是建⽴⽩名单，开发者明确告诉浏览器哪些外部资源可以加载和执⾏。我们只需要配置规则，如何拦截是由浏览器⾃⼰实现的。我们可以通过这种⽅式来尽量减少XSS攻击。

```html
// 只允许加载本站资源
Content-Security-Policy: default-src 'self'

// 只允许加载HTTPS 协议图片
Content-Security-Policy: img-src https://*

// 不允许加载任何来源框架
Content-Security-Policy: child-src 'none'
```

```js
ctx.set('Content-Security-Policy', "default-src 'self' ")
```

#### 4. 转义字符

- ⿊名单转义

> ⽤户的输⼊永远不可信任的，最普遍的做法就是转义输⼊输出的内容，对于引号、尖括号、斜杠进⾏转义

```js
function escape(str) {
	str = str.replace(/&/g, "&amp;")
	...
	return str
}
```

- ⽩名单转义

富⽂本来说，显然不能通过上⾯的办法来转义所有字符，因为这样会把需要的格式也过滤掉。对于这种情况，通常采⽤⽩名单过滤的办法，当然也可以通过⿊名单过滤，但是考虑到需要过滤的标签和标签属性实在太多，更加推荐使⽤⽩名单的⽅式。

```js
const xss = require('xss')
const str = xss("")
```

#### 5. HttpOnly Cookie

> 这是预防XSS攻击窃取⽤户cookie最有效的防御⼿段。Web应⽤程序在设置cookie时，将其属性设为HttpOnly，就可以避免该⽹⻚的cookie被客户端恶意JavaScript窃取，保护⽤户cookie信息。

```js
httpOnly: true[koa-session  Config设置]
```

```js
// 原生设置
response.addHeader('Set-Cookie',"uid=112; Path=/;HttpOnly")
```

## * CSRF

> CSRF(Cross Site Request Forgery)，即跨站请求伪造，是⼀种常⻅的Web攻击，它利⽤⽤户已登录的身份，在⽤户毫不知情的情况下，以⽤户的名义完成⾮法操作。

- ⽤户已经登录了站点A，并在本地记录了cookie
- 在⽤户没有登出站点A的情况下（也就是cookie⽣效的情况下），访问了恶意攻击者提供的引诱危险站点B (B站点要求访问站点A)。
- 站点A没有做任何CSRF防御

### CSRF攻击危害

- 利⽤⽤户登录态
- ⽤户不知情
- 完成业务请求
- 盗取⽤户资⾦（转账，消费）
- 冒充⽤户发帖背锅
- 损害⽹站声誉

### 防御

#### 1. 禁⽌第三⽅⽹站带Cookie -有兼容性问题

#### 2. Referer Check - Https不发送referer

```js
app.use(async (ctx, next) => {
	await next()
	const referer = ctx.request.header.referer
	console.log('referer')
})
```

#### 3. 验证码「最有效的方法」

## * 点击劫持- clickjacking

点击劫持是⼀种视觉欺骗的攻击⼿段。攻击者将需要攻击的⽹站通过iframe嵌套的⽅式嵌⼊⾃⼰的⽹⻚中，并将iframe设置为透明，在⻚⾯中透出⼀个按钮诱导⽤户点击。

### 防御

#### 1. X-FRAME-OPTIONS

`X-FRAME-OPTIONS` 是⼀个 HTTP 响应头，在现代浏览器有⼀个很好的⽀持。这个 HTTP 响应头就是为了防御⽤ iframe 嵌套的点击劫持攻击。该响应头有三个值可选，分别是‘DENY，表示⻚⾯不允许通过 iframe 的⽅式展示’， ‘SAMEORIGIN，表示⻚⾯可以在相同域名下通过 iframe 的⽅式展示’，‘ALLOW-FROM，表示⻚⾯可以在指定来源的 iframe 中展示’

`ctx.set('X-FRAME-OPTIONS', 'DENY')`

#### 2. JS方式

```html
<head> <style id="click-jack">
html {
display: none !important;
 }
</style>
</head>
<body> <script>
if (self == top) {
  var style = document.getElementById('click-jack')
  document.body.removeChild(style)
   } else {
  top.location = self.location
 }
</script>
</body>
```

## * SQL注入

```js
// 填⼊特殊密码
1'or'1'='1
// 拼接后的SQL
SELECT *
FROM test.user
WHERE username = 'laowang'
AND password = '1'or'1'='1'
```

### 防御

- 所有的查询语句建议使⽤数据库提供的参数化查询接⼝**，参数化的语句使⽤参数⽽不是将⽤户输⼊变量嵌⼊到 SQL 语句中，即不要直接拼接 SQL 语句。例如 Node.js 中的 mysqljs 库的query ⽅法中的 ? 占位参数。
- 严格限制Web应⽤的数据库的操作权限**，给此⽤户提供仅仅能够满⾜其⼯作的最低权限，从⽽最⼤限度的减少注⼊攻击对数据库的危害
- 后端代码检查输⼊的数据是否符合预期**，严格限制变量的类型，例如使⽤正则表达式进⾏⼀些匹配处理。
- 对进⼊数据库的特殊字符（'，"，\，<，>，&，*，; 等）进⾏转义处理，或编码转换**。基本上所有的后端语⾔都有对字符串进⾏转义处理的⽅法，⽐如 lodash 的 lodash._escapehtmlchar库。

## OS注入

OS命令注⼊和SQL注⼊差不多，只不过SQL注⼊是针对数据库的，⽽OS命令注⼊是针对操作系统的。
OS命令注⼊攻击指通过Web应⽤，执⾏⾮法的操作系统命令达到攻击的⽬的。只要在能调⽤Shell函数的地⽅就有存在被攻击的⻛险。倘若调⽤Shell时存在疏漏，就可以执⾏插⼊的⾮法命令。

```js
// 以 Node.js 为例，假如在接⼝中需要从 github 下载⽤户指定的 repo
const exec = require('mz/child_process').exec;
let params = {/* ⽤户输⼊的参数 */};
exec(`git clone ${params.repo} /some/path`);

如果传⼊的参数是会怎样
https://github.com/xx/xx.git && rm -rf /* &&
```

## 请求劫持

### DNS劫持

顾名思义，DNS服务器(DNS解析各个步骤)被篡改，修改了域名解析的结果，使得访问到的不是预期的ip

### HTTP劫持

运营商劫持，此时⼤概只能升级HTTPS了

## DDOS

> http://www.ruanyifeng.com/blog/2018/06/ddos.html 阮⼀峰

**定义**

distributed denial of service【分布式拒绝访问攻击】

DDOS 不是⼀种攻击，⽽是⼀⼤类攻击的总称。它有⼏⼗种类型，新的攻击⽅法还在不断发明出来。
⽹站运⾏的各个环节，都可以是攻击⽬标。只要把⼀个环节攻破，使得整个流程跑不起来，就达到了瘫痪服务的⽬的。

其中，⽐较常⻅的⼀种攻击是 cc 攻击。它就是简单粗暴地送来⼤量正常的请求，超出服务器的最⼤承受量，导致宕机。我遭遇的就是 cc 攻击，最多的时候全世界⼤概20多个 IP 地址轮流发出请求，每个地址的请求量在每秒200次~300次。我看访问⽇志的时候，就觉得那些请求像洪⽔⼀样涌来，⼀眨眼就是⼀⼤堆，⼏分钟的时间，⽇志⽂件的体积就⼤了100MB。说实话，这只能算⼩攻击，但是我的个⼈⽹站没有任何防护，服务器还是跟其他⼈共享的，这种流量⼀来⽴刻就下线了。

### 常见攻击方式

#### FYN Flood

此攻击通过向目标发送具有欺骗性源IP地址的大量TCP“初始连接请求”SYN数据包来利用TCP握手。目标机器响应每个连接请求，然后等待握手中的最后一步，这一步从未发生过，耗尽了进程中的目标资源。

#### HTTP Flood

此攻击类似于同时在多个不同计算机上反复按Web浏览器中的刷新 - 大量HTTP请求泛滥服务器，导致拒绝服务

### 防御手段

#### 备份⽹站

备份⽹站不⼀定是全功能的，如果能做到全静态浏览，就能满⾜需求。最低限度应该可以显示公
告，告诉⽤户，⽹站出了问题，正在全⼒抢修

#### HTTP 请求的拦截 高防IP - 靠谱的多个运营商  Docker

硬件 服务器 防⽕墙

#### 带宽扩容 + CDN

提⾼犯罪成本

