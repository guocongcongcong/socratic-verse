You are a QA reviewer for the Socratic Verse project at /Users/guoliwei/Downloads/workspace/socratic-verse/.

Run a comprehensive review. Check ALL of the following and report every issue found:

1. FILE EXISTENCE: Every file referenced in README.md, CLAUDE.md, index.md, and _database.md must exist.
2. CONTENT ACCURACY: Every claim about file counts, dates, word counts, and numbers in all documents must match reality. Check:
   - README.md says 8 methods (V1-V8) — count them in prompts/system-prompts.md
   - methodology-spectrum.md says 9 methods — are all 9 actually described?
   - Count actual files in each directory
3. FORMATTING: Check for broken markdown links ([[wikilinks]] that point to nonexistent files)
4. CONSISTENCY: Compare zhihu-article-v2.md, methodology-spectrum.md, business-plan.md, prd.md for conflicting claims about:
   - Number of methods (8 vs 9?)
   - Phase timeline
   - Market numbers
5. COMPLETENESS: Check if any teacher files are missing sections required by the socratic-md-tutor skill:
   - Each character must have: 基础信息, 外貌, 性格(core/social/emotional/hidden/flaws), 家庭背景, 学业, 人生愿景, 授课风格, 经典小动作, 对我的态度, 教学备注, 与另外两位的关系
6. SPELLING/TYPOS: Scan all .md files for obvious errors
7. OBSIDIAN: Does _database.md have valid frontmatter? Does index.md link to existing files?

Report EVERY issue found. Be exhaustive — no issue is too small. Output in this format:
- [SEVERITY] file: description of issue

SEVERITY levels: CRITICAL (broken link/wrong number), HIGH (missing section), MEDIUM (inconsistency), LOW (typo)
