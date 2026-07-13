---
name: source-verification
description: "Verify derivative content (articles, reviews, summaries) against primary source material — extract source text, audit claims, categorize findings."
version: 1.0.0
author: Hermes Agent
platforms: [linux, macos]
metadata:
  hermes:
    tags: [verification, fact-checking, editorial, auditing, research]
---

# Source Verification

Systematic workflow for fact-checking derivative content (book reviews, podcast articles, academic citations, documentation summaries) against the primary source text.

## When to Use

- A draft article claims "the author says X" or "the book divides Y into Z levels" — verify it against the actual source
- Comparing two versions of content to determine which is more accurate
- Auditing translated or summarized material for terminology drift and attribution errors
- Any situation where secondary content makes specific factual claims about a primary text

## Workflow

### 1. Extract the full source

For EPUB (reflowable books):
```bash
pip install pymupdf pymupdf4llm
python3 -c "
import pymupdf4llm
md = pymupdf4llm.to_markdown('book.epub')
with open('/tmp/source.md', 'w') as f:
    f.write(md)
"
```

For PDF: see `ocr-and-documents` skill (pymupdf with `--markdown` flag).

### 2. Map the source structure

```bash
python3 -c "
import pymupdf
doc = pymupdf.open('source_file')
for level, title, page in doc.get_toc():
    print('  '*(level-1) + f'[p{page}] {title}')
"
```

This gives you the book's real architecture — essential baseline for verifying whether a claimed framework or chapter actually exists.

### 3. Search each claim in the source

For every specific claim in the article, grep the extracted text:

```bash
grep -n 'claimed term' /tmp/source.md
```

**PITFALL**: The Hermes `search_files` tool can silently miss matches in pymupdf4llm markdown output due to encoding/formatting quirks. When `search_files` returns 0 results for a term you suspect should exist, **always fall back to `grep` in the terminal**. In one verified case, `search_files` found 0 matches for "俯瞰" while `grep` returned 16 matches in the same file. This is not a one-off — treat `search_files` results on pymupdf4llm output as suggestive only.

### 4. Read surrounding context

When a searched term is found, read ±20 lines around the match with `read_file` or the grep `-C` flag. A term existing is not enough — verify the *meaning* matches what the article claims.

### 5. Categorize by severity

| Level | Criteria | Example |
|-------|----------|---------|
| 🔴 **Must fix** | Claimed structure/framework/quote doesn't exist in source; practice attributed to wrong author | Seven-level taxonomy fabricated; "thank the item" ritual is Kondo, not Yamashita |
| 🟡 **Should fix** | Wrong terminology; translation inaccurate; interpretation presented as author's stated view; quantification where author was intentionally vague | "关系轴" should be "自我轴"; "舒服" should be "愉快"; "一年内" should be "现在" |
| 🟢 **Accurate** | Term and meaning both match the source | "7·5·1法", "1 out 1 in", "三类人" |

### 6. For each finding, state the correction explicitly

Bad: "This is wrong."
Good: "Line says '感谢仪式'. This practice is from 近藤麻理惠 (The Life-Changing Magic of Tidying Up), not from 山下英子. The source book contains no such ritual. Either remove or re-attribute."

## Common Pitfalls in Derivative Content

1. **Fabricated frameworks** — Writers invent neat taxonomies and attribute them to the source author (e.g., a non-existent "seven levels")
2. **Cross-contamination** — Mixing methods from different authors in the same genre without attribution (Yamashita's methods blended with Kondo's)
3. **Terminology drift** — Replacing the author's precise terms with colloquial near-equivalents ("愉快" → "舒服")
4. **Opinion-as-fact** — The writer's analysis or interpretation presented as the author's own stated view
5. **Spurious quantification** — Adding numerical thresholds ("one year") where the author was deliberately open-ended ("now")

## Verification Questions

For every claim in the derivative content:
1. Does this specific term/phrase appear in the source? (grep → read context)
2. Is it attributed to the correct person?
3. Is the translation/paraphrase faithful to the original meaning?
4. Is this the author's actual framework, or the article writer's interpretation? (the most common error)
5. If it's interpretation, is it clearly labeled as such?

See `references/book-audit-example.md` for a worked example from this methodology applied to a real book review audit.
