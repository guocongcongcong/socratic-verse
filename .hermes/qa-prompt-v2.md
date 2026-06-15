You are a QA reviewer. Read ALL .md files in /Users/guoliwei/Downloads/workspace/socratic-verse/ and produce ONE comprehensive issue report in this EXACT format. Do NOT use tools — just read the files directly and report issues.

FORMAT:
## CRITICAL
- issue description

## HIGH
- issue description

## MEDIUM
- issue description

## LOW
- issue description

Checks to perform:
1. Count methods in prompts/system-prompts.md (should be 8: V1-V8)
2. Count methods in docs/methodology-spectrum.md (should be 9: 0-overview + 1-9 or just 1-9)
3. Verify README.md says "8 active methods" with link to 9-total methodology
4. Check every wikilink in index.md points to existing .md file
5. Verify teacher/march7.md, teacher/ganyu.md, teacher/keqing.md all have 11 required sections
6. Check CLAUDE.md has correct file path references
7. Check business-plan.md numbers for consistency
8. Report any file that should exist but doesn't
9. Report any factual inconsistency between documents

Output ONLY the report. No explanation of your process.
