# Worked Example: Auditing a 《断舍离》Book Review

This is a real audit from a podcast book club article ("路上书简") about 山下英子's《断舍离》(湖南文艺出版社, 2019).

## Source

- EPUB: 《断舍离》by 山下英子, translated by 贾耀平
- 185 pages, extracted to 86,958 characters of markdown via pymupdf4llm

## Findings

### 🔴 Must Fix

#### Fabricated: "七层逻辑" (Seven-Level Classification)
Both article drafts claim 山下英子 divides accumulation into 7 levels (可见垃圾 → 隐藏堆积 → 囤积 → 身份执着 → 决策瘫痪 → 焦虑驱动 → 深层执念). **This framework does not exist in the book.** grep returned zero matches for any variation. It is entirely fabricated by the article writer.

#### Misattributed: "感谢仪式" (Gratitude Ritual)
The article presents "对物品说谢谢" as 山下英子's suggested method. **This is 近藤麻理惠's (Marie Kondo) signature practice from《怦然心动的人生整理魔法》.** 山下英子's book has no such ritual. Her method is rational: use "自我轴" and "时间轴" to decide — not emotional farewell ceremonies.

#### Wrong term: "关系轴" → should be "自我轴"
The book uses "自我轴" (self-axis), contrasted with "他人轴" (other-axis) and "物质轴" (material-axis). "关系轴" is not a term used in the book.

### 🟡 Should Fix

| Article says | Book says | Issue |
|-------------|-----------|-------|
| "舒服" | "愉快" | Different meaning: "愉快" = pleasant/joyful (emotional), "舒服" = comfortable (physical) |
| "断·舍·离不是并列，是递进" | 三字诀 (three-part classification like 松竹梅) | The "递进" framing is the writer's interpretation, not the author's stated view |
| "一年内用过吗？" | "现在的我需要吗？" | The author deliberately avoided time quantification — adding "一年内" misrepresents the method |
| "俯瞰力" as practice rule | "俯瞰力" as higher-level result | The book places 俯瞰力 as an outcome of practice, not one of the five收纳指南 |

### 🟢 Accurate

- "7·5·1法" (看不见7成 / 看得见5成 / 展示1成) ✓ — matches book exactly
- "1 out 1 in法" ✓ — matches book
- "三类人" (逃避现实型 / 执着过往型 / 忧虑未来型) ✓ — matches book
- "物质轴"思维 ✓ — matches book
- "舍与弃的不同" ✓ — matches book

## Tool Pitfall

`search_files` (Hermes tool) silently returned 0 results for "俯瞰" in the pymupdf4llm markdown output. Terminal `grep` found 16 matches in the same file. **Always validate search_files results with grep when working with pymupdf4llm output.**
