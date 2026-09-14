# q-drug.com —— OAuth 应用所需的公开页面

**用途**：Google Auth Platform 的 Branding 页要求填 *Application home page / privacy policy /
terms of service*，且这些 URL 必须落在 **Authorized domains** 里注册过的域下。
q-drug.com 目前（2026-09-14）**无任何 DNS 记录**，先把这套静态页发布出去即可。

放在 `~/code/tmp/` 是草稿位；要长期用就 `git init` 成独立仓（建议 `downsea/q-drug-site`）。

## 文件

| 路径 | 对应 URL |
|---|---|
| `index.html` | `https://q-drug.com` |
| `privacy/index.html` | `https://q-drug.com/privacy/` |
| `terms/index.html` | `https://q-drug.com/terms/` |

页面是纯静态、零依赖、无外链；privacy 里明确写了「数据只在本地处理、不外传、不用于训练、
随时可在 myaccount.google.com/permissions 撤销」——这几条正是 Google 对受限 scope 的关注点。

## A 路：GitHub Pages（最省，推荐）

1. 建仓并推上去（`downsea` 账号已有 `repo`/`workflow` 之外的 `repo` 权限即可；Pages 用公开仓免费）：
   ```bash
   cd ~/code/tmp/q-drug-site && git init -b main && git add -A \
     && git commit -m "chore: q-drug.com public pages for OAuth branding" \
     && gh repo create downsea/q-drug-site --public --source=. --push
   ```
2. 仓库 Settings → Pages → Source = `main` / `/ (root)` → 等 1 分钟出 `https://downsea.github.io/q-drug-site/`。
3. Pages 页 Custom domain 填 `q-drug.com`（会写入 `CNAME` 文件）→ 勾 Enforce HTTPS。
4. **阿里云 DNS 控制台**（q-drug.com 的 NS 是 `dns7/dns8.hichina.com`）加记录：

   | 类型 | 主机记录 | 记录值 |
   |---|---|---|
   | A | `@` | `185.199.108.153` |
   | A | `@` | `185.199.109.153` |
   | A | `@` | `185.199.110.153` |
   | A | `@` | `185.199.111.153` |
   | CNAME | `www` | `downsea.github.io` |

   （可选 AAAA：`2606:50c0:8000::153` / `8001::153` / `8002::153` / `8003::153`）
5. 等解析生效后回到 Pages 页点 Check again → 证书签发（几分钟）。

## B 路：Fly.io（你 huaier.fun 已在用，同样免费）

```bash
cd ~/code/tmp/q-drug-site && fly launch --no-deploy --name q-drug-site --region hkg
fly deploy
fly certs add q-drug.com      # 按输出在阿里云加 A / AAAA 记录
```

## 顺带做掉（建议）

- **Search Console 验证域名**（<https://search.google.com/search-console> → 网域资源 → TXT 记录，
  在阿里云 DNS 加一条 `google-site-verification=…`）。Google 的 Branding 帮助里点名的就是这一步。
- 站点活了以后，Branding 表单就填：

  | 字段 | 值 |
  |---|---|
  | Application home page | `https://q-drug.com` |
  | Application privacy policy link | `https://q-drug.com/privacy/` |
  | Application terms of service link | `https://q-drug.com/terms/` |
  | Authorized domains | `q-drug.com`（只填 apex，不带 `www`、不带路径） |

## 顺序建议（别白干）

先只填三个字段 + Authorized domains，**直接点 Publish 试一次**：
Console 很可能只校验「域名字段非空 + 域名已注册」。若过了，就先把
`gog auth add … --force-consent` 重授权一次换长期 token，站点再从容上线；
若它报 URL 不可达/未注册，再走 A 路把站点立起来。
