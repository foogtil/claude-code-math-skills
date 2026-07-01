# math-rigor & latex-guardian — Claude Code 论文严谨性双技能

> 两个为 Claude Code 设计的 skill，专门为数学推导与 LaTeX 论文写作提供**强制性的严谨性约束**。它们让 Claude 在帮你写论文时，不再"想当然"地跳步、不再随手改坏你的核心证明。

---

## 这是什么？

用 Claude Code 写数学论文时，常遇到两类风险：

1. **数学不严谨**——Claude 喜欢说"显然""易证"，跳过关键步骤，给出的等式/不等式有时根本不成立。
2. **结构被破坏**——Claude 在改一个公式时，可能顺手破坏了 `\begin/\end` 配对、弄丢 `\label`、改坏了你精心打磨的核心证明。

这两个 skill 分别对症下药，并且**互相联动**：

| Skill | 职责 |
|-------|------|
| **math-rigor**（数学严谨性审查） | 强制证明模板、编号步骤、逻辑漏洞自查、SymPy 数值验证、反例搜索 |
| **latex-guardian**（LaTeX 项目守护） | 受保护区块、修改后自动编译门禁、结构完整性检查、修改前意图确认 |

它们的关键词是 **"强制"**：规则写死在 skill 正文中，不可协商、不可降级，即使为了"简洁"或"效率"也不得绕过。

---

## math-rigor —— 数学严谨性审查

### 核心能力

- **强制读取 `math-context.md`**：每次推导前必须先加载项目根目录的符号表与假设定义；文件不存在则暂停并提示你创建，绝不"盲推"。
- **固定主张模板**：所有数学主张必须包含 **命题/引理/定理**、**前提**、**结论**、**证明概要** 四字段，缺一不可。
- **禁用含糊措辞**："显然""易证""trivially""clearly""by inspection" 等一律禁止；简单步骤也必须写出具体恒等变形或引用的引理编号。
- **编号步骤证明 + 逐步逻辑审查**：证明拆成编号小步，每步显式给出依据、推导、新增假设；逐项审查循环论证、不等式方向、隐含假设、边界情况、量词顺序。
- **即时 SymPy 数值验证**：对每个可计算步骤，自动编写并运行 Python/SymPy 代码——先符号化简，再多组随机参数（≥3 种规模、每组 ≥20 样本）数值验证，并测试零矩阵/奇异/边界等病态情形。
- **反例搜索模式**：当你说"猜想：…"或"是否成立？"时，自动进入暴力反例搜索（小规模穷举 + 大样本撒点），找到反例即给出具体数值。
- **`check all proofs` 命令**：一键审查整个 `.tex` 文件的所有 `proof` 环境，逐个重建模板、切分步骤、数值验证、逻辑审查。

### 触发示例

```
> 猜想：对任意实对称正定矩阵 A，有 tr(A)·tr(A⁻¹) ≥ n²
（math-rigor 自动启动反例搜索，对小维数矩阵撒点验证……）
```

---

## latex-guardian —— LaTeX 项目守护

### 核心能力

- **受保护区块机制**：用 `%%% PROTECTED BEGIN` / `%%% PROTECTED END` 包裹的代码，Claude **严禁修改内部任何字符**（含空格、换行）。被要求修改时，只能给出建议，由你手动解除保护后替换。
- **修改后自动编译门禁**：每次改 `.tex` 后立即后台运行 `latexmk -pdf` / `pdflatex`，扫描 `Error` 与 `Undefined control sequence`。**编译失败则阻止继续修改其他文件**，提取错误并要求立即修复。
- **编译报告**：通过后输出主文件是否更新、页数、Overfull/Underfull 警告数量与位置。
- **结构完整性检查**：自动校验
  - `\begin{X}` 与 `\end{X}` 是否一一对应；
  - `\label` 与 `\ref`/`\eqref`/`\pageref`/`\autoref` 是否成对；
  - `\cite` 的 key 是否在 `references.bib` 中存在（文件存在时）。
  - 任一不匹配立即警告并停止。
- **修改前意图说明**：改任何 `.tex` 内容前，必须先输出"改哪里、为什么、是否触及受保护区块、预期编译影响"，获你确认后再动手。

### 受保护区块示例

```latex
%%% PROTECTED BEGIN 定理3的主证明
\begin{proof}
... 你精心打磨的证明，Claude 碰不得 ...
\end{proof}
%%% PROTECTED END
```

---

## 双技能联动

两个 skill 设计为协同工作：

- **math-rigor** 启动时检测是否存在 `latex`/`tex` 守护类 skill，并对受保护 `proof` 环境的修改触发重新验证。
- **latex-guardian** 在涉及受保护 proof 内部修改时，提示联动 `math-rigor` 重验该证明。

效果：**核心证明被物理保护（latex-guardian），一旦解保护修改则被逻辑重验（math-rigor）**——结构与内容双保险。

---

## 安装

将两个 `.md` 文件放入项目的 `.claude/skills/` 目录（或用户级 `~/.claude/skills/`）：

```bash
# 项目级（仅当前项目生效）
mkdir -p .claude/skills
cp skills/math-rigor.md .claude/skills/
cp skills/latex-guardian.md .claude/skills/

# 用户级（所有项目生效）
mkdir -p ~/.claude/skills
cp skills/math-rigor.md ~/.claude/skills/
cp skills/latex-guardian.md ~/.claude/skills/
```

Claude Code 会自动加载 `.claude/skills/` 下的 skill，无需额外配置。

### 依赖

- **math-rigor**：Python 3 + SymPy（`pip install sympy`）
- **latex-guardian**：TeX 发行版（TeX Live / MiKTeX），需含 `latexmk` 或 `pdflatex`

### 推荐配套文件

在论文项目根目录创建 `math-context.md`（math-rigor 强制依赖）：

```markdown
# 符号表
- A: 实对称正定矩阵，n×n
- ‖·‖: 谱范数

# 全局假设
- 所有矩阵为实数域上方阵
- n 为正整数

# 记号约定
- tr(·): 迹
- 内积为标准 Euclidean 内积
```

---

## 使用建议

1. 把你最核心的 `proof`/`equation` 用 `%%% PROTECTED BEGIN/END` 包起来。
2. 维护好 `math-context.md`，符号定义越清晰，math-rigor 越能帮你抓漏洞。
3. 让 Claude 写新证明时，它会自动走"模板 → 编号步骤 → 数值验证"全流程。
4. 对存疑命题直接问"是否成立？"，让反例搜索先于证明。
5. 改完 `.tex` 后，latex-guardian 会自动编译 + 结构检查，不必手动跑。

---

## 局限与说明

- 这两个 skill 的约束属**指令性约束**，靠 Claude 遵守而非硬性技术拦截。若需真正"硬阻止"某些操作（如绝对禁止写入受保护区块），建议配合 Claude Code 的 **hooks**（PreToolUse hook 拦截 Write/Edit 并校验）使用。
- 数值验证通过**不等于**形式证明成立——math-rigor 会在"未找到反例"时明确声明这一点，绝不伪称已证明。
- SymPy/编译器不可用时，skill 会如实报告"未验证状态"，不会伪造结果。

---

## 许可证

MIT，详见 [LICENSE](./LICENSE)。

## 贡献

欢迎提 issue 与 PR。
