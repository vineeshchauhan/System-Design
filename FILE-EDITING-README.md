# File Editing Best Practices

This document captures important workflows and decisions for this repository.

## File Editing Workflow

### Problem
When editing files, rewriting the entire file can cause:
- Corruption of content
- Loss of original formatting
- Issues with line endings and encoding
- Difficulty tracking actual changes

### Solution
Use the `editor` tool with `insert_line` parameter to append content at specific positions instead of rewriting entire files.

### How to Add Content to a File

1. Read the file to find the last line number
2. Use `editor` with `insert_line` parameter set to that line number
3. The new content will be inserted AFTER that line

### If File Gets Corrupted

```
git checkout filename.md
```

### Verify Changes

```
git diff filename.md
```

## Recent Outcomes

- **out_0001**: File Editing Best Practices - September 2, 2026
  - Decision: Use insert_line instead of full file rewrite