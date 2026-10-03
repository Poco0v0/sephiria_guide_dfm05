# 赛菲莉娅 Sephiria 攻略站

B站 DFM05 独立制作的《赛菲莉娅（Sephiria）》游戏攻略资料站。

站内内容包括但不限于：

- 不同版本武器排行榜（含分期详细分析）
- 武器解析（六大武器全改造一图流）
- 流派解析（流派词条图解、武器所属流派索引、单独武器/流派解析）
- 游戏基础/机制解析
- 预设合集（六大武器可复制预设码）
- 网站更新公告

## 运行

推荐直接使用 uv：

```powershell
uv run streamlit run sephiriadfm05.py
```

如果已经激活虚拟环境，也可以运行：

```powershell
streamlit run sephiriadfm05.py
```

## Docker 镜像与 GHCR

仓库内的 `.github/workflows/docker.yml` 会先构建 AMD64 镜像、检查容器健康接口和应用入口，检查通过后再发布 AMD64 / ARM64 多架构镜像到 GHCR。

工作流仅支持手动触发：在 GitHub 的 **Actions → Build and publish Docker image → Run workflow** 中运行。
根据所选分支或标签生成镜像标签；选择默认分支时会发布 `latest`、分支名和 `sha-<提交短哈希>` 标签，其他分支或标签不会更新 `latest`。

镜像名称自动使用 `ghcr.io/<仓库所有者的小写名称>/<仓库名的小写名称>`，fork 后无需修改工作流中的账号。
发布使用 GitHub 自动提供的 `GITHUB_TOKEN`，工作流已声明 `packages: write` 权限，无需另外创建 PAT 或配置发布密码。
如果组织策略限制包发布，需要由仓库管理员允许该工作流写入 Packages。

### 拉取并运行

以下示例对应当前 fork；如果再次 fork，请替换镜像地址：

```bash
docker pull ghcr.io/poco0v0/sephiria_guide_dfm05:latest
docker run -d \
  --name sephiria-guide \
  --restart unless-stopped \
  -p 8501:8501 \
  ghcr.io/poco0v0/sephiria_guide_dfm05:latest
```

浏览器访问 `http://服务器IP:8501`。镜像自带 Python、锁文件指定的依赖和攻略素材，服务器只需安装 Docker。
如果通过 Nginx 等反向代理访问，需要支持 WebSocket 转发。

首次发布的 GHCR 包默认可能为私有。若希望匿名拉取，在 GitHub 的 **Packages → 镜像包 → Package settings → Change visibility** 中设为 Public；保留私有时，拉取机器需要先执行 `docker login ghcr.io`，使用具有 `read:packages` 权限的 PAT 登录。

### 本地构建

在仓库根目录运行：

```bash
docker build -t sephiria-guide:local .
docker run --rm -p 8501:8501 sephiria-guide:local
```

Dockerfile 使用 Python 3.12 和 `uv sync --locked --no-dev --no-install-project` 安装依赖。
`.dockerignore` 排除 Git 历史、本地虚拟环境、缓存和密钥配置；攻略图片和静态页面会保留在镜像中。
容器以普通用户运行，并保留 `static/` 的写入权限，供 PDF/Word 攻略页生成下载文件。

## 同步攻略图片

站内图片素材由 `asset_manifest.json` 记录源文件路径和站内目标路径。

检查源图是否有更新：

```powershell
uv run python sync_assets.py --dry-run
```

同步所有已更新的源图：

```powershell
uv run python sync_assets.py
```

只检查或同步某个条目：

```powershell
uv run python sync_assets.py --only 物理剑盾
```
