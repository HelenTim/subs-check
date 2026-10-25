# gist 保存方法

## 部署

- 随意创建一个Gist（建议创建 secret Gist），浏览器地址栏最后一段即为 gist id

- 创建 GitHub token，需要 gist 的读写权限（classic token 勾选 `gist` scope 即可）

- 配置 gist id 和 token，两种方式二选一（**config.yaml 优先，未配置时读取环境变量**）：

  - 方式一：配置到 `config.yaml`

    ```yaml
    github-gist-id: "你的gist id"
    github-token: "你的github token"
    ```

    > 注意：config.yaml 会被提交到仓库时（例如本仓库的 Actions 场景），token 千万不要写在里面，请使用方式二。

  - 方式二：配置到环境变量（推荐）

    - 环境变量名：`GIST_ID` / `GIST_TOKEN`

    - GitHub Actions 场景：在仓库 `Settings` → `Secrets and variables` → `Actions` 中把这两个值配置为 Repository secrets，workflow 会自动透传给容器，token 不进仓库

    - 本地或 Docker 自行运行：给进程设置同名环境变量即可，效果相同

- 将 `save-method` 配置为 `gist`

- 两处都未配置时，gist 校验会失败并跳过本轮上传（本地 `output` 目录仍会正常保存文件）

## GitHub Actions 自动运行

项目自带 [.github/workflows/subs-check.yml](../.github/workflows/subs-check.yml)，默认每 3 小时定时运行一次，也可手动触发（workflow_dispatch）。使用前：

1. 在仓库 `Settings` → `Secrets and variables` → `Actions` 中配置 Repository secrets：

   | Secret | 说明 |
   | --- | --- |
   | `GIST_TOKEN` | GitHub token，上传 gist 用（必需） |
   | `GIST_ID` | gist id，上传 gist 用（必需） |
   | `MYNODES` | 可选，订阅链接或 gist 页面链接，作为额外节点来源 |

2. workflow 会用**仓库当前代码现场构建镜像**后运行（不拉取上游镜像），并通过 `-e` 把 `GIST_TOKEN` / `GIST_ID` 透传进容器；仓库中的 `config.yaml` 保持 `github-token: ""` 即可，无需任何注入步骤。

3. 运行完成后，最终筛选出的节点以 `all.yaml` / `mihomo.yaml` / `base64.txt` 三个文件更新到你的 gist。

## Worker 反代 GitHub API

- 将 [worker](./cloudflare/worker.js) 部署到 cloudflare workers

- `变量和机密` 设置`GITHUB_USER`为你的 github 用户名

- `变量和机密` 设置`GITHUB_ID`为你的 gist id

- `变量和机密` 设置`AUTH_TOKEN`为访问密钥

- 将 `github-api-mirror` 配置为你的 worker 地址

```
    github-api-mirror: "https://your-worker-url/github"
```

## 获取订阅

> 如果配置了Woker , 将 `key` 修改为对应的即可
> 订阅格式为 `https://your-worker-url/gist?key=all.yaml&token=AUTH_TOKEN`

- yaml格式的订阅

```
https://gist.githubusercontent.com/YOUR_GITHUB_USERNAME/YOUR_GIST_ID/raw/all.yaml
```

- base64编码的订阅

```
https://gist.githubusercontent.com/YOUR_GITHUB_USERNAME/YOUR_GIST_ID/raw/base64.txt
```

- 带规则的mihomo.yaml文件

```
https://gist.githubusercontent.com/YOUR_GITHUB_USERNAME/YOUR_GIST_ID/raw/mihomo.yaml
```
