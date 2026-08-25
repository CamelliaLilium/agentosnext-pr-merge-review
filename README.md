# AgentOSNext PR Merge Review

[![Version](https://img.shields.io/badge/version-v2.0.0-blue)](https://github.com/CamelliaLilium/agentosnext-pr-merge-review/releases/tag/v2.0.0)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

一个面向 AgentOSNext 自托管 Forgejo PR 的 Codex Review Skill。

它让当前认证用户以与 Codex Reviewer / Claude Reviewer 并列的独立 Reviewer 身份，对 exact head 作出自己的判断。Skill 不读取、不汇总、不响应其他 Reviewer 的结论；先只读评审，再把 `APPROVE` 或 `REQUEST_CHANGES` 状态和完整理由展示给用户。只有用户明确确认 PR、40 位 head SHA、状态和正文后，才允许写入 Forgejo。

当前稳定版本：`v2.0.0`。

## 主要能力

- 固定 PR 的 exact base/head SHA，避免审错版本。
- 独立 Reviewer：不读取或采纳 Claude、Codex 或其他 Reviewer 的 Review。
- 区分 owning diff、机械合并和真实冲突解。
- 审查产品/架构文档、API/Proto、Go、Persistence/Migration、Web/BFF、部署、身份、安全与 SDK Driver。
- 要求 Bugfix 具备“修复前失败、修复后通过”的定向回归证据。
- Reviewer 只检查代码、架构、风险、测试变更和作者/CI 证据，不替 PR 作者执行测试。
- 最终结论只允许 `APPROVE` 或 `REQUEST_CHANGES`，并给出原因。
- 所有理由集中在一条 top-level formal Review 中，不发布 inline/comment-only 记录。
- Review 发布采用二阶段确认，不会把“帮我 Review”误当成 Forgejo 写权限。

## 一条 Prompt 完成安装和配置

复制下面整段 Prompt，粘贴给你自己的 Codex。该 Prompt 固定安装 `v2.0.0`，因此结果可复现。

```text
请安装并配置公开 Codex Skill：agentosnext-pr-merge-review。

来源（固定版本）：
https://github.com/CamelliaLilium/agentosnext-pr-merge-review/tree/v2.0.0/skills/agentosnext-pr-merge-review

请严格执行以下要求：

1. 使用系统自带的 $skill-installer。先完整读取它的 SKILL.md，再通过上述 GitHub 目录 URL 安装，不要只复制单个 SKILL.md；必须连同 agents/、references/ 和 LICENSE.txt 一起安装。
2. 安装目标应为 $CODEX_HOME/skills/agentosnext-pr-merge-review；未设置 CODEX_HOME 时使用 ~/.codex/skills/agentosnext-pr-merge-review。
3. 如果目标目录已存在，不要直接覆盖或删除。先核对现有版本/内容：若完全相同，只做验证；若不同，向我报告当前版本、目标版本和差异，并在覆盖前单独征得确认。
4. 安装完成后验证：
   - SKILL.md 的 name 必须是 agentosnext-pr-merge-review；
   - agents/openai.yaml 存在；
   - references/author-self-check-matrix.md、agentosnext-review-checks.md、review-publication-protocol.md 均存在；
   - LICENSE.txt 存在；
   - Skill 明确以当前用户的独立 Reviewer 身份工作，不读取其他 Review，只产生 APPROVE 或 REQUEST_CHANGES；
   - 如果本机有 skill-creator 的 quick_validate.py，使用 UTF-8 环境运行验证。
5. 只读检查运行前置条件并逐项报告，不要替我登录或索要/打印凭据：
   - git 可用；
   - tea CLI 可用；
   - tea 已连接到目标 Forgejo；
   - 本机存在有权读取的 AgentOSNext checkout，且其中 AGENTS.md、CLAUDE.md、docs/README.md 可读。
   缺少前置条件时给出最小配置建议，但不要创建测试评论、Review、Approve、Request Changes、分支、push 或 merge。
6. 本轮只负责安装、配置和验证，不要实际评审任何 PR，也不要向 Forgejo 写入任何内容。
7. 最后报告：安装路径、安装版本、文件完整性、验证结果、前置条件状态，以及我是否需要重启 Codex。安装或更新后提醒我重启 Codex，并说明该 Skill 从下一轮对话开始可用。

如果 $skill-installer 无法自动调用，可以使用其官方 install-skill-from-github.py helper，参数必须等价于：
--repo CamelliaLilium/agentosnext-pr-merge-review
--path skills/agentosnext-pr-merge-review
--ref v2.0.0
仍然必须遵守“不覆盖已有目录”和“不执行 Forgejo 写操作”的要求。
```

## 手动安装

如果不使用上面的 Prompt，可以让 Codex 调用 `$skill-installer`：

```text
$skill-installer install https://github.com/CamelliaLilium/agentosnext-pr-merge-review/tree/v2.0.0/skills/agentosnext-pr-merge-review
```

或直接运行本机 Codex 自带的 installer helper：

```bash
python ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo CamelliaLilium/agentosnext-pr-merge-review \
  --path skills/agentosnext-pr-merge-review \
  --ref v2.0.0
```

安装或更新后重启 Codex，使新的 Skill metadata 生效。

## 使用示例

只读评审：

```text
使用 $agentosnext-pr-merge-review 只读评审
https://forgejo.example.com/owner/repo/pulls/123，判断当前 exact head 是否可以合并。
不要读取其他 Reviewer 的 Review，也不要向 Forgejo 写入；先给我 APPROVE 或 REQUEST_CHANGES、完整原因和待发布提案。
```

对于 AgentOSNext：

```text
使用 $agentosnext-pr-merge-review 评审 AgentOSNext PR #1290。
你是与 Claude Reviewer、Codex Reviewer 并列的独立 Reviewer，不读取或采纳他们的 Review。
先完成只读 Review，只给出 APPROVE 或 REQUEST_CHANGES 及完整 top-level 正文；等待我确认后再发布。
```

确认发布必须包含完整 SHA，例如：

```text
确认将以上 top-level Review 正文以 REQUEST_CHANGES 发布到
agentos/AgentOSNext#1290，head cf5d4e8ffb6accf057a6b4f3b25e91d46f5418a8。
```

`继续`、`当前 head`、`帮我处理一下` 都不构成发布授权。

## 发布边界

该 Skill 默认只读。用户确认后只可以发布一条正式 top-level Review：

- `APPROVE`；
- `REQUEST_CHANGES`。

Review 正文必须包含作出该状态的原因；不创建 inline comment 或 comment-only 记录。

以下操作始终不包含在 Review 发布授权中：

- `git push`；
- 创建或删除分支；
- merge；
- 关闭/重开 PR；
- 修改 PR 标题、正文、标签或其他元数据。

PR 已合并或关闭时不发布任何 Review 状态。

## 前置条件

- Codex 已提供 `$skill-installer`；
- 能访问目标 Forgejo 和目标仓库；
- `tea` 已由使用者自行完成认证；
- 本地存在目标仓库 checkout，供只读 diff、对象和文档权威检查使用。

本仓库不包含任何 Forgejo/GitHub token、密码、Cookie、私钥或用户凭据。安装 Skill 不会自动获得目标仓库访问权。

## 仓库结构

```text
.
├── README.md
├── CHANGELOG.md
├── LICENSE
└── skills/
    └── agentosnext-pr-merge-review/
        ├── SKILL.md
        ├── LICENSE.txt
        ├── agents/
        │   └── openai.yaml
        └── references/
            ├── author-self-check-matrix.md
            ├── agentosnext-review-checks.md
            └── review-publication-protocol.md
```

真正安装到 Codex 的目录是：

```text
skills/agentosnext-pr-merge-review/
```

## 版本策略

本项目使用 [Semantic Versioning](https://semver.org/)：

- `MAJOR`：Review 决策、授权边界或发布协议发生不兼容变化；
- `MINOR`：新增评审面、Forgejo 能力或兼容功能；
- `PATCH`：澄清规则、修复误触发或文档问题。

稳定安装建议使用 tag URL。需要跟踪最新开发版本时才使用 `main`。

版本变化见 [CHANGELOG.md](CHANGELOG.md)。

## 安全说明

该 Skill 能在用户明确确认后，以本机已认证的 Forgejo 身份提交 Review。使用前请阅读并理解确认卡片；不要在共享环境中复用不属于你的认证会话。

发现安全问题时，请通过 GitHub 仓库的私密安全报告渠道提交，不要在公开 Issue 中粘贴凭据或内部代码。

## License

[MIT](LICENSE)
