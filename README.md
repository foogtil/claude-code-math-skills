# math-rigor & latex-guardian — Rigorous-Math Skills for Claude Code

> Two Claude Code skills that enforce **strict rigor** in mathematical derivation and LaTeX paper writing. They stop Claude from hand-waving through proofs with "obviously" / "trivially", and stop it from silently breaking your core proofs, equations, or document structure.

---

## What is this?

When using Claude Code to write math-heavy papers, two risks show up again and again:

1. **Sloppy math** — Claude loves to say "obviously", "trivially", "it is easy to see", skipping critical steps, and sometimes asserting equalities or inequalities that simply do not hold.
2. **Broken structure** — while tweaking one formula, Claude can casually unbalance `\begin/\end` pairs, drop a `\label`, or damage a carefully crafted proof.

These two skills address each risk head-on, and are designed to **interlock**:

| Skill | Role |
|-------|------|
| **math-rigor** (Math Rigor Audit) | Enforce proof templates, numbered steps, per-step logic checks, SymPy numerical verification, counterexample search |
| **latex-guardian** (LaTeX Project Guardian) | Protected blocks, post-edit compile gate, structural-integrity checks, pre-edit intent confirmation |

The key word is **"enforce"**: the rules are written into the skill body and are non-negotiable — they cannot be relaxed for "brevity" or "efficiency".

---

## math-rigor — Math Rigor Audit

### Core capabilities

- **Mandatory `math-context.md` read** — before any derivation, the skill must load the symbol table and assumptions from `math-context.md` in the project root. If the file is missing, it pauses and asks you to create it. No "blind" derivation.
- **Fixed claim template** — every claim must include all four fields: **Proposition/Lemma/Theorem**, **Hypotheses**, **Conclusion**, **Proof Sketch**. None may be omitted.
- **Banned vague wording** — "obviously", "trivially", "clearly", "it is easy to see", "by inspection", and their Chinese equivalents ("显然", "易证") are forbidden. Even a simple step must show the concrete identity used or cite a specific lemma number.
- **Numbered-step proofs + per-step logic audit** — proofs are split into numbered steps, each with explicit basis, derivation, and new assumptions. Each step is audited for circular reasoning, inequality direction, hidden assumptions, edge cases, and quantifier order.
- **Instant SymPy numerical verification** — for every computable step, Claude writes and runs Python/SymPy code: symbolic simplification first, then random numerical sampling (≥3 sizes, ≥20 samples each), plus pathological cases (zero/singular matrices, n=1, complex values if allowed).
- **Counterexample search mode** — when you say "Conjecture: …" or "Is this true?", the skill automatically enters brute-force counterexample search (small-scale exhaustive + large-sample random). If a counterexample is found, it reports the concrete values.
- **`check all proofs` command** — one-shot audit of every `proof` environment in a `.tex` file: rebuild the template, split into steps, run numerical verification, and apply the logic checklist for each.

### Example trigger

```
> Conjecture: For any real symmetric positive-definite matrix A, tr(A)·tr(A⁻¹) ≥ n²
(math-rigor auto-launches counterexample search, sampling small matrices…)
```

---

## latex-guardian — LaTeX Project Guardian

### Core capabilities

- **Protected blocks** — code wrapped between `%%% PROTECTED BEGIN` and `%%% PROTECTED END` cannot be modified by Claude **at all** (including whitespace and newlines). When asked to edit a protected block, Claude may only propose a suggestion; you must manually unprotect and replace it yourself.
- **Post-edit compile gate** — after every `.tex` edit, `latexmk -pdf` / `pdflatex` runs immediately in the background, scanning the log for `Error` and `Undefined control sequence`. **A failed compile blocks further edits to other files** and surfaces the error for immediate fixing.
- **Compile report** — on success, reports whether the main file was updated, page count, and the number/locations of Overfull/Underfull warnings.
- **Structural integrity checks** — automatically verifies:
  - `\begin{X}` / `\end{X}` are balanced;
  - `\label` and `\ref` / `\eqref` / `\pageref` / `\autoref` are paired;
  - every `\cite` key exists in `references.bib` (when present).
  - Any mismatch triggers an immediate warning and halt.
- **Pre-edit intent confirmation** — before editing any `.tex` content, Claude must output a short intent statement (what to change, why, whether it touches a protected block, expected compile impact) and wait for your approval.

### Protected block example

```latex
%%% PROTECTED BEGIN main proof of Theorem 3
\begin{proof}
... your carefully crafted proof — Claude may not touch this ...
\end{proof}
%%% PROTECTED END
```

---

## Interlocking behavior

The two skills are designed to work together:

- **math-rigor** detects whether a `latex`/`tex` guardian skill is present at startup, and triggers re-verification when a protected `proof` environment is modified.
- **latex-guardian** prompts you to invoke `math-rigor` to re-verify a proof when a protected `proof` block is unprotected and edited.

Net effect: **core proofs are physically protected (latex-guardian), and once unprotected for editing they are logically re-verified (math-rigor)** — structure and content, double-protected.

---

## Installation

Place the two `.md` files into your project's `.claude/skills/` directory (or the user-level `~/.claude/skills/`):

```bash
# Project-level (current project only)
mkdir -p .claude/skills
cp skills/math-rigor.md .claude/skills/
cp skills/latex-guardian.md .claude/skills/

# User-level (all projects)
mkdir -p ~/.claude/skills
cp skills/math-rigor.md ~/.claude/skills/
cp skills/latex-guardian.md ~/.claude/skills/
```

Claude Code auto-loads skills under `.claude/skills/` — no extra configuration needed.

### Dependencies

- **math-rigor**: Python 3 + SymPy (`pip install sympy`)
- **latex-guardian**: a TeX distribution (TeX Live / MiKTeX) with `latexmk` or `pdflatex`

### Recommended companion file

Create `math-context.md` in your paper project root (math-rigor requires it):

```markdown
# Symbols
- A: real symmetric positive-definite matrix, n×n
- ‖·‖: spectral norm

# Global assumptions
- All matrices are over the real field
- n is a positive integer

# Notation conventions
- tr(·): trace
- Inner product is the standard Euclidean one
```

---

## Usage tips

1. Wrap your most critical `proof` / `equation` blocks with `%%% PROTECTED BEGIN/END`.
2. Keep `math-context.md` up to date — clearer symbol definitions let math-rigor catch more bugs.
3. When Claude writes a new proof, it will automatically run the full "template → numbered steps → numerical verification" pipeline.
4. For any dubious claim, just ask "Is this true?" and let counterexample search precede the proof.
5. After editing `.tex`, latex-guardian auto-compiles and runs structural checks — no need to run them manually.

---

## Limitations

- These skills impose **instructional** constraints, relied upon by Claude rather than enforced by hard technical blocking. For truly hard enforcement (e.g. absolutely forbidding writes to protected blocks), pair them with Claude Code **hooks** (a PreToolUse hook intercepting Write/Edit and validating).
- Passing numerical verification **does not** equal a formal proof — math-rigor explicitly states this when "no counterexample found", and never claims a proof is complete.
- When SymPy or the compiler is unavailable, the skill honestly reports an "unverified" state and never fabricates results.

---

## License

MIT — see [LICENSE](./LICENSE).

## Contributing

Issues and PRs welcome.

## Contact information

Please contact me via the email address below if you have any questions: lhy_ynmzdx@qq.com
