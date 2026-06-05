# 南+签到

## 修改于： [cccp6/SouthPlusQianDao_WebDriver](https://github.com/cccp6/SouthPlusQianDao_WebDriver/)

**本项目使用的CloudFlare都是免费功能**

## 当前部署信息

| 资源 | 值 |
|------|-----|
| Worker URL | `https://southplus-qiandao.hhdheuwjbgg.workers.dev` |
| Worker 密码 | 见 GitHub Secret `PWD` |
| KV 命名空间 | `southplus-qiandao` |
| KV Namespace ID | `fdd7ecea3b94474d8fec9e9ab5b6616d` |
| KV 绑定名 | `QIANDAO_BINDING` |

## 签到状态面板

| 项目 | KV 值 (时间戳) | 上次签到时间 (北京时间) | 状态 |
|------|---------------|----------------------|------|
| 日常签到 (daily) | 异常 | 2026-06-06 01:56:06 | 失败 |
| 周常任务 (weekly) | 异常 | 2026-06-06 01:56:06 | 失败 |

> 读取 KV 的命令：
> ```bash
> export CLOUDFLARE_API_TOKEN="你的Cloudflare_API_Token"
> export CLOUDFLARE_ACCOUNT_ID="你的Account_ID"
> KV_ID="fdd7ecea3b94474d8fec9e9ab5b6616d"
> echo "daily:" && curl -s "https://api.cloudflare.com/client/v4/accounts/${CLOUDFLARE_ACCOUNT_ID}/storage/kv/namespaces/${KV_ID}/values/daily" -H "Authorization: Bearer ${CLOUDFLARE_API_TOKEN}"
> echo "weekly:" && curl -s "https://api.cloudflare.com/client/v4/accounts/${CLOUDFLARE_ACCOUNT_ID}/storage/kv/namespaces/${KV_ID}/values/weekly" -H "Authorization: Bearer ${CLOUDFLARE_API_TOKEN}"
> ```

## 实际部署流程记录

以下为本次实际执行的完整部署步骤。

### 1. 设置环境变量

```bash
export CLOUDFLARE_API_TOKEN="${CLOUDFLARE_API_TOKEN}"
export CLOUDFLARE_ACCOUNT_ID="8a63d8487d624f0be7fd03fd9536b9f9"
```

### 2. 创建 KV 命名空间

```bash
curl -s -X POST "https://api.cloudflare.com/client/v4/accounts/${CLOUDFLARE_ACCOUNT_ID}/storage/kv/namespaces" \
  -H "Authorization: Bearer ${CLOUDFLARE_API_TOKEN}" \
  -H "Content-Type: application/json" \
  --data '{"title": "southplus-qiandao"}'
```

返回结果：
```json
{"result":{"id":"fdd7ecea3b94474d8fec9e9ab5b6616d","title":"southplus-qiandao","supports_url_encoding":true},"errors":[],"messages":[],"success":true}
```

KV Namespace ID 为 `fdd7ecea3b94474d8fec9e9ab5b6616d`。

### 3. 初始化 KV 值

创建 `daily` 和 `weekly` 两个 key，值设为 `0`：

```bash
KV_ID="fdd7ecea3b94474d8fec9e9ab5b6616d"

curl -s -X PUT "https://api.cloudflare.com/client/v4/accounts/${CLOUDFLARE_ACCOUNT_ID}/storage/kv/namespaces/${KV_ID}/values/daily" \
  -H "Authorization: Bearer ${CLOUDFLARE_API_TOKEN}" --data '0'

curl -s -X PUT "https://api.cloudflare.com/client/v4/accounts/${CLOUDFLARE_ACCOUNT_ID}/storage/kv/namespaces/${KV_ID}/values/weekly" \
  -H "Authorization: Bearer ${CLOUDFLARE_API_TOKEN}" --data '0'
```

返回结果均为：
```json
{"result":null,"errors":[],"messages":[],"success":true}
```

### 4. 生成 Worker 密码

```bash
openssl rand -base64 18
```

生成密码：`你的密码`（请自行生成并填入 worker.js）

### 5. 修改 worker.js 中的密码

将 `worker.js` 第 9 行的密码占位符修改为生成的密码：

```javascript
// 修改前
if (pwd !== *** /* YourPassword */) {

// 修改后
if (pwd !== "你的密码") {
```

### 6. 创建 wrangler.toml 配置文件

在项目根目录创建 `wrangler.toml`：

```toml
name = "southplus-qiandao"
main = "worker.js"
compatibility_date = "2024-01-01"

[[kv_namespaces]]
binding = "QIANDAO_BINDING"
id = "fdd7ecea3b94474d8fec9e9ab5b6616d"
```

其中 `binding` 必须是 `QIANDAO_BINDING`，与 `worker.js` 中 `env.QIANDAO_BINDING` 对应。

### 7. 安装 wrangler 并部署 Worker

```bash
npm install -g wrangler

cd SouthPlusQianDao
export CLOUDFLARE_API_TOKEN="${CLOUDFLARE_API_TOKEN}"
export CLOUDFLARE_ACCOUNT_ID="8a63d8487d624f0be7fd03fd9536b9f9"
wrangler deploy
```

部署输出：
```
⛅️ wrangler 4.95.0
Total Upload: 5.00 KiB / gzip: 1.37 KiB
Your Worker has access to the following bindings:
Binding                Resource
env.QIANDAO_BINDING    KV Namespace
  fdd7ecea3b94474d8fec9e9ab5b6616d

Uploaded southplus-qiandao (1.17 sec)
Deployed southplus-qiandao triggers (1.07 sec)
  https://southplus-qiandao.hhdheuwjbgg.workers.dev
Current Version ID: 741d703a-f69d-4dfb-8c12-4324f324243e
```

Worker 部署成功，URL 为 `https://southplus-qiandao.hhdheuwjbgg.workers.dev`。

### 8. 更新 GitHub Actions 工作流 URL

修改 `.github/workflows/work.yaml` 第 30 行，将 Worker URL 替换为新部署的地址：

```powershell
# 修改前
$url = "https://south-plus.poker-sang.workers.dev/?pwd=$myPwd&cookie=$cookie&ua=$ua"

# 修改后
$url = "https://southplus-qiandao.hhdheuwjbgg.workers.dev/?pwd=$myPwd&cookie=$cookie&ua=$ua"
```

### 9. 验证部署

通过 API 验证 Worker 存在：

```bash
curl -s "https://api.cloudflare.com/client/v4/accounts/${CLOUDFLARE_ACCOUNT_ID}/workers/scripts/southplus-qiandao" \
  -H "Authorization: Bearer ${CLOUDFLARE_API_TOKEN}"
```

验证 KV 值已初始化：

```bash
curl -s "https://api.cloudflare.com/client/v4/accounts/${CLOUDFLARE_ACCOUNT_ID}/storage/kv/namespaces/${KV_ID}/values/daily" \
  -H "Authorization: Bearer ${CLOUDFLARE_API_TOKEN}"
# 返回: 0

curl -s "https://api.cloudflare.com/client/v4/accounts/${CLOUDFLARE_ACCOUNT_ID}/storage/kv/namespaces/${KV_ID}/values/weekly" \
  -H "Authorization: Bearer ${CLOUDFLARE_API_TOKEN}"
# 返回: 0
```

### 10. 需要用户手动完成的步骤

Fork 仓库后，在 GitHub 仓库的 **Settings > Secrets and variables > Actions > Repository secrets** 中添加：

| Secret 名称 | 值 | 说明 |
|-------------|-----|------|
| `PWD` | Worker 的访问密码 | 与 worker.js 中设置的一致 |
| `COOKIE` | 从浏览器获取的 Cookie | 只需 `eb9e6_winduser` 和 `eb9e6_cknum` 两个字段 |
| `UA` | 浏览器的 User-Agent | 必须和获取 Cookie 时的 UA 完全一致 |

---

## 请求方式说明

签到通过论坛任务系统实现，基础 URL 为 `https://www.south-plus.net/plugin.php`，拼接不同参数实现不同任务。

### 日常签到 (cid=15)

**领取任务：**
```
https://www.south-plus.net/plugin.php?H_name=tasks&action=ajax&actions=job&cid=15&nowtime=1672151938011&verify=f2807318
```

**完成任务：**
```
https://www.south-plus.net/plugin.php?H_name=tasks&action=ajax&actions=job2&cid=15&nowtime=1672152113906&verify=f2807318
```

### 新年红包 (cid=19)

**领取任务：**
```
https://www.south-plus.net/plugin.php?H_name=tasks&action=ajax&actions=job&cid=19&nowtime=1672570369699&verify=f2807318
```

**完成任务：**
```
https://www.south-plus.net/plugin.php?H_name=tasks&action=ajax&actions=job2&cid=19&nowtime=1672570470977&verify=f2807318
```

### 周常任务 (cid=14)

**领取任务：**
```
https://www.south-plus.net/plugin.php?H_name=tasks&action=ajax&actions=job&cid=14&nowtime=1673581486261&verify=42cb3e60
```

**完成任务：**
```
https://www.south-plus.net/plugin.php?H_name=tasks&action=ajax&actions=job2&cid=14&nowtime=1673581486561&verify=42cb3e60
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `H_name` | 固定值 `tasks`，表示任务系统 |
| `action` | 固定值 `ajax`，表示异步请求 |
| `actions` | `job` = 领取任务，`job2` = 完成任务 |
| `cid` | 任务 ID：`15` 日常签到，`19` 新年红包，`14` 周常任务 |
| `nowtime` | 时间戳（固定值即可） |
| `verify` | 验证码（固定值） |

---

- 将 API 获取改为了简单、粗暴、无脑的 Invoke-WebRequest
- 在CloudFlare创建KV，用来记录上次成功签到的时间，创建完后添加daily和weekly两项，值为0即可
- 在CloudFlare创建worker（计算），内容为[worker.js](worker.js)（文件中需要修改为自己指定的密码）
- 根据<https://developers.cloudflare.com/kv/get-started/>将KV绑定到CloudFlare，以便从worker.js中读写KV
- 并修改[work.yaml](.github/workflows/work.yaml)中使用的网址为worker的网址
- 在 **Settings > Secret and variables > Actions > Repository secrets** 中设置好 **COOKIE**（需要 Chrome 格式）、**PWD**（worker.js文件中自己指定的，防止被他人使用worker）和**UA**（User-Agent）。
- 运行 Action 即可

### 注意

- User-Agent必须和获取Cookie浏览器的User-Agent完全相同，否则会报错“您还没有登录或注册，暂时不能使用此功能!!”
- Action虽然每半小时运行一次，但worker.js会判断距离上次成功是否有18小时，有才会发送请求
- 第一次使用时，KV为0、且实际上已经签过到时，Action可能会一直报错；直到第一次成功后更新KV，才不会继续报错
- Cookie的获取
  ![图片](https://github.com/user-attachments/assets/2c010da4-0078-4426-8084-ffd7c7540876)
- 经过*前人*测试得出，cookies只需要eb9e6_winduser与eb9e6_cknum两个即可，其他可以删除。有效期是一年。
