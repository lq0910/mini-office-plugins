# Mini Office 官方插件仓库

这是 Mini Office 的官方插件源。客户端通过 GitHub API 读取根目录的 [`index.json`](index.json)，
拿到插件列表后一键安装。

不需要服务器、不需要审核后台。发布一个插件 = 提交一个目录 + 打一个 tag。

## 目录结构

```text
.
├── index.json                 # 客户端唯一入口（由脚本生成，不要手改）
├── plugins/
│   ├── theme-office-2003/
│   │   └── manifest.json
│   ├── theme-wps/
│   │   └── manifest.json
│   └── theme-office-365/
│       └── manifest.json
├── scripts/
│   └── build-index.mjs        # 打包全部插件 + 计算 SHA-256 + 生成 index.json
└── .github/workflows/
    └── release.yml            # 打 tag 后自动打包、发 Release、提交索引
```

`.moext` 产物与 `dist-plugins/` 不进版本库，由 CI 生成并挂到 Release。

## 当前插件

| 插件 | 类型 | 说明 |
|---|---|---|
| `mini-office.theme-office-2003` | 主题 | Windows XP / Office 2003 Luna 风格：米黄底、直角凸起控件、紧凑密度、经典菜单栏 |
| `mini-office.theme-wps` | 主题 | WPS Office 观感：冷灰分区、小圆角、默认密度 |
| `mini-office.theme-office-365` | 主题 | Microsoft 365 Fluent 观感：中性灰、扁平控件、舒适密度 |

## 本地开发

```bash
npm install
node scripts/build-index.mjs --owner lq0910 --repo mini-office-plugins --tag v1
```

脚本会：

1. 扫描 `plugins/*/manifest.json`；
2. 打包为 `dist-plugins/<id>-<version>.moext`；
3. 计算 SHA-256 与体积；
4. 写出 `index.json`。

打包固定了 zip 条目时间戳，**同一份源码重复打包得到完全相同的字节与哈希**。
这一点很重要：否则每次重新打包哈希都会变，索引里的校验值与 Release 资产对不上，
客户端安装时会报"插件包校验失败"。

## 发布

```bash
git tag v1
git push origin v1
```

`.github/workflows/release.yml` 会自动：

1. 打包全部插件；
2. 创建 Release 并上传 `.moext`；
3. 把新的 `index.json` 提交回 `main`。

客户端下次「刷新列表」即可看到。

## 新增一个主题插件

1. 新建 `plugins/<名字>/manifest.json`，参考现有主题；
2. 本地跑一次 `build-index.mjs`，确认能通过打包（清单字段缺失会直接报错）；
3. 在 Mini Office 里用「安装本地插件」选择生成的 `.moext`，先自测效果；
4. 提交并打 tag。

## 主题清单要点

- `permissions` 必须是 `[]`——声明式插件不执行代码。
- `tokens` 只能写 `--mo-*` 白名单内的令牌；写 `--mo-doc-*` 会被拒绝，
  因为它会改变文档画布的字体度量，进而改变分页与打印结果。
- 必填令牌：`--mo-bg`、`--mo-surface`、`--mo-surface-2`、`--mo-text-primary`、
  `--mo-text-muted`、`--mo-line`。
- 正文与背景对比度需 ≥ 4.5:1，次要文字 ≥ 3:1。
- `chrome` 只接受枚举值：

  | 键 | 取值 |
  |---|---|
  | `toolbar` | `modern` / `classic-two-row` / `ribbon` |
  | `controlShape` | `flat` / `beveled` / `rounded` |
  | `density` | `compact` / `default` / `comfortable` |
  | `titlebar` | `flat` / `gradient` |

  其中 `classic-two-row` 会让应用在标题栏下渲染一行经典菜单栏（文件 / 编辑 / 视图 / 工具 / 帮助）。

完整字段定义见 `mini-office` 仓库的 `packages/plugin-api/src/index.ts`，
发布流程详解见 `docs/plugin-publishing.md`。

## 校验

客户端在安装前会做这些检查，任何一项不通过都不会写入：

- 清单必填字段与 `engines` 版本范围；
- 声明式插件权限必须为空；
- 主题令牌白名单与对比度；
- 包内条目数与解压体积上限（防压缩炸弹）；
- 路径穿越（`..`、绝对路径、反斜杠）；
- 下载包的 SHA-256 与索引一致。

## 许可

各插件在其 `manifest.json` 的 `license` 字段声明。当前均为 MIT。
