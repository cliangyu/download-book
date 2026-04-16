# download-book

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) slash command that downloads books from LibGen — with automatic mirror fallback and 4-layer file verification.

<p align="center">
  <img src="https://covers.openlibrary.org/b/isbn/9780374533557-M.jpg" height="160" alt="Thinking, Fast and Slow" />
  <img src="https://covers.openlibrary.org/b/isbn/9780062316097-M.jpg" height="160" alt="Sapiens" />
  <img src="https://covers.openlibrary.org/b/isbn/9780735211292-M.jpg" height="160" alt="Atomic Habits" />
  <img src="https://covers.openlibrary.org/b/isbn/9780804139298-M.jpg" height="160" alt="Zero to One" />
  <img src="https://covers.openlibrary.org/b/isbn/9780062060242-M.jpg" height="160" alt="The Innovator's Dilemma" />
</p>

```
> /download-book Thinking, Fast and Slow by Daniel Kahneman

Searching libgen.li... found 8 candidates
Downloading: Daniel Kahneman - Thinking, Fast and Slow (EPUB, 2.6MB)
Verifying... file type OK, size OK, not a stub

 DONE  ~/Downloads/Daniel_Kahneman-Thinking_Fast_and_Slow.epub (2.6 MB)
```

Batch mode downloads multiple books in parallel:

```
> /download-book Sapiens; Atomic Habits; Zero to One; The Innovator's Dilemma

Launching 2 parallel agents...

 DONE  Yuval_Noah_Harari-Sapiens.epub                    (2.1 MB)
 DONE  James_Clear-Atomic_Habits.epub                    (3.4 MB)
 DONE  Peter_Thiel-Zero_to_One.epub                      (1.8 MB)
 DONE  Clayton_Christensen-The_Innovators_Dilemma.epub    (1.2 MB)

4/4 verified — all files in ~/Downloads/
```

## Install

```bash
git clone https://github.com/cliangyu/download-book.git ~/.claude/commands/download-book
```

Or copy `SKILL.md` directly into `~/.claude/commands/download-book/`.

## How it works

1. **Search** LibGen mirrors for the book, extracting multiple download candidates
2. **Filter** results to English editions, sorted by format preference (EPUB > PDF > MOBI)
3. **Download** the best candidate with automatic retry across up to 5 alternatives
4. **Verify** every download with 4 checks:
   - HTTP status (catches dead links returning 307)
   - File type (catches error pages served as fake book files)
   - Minimum size (catches metadata-only stubs under 50KB)
   - Page count for PDFs (catches 2-page preview stubs)

If a download fails any check, the file is deleted and the next candidate is tried automatically. You get a verified book or a clear failure report — never a corrupt file.

## Configuration

**Download directory:** Files are saved to `$HOME/Downloads/` by default. Specify a different path in your request:

```
/download-book Sapiens — save to ~/Books/
```

**Language:** English editions are selected by default. Request other languages explicitly:

```
/download-book Sapiens in Spanish
```

## Limitations

- Depends on LibGen mirror availability — if all mirrors are down, the skill cannot download
- No audiobook support (LibGen hosts text formats only)
- Some books only exist in PDF format; EPUB is preferred when available
- LibGen mirrors change over time — if the primary mirror stops working, the skill includes fallback mirrors but they may need updating

## License

MIT
