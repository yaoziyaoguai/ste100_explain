# ste100_explain

用 ASD-STE100 启发式文风讲清复杂主题，并根据问题制作图解、交互网页或讲解视频。适用于 Codex 和 Claude Code。

这个技能把“解释”作为一项需要完成的工作：阅读材料、建立例子、制作产物、检查结果。目标是让读者理解机制、预测变化，并能核对模型的结论。

## 可以做什么

| 需要理解的问题 | 可能的交付 |
| --- | --- |
| 概念、结论、操作方法 | 精炼文字与具体例子 |
| 结构、调用、状态和顺序 | 结构图、时序图或状态图 |
| 参数变化、因果机制、方案取舍 | 可交互 HTML、模拟器或比较工具 |
| 连续变化、教学与传播 | 自定义讲解视频，按要求加入旁白和字幕 |

形式由问题决定。简单问题直接回答；复杂问题可以生成较大的、使用一次即可丢弃的解释工具。指定了 HTML 或视频时，技能要求交付实际页面或成片，并核验能够检查的部分。

默认使用中文，面向具备一般工程背景、但未必了解当前领域的读者。可以直接指定其他语言、受众和解释深度。

## 使用

Codex 使用 `$ste100_explain`，Claude Code 使用 `/ste100_explain`。也可以在请求里明确说“使用 ste100_explain”。

```text
$ste100_explain 讲清这篇论文的核心假设、方法和结论边界。

$ste100_explain 用 80% STE 的风格解释这段设计。自然一点，保留关键术语。

$ste100_explain 用图解释这个服务收到请求后发生了什么。

$ste100_explain 用 HTML 解释 KV cache，让我切换有无缓存并逐步生成 token。

$ste100_explain 制作一个 90 秒的中文讲解视频，解释这段算法。
用逐步构建图形的方式建立直觉，并找能在本机运行的免费配音方案。

$ste100_explain 制作有旁白的讲解视频，使用我已授权的 ElevenLabs 凭据和预算。
```

在 Claude Code 中，将示例开头的 `$ste100_explain` 换成 `/ste100_explain`。

## STE 文风

默认约束包括固定术语、明确动作与执行者、区分说明与操作、前置必要条件、拆解长句，以及保留原有的数值、范围和不确定性。规则覆盖正文、图中说明、页面解释、字幕和旁白。

- **默认模式**：使用 STE 启发式约束，并迁移为适合中文的表达方式。
- **“80% STE”模式**：保持清楚和准确，放松机械的句长与句式要求。80% 表示文风偏好，不是合规分数。
- **严格英文 STE**：需要指定版本的完整规则、批准词典和技术词依据。没有完成这些检查时，只能交付明确标注的草稿。

中文不套用英文词数，也不宣称符合英文标准。所选规则及自拟改写示例见 [references/ste-style.md](references/ste-style.md)。本项目不附带完整标准或词典，也不宣称获得 ASD 认证。

## 安装到 Codex 与 Claude Code

推荐让两个宿主链接到同一个仓库，避免维护两份副本。以下命令适用于 macOS/Linux，会跳过所有已存在的目标，包括失效链接。跳过时先检查它是否指向正确仓库，不要直接覆盖。

```bash
git clone https://github.com/yaoziyaoguai/ste100_explain.git ~/work_space/ste100_explain
skill_repo="$HOME/work_space/ste100_explain"
for skill_root in "${CODEX_HOME:-$HOME/.codex}/skills" "$HOME/.claude/skills"; do
  skill_target="$skill_root/ste100_explain"
  if [ -e "$skill_target" ] || [ -L "$skill_target" ]; then
    printf '跳过现有路径：%s\n' "$skill_target"
    continue
  fi
  mkdir -p "$skill_root"
  ln -s "$skill_repo" "$skill_target"
done
```

如果仓库已存在，跳过 `git clone`。如果设置了自定义 `CODEX_HOME`，把 Codex 链接放到该目录下的 `skills/ste100_explain`。Windows 可按宿主的目录链接能力安装，或复制完整技能目录；复制安装后需要自行同步更新。

这是原 `explain` 技能的替代版本。迁移时先检查旧目录内容，安装并验证新名称后，再移除旧的 `explain` 安装。不要保留两个自动发现入口。

Codex 在下一轮读取更新后的技能列表。Claude Code 可重新打开会话，再使用 `/ste100_explain`。

技能名、目录名、仓库名均按作者要求使用 `ste100_explain`。部分通用技能校验器只接受连字符命名，会因此报命名错误；本仓库保留下划线，并单独核验宿主发现结果，见 [审计记录](docs/audit.md)。

## 更新与卸载

使用符号链接安装后，更新仓库即可同步到两端：

```bash
git -C ~/work_space/ste100_explain pull --ff-only
```

卸载前，先确认下面两个路径是指向本仓库的符号链接：

```bash
ls -ld ~/.codex/skills/ste100_explain ~/.claude/skills/ste100_explain
```

确认后删除链接；仓库仍会保留：

```bash
unlink ~/.codex/skills/ste100_explain
unlink ~/.claude/skills/ste100_explain
```

## 制作能力与依赖

文字解释不需要额外依赖。图解、页面和视频按实际任务使用宿主已有能力。

- Mermaid、SVG 或代码绘图可用于结构图和精确示意。
- `visualize`、`answer-me-with-html` 等技能可用于合适的页面交付；没有这些技能时仍可制作独立 HTML。
- `manim-video`、`video-use` 等技能可用于动画和后期。渲染器、字体、编码与配音能力必须在实际环境中确认。
- 旁白可以使用已授权的 ElevenLabs 等服务，也可以先评估现有本地 TTS。付费服务不是必需依赖。

独立 HTML 与宿主内嵌显示是两种交付。技能按 Codex 或 Claude Code 的实际能力选择展示方式，不把某一宿主的显示标记当成通用格式。

具体交付与验证要求见 [references/artifact-delivery.md](references/artifact-delivery.md)。

## 验证范围

[docs/audit.md](docs/audit.md) 对照最初的需求记录覆盖情况、修复和实际验证结果。[evals/cases.md](evals/cases.md) 提供可重复使用的行为检查请求。

文件格式、宿主发现和成品行为是不同层次的验证。检查通过不能保证模型每次严格遵循全部要求；图像、网页、视频和外部配音需要在具体制作任务中验证。
