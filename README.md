<p align="center">
  <h1 align="center">download-book</h1>
  <p align="center">
    A <a href="https://docs.anthropic.com/en/docs/claude-code">Claude Code</a> slash command that downloads books — with automatic mirror fallback and 4-layer file verification.
  </p>
</p>

<br />

<p align="center">
  <strong>One command. Any book. Verified.</strong>
</p>

<br />

<table align="center">
  <tr>
    <td align="center" width="130">
      <img src="https://covers.openlibrary.org/b/isbn/9780374533557-L.jpg" width="100" alt="Thinking, Fast and Slow" /><br />
      <sub><b>Thinking, Fast<br/>and Slow</b></sub>
    </td>
    <td align="center" width="130">
      <img src="https://covers.openlibrary.org/b/isbn/9780062316097-L.jpg" width="100" alt="Sapiens" /><br />
      <sub><b>Sapiens</b></sub>
    </td>
    <td align="center" width="130">
      <img src="https://covers.openlibrary.org/b/isbn/9780735211292-L.jpg" width="100" alt="Atomic Habits" /><br />
      <sub><b>Atomic Habits</b></sub>
    </td>
    <td align="center" width="130">
      <img src="https://covers.openlibrary.org/b/isbn/9780804139298-L.jpg" width="100" alt="Zero to One" /><br />
      <sub><b>Zero to One</b></sub>
    </td>
    <td align="center" width="130">
      <img src="https://covers.openlibrary.org/b/isbn/9780062060242-L.jpg" width="100" alt="The Innovator's Dilemma" /><br />
      <sub><b>The Innovator's<br/>Dilemma</b></sub>
    </td>
  </tr>
</table>

<br />

```
> /download-book Thinking, Fast and Slow

  Searching libgen.li... found 8 candidates
  Downloading: Daniel Kahneman - Thinking, Fast and Slow (EPUB, 2.6 MB)
  Verifying... file type ✓  size ✓  not a stub ✓

  ✅ ~/Downloads/Daniel_Kahneman-Thinking_Fast_and_Slow.epub (2.6 MB)
```

### Batch mode

Download an entire reading list in one shot — parallel agents handle multiple books simultaneously:

```
> /download-book Sapiens; Atomic Habits; Zero to One; The Innovator's Dilemma

  Launching 2 parallel agents...

  ✅ Yuval_Noah_Harari-Sapiens.epub                    2.1 MB
  ✅ James_Clear-Atomic_Habits.epub                    3.4 MB
  ✅ Peter_Thiel-Zero_to_One.epub                      1.8 MB
  ✅ Clayton_Christensen-The_Innovators_Dilemma.epub    1.2 MB

  4/4 verified
```

---

## Install

```bash
git clone https://github.com/cliangyu/download-book.git ~/.claude/commands/download-book
```

Or copy `SKILL.md` directly into `~/.claude/commands/download-book/`.

## How it works

Every download goes through a **4-layer verification pipeline** — you get a real book or a clear failure report, never a corrupt file:

| Check | What it catches |
|-------|----------------|
| **HTTP status** | Dead links returning 307 redirects |
| **File type** | Error pages served as fake `.epub` files |
| **Minimum size** | Metadata-only stubs under 50 KB |
| **Page count** | 2-page PDF previews pretending to be full books |

If any check fails, the file is deleted and the next mirror candidate is tried automatically — up to 5 attempts per book.

## Configuration

**Download directory** — files go to `$HOME/Downloads/` by default:

```
/download-book Sapiens — save to ~/Books/
```

**Language** — English editions are selected by default:

```
/download-book Sapiens in Spanish
```

## Limitations

- Depends on LibGen mirror availability — if all mirrors are down, the skill cannot download
- No audiobook support (LibGen hosts text formats only)
- Some books only exist in PDF; EPUB is preferred when available
- LibGen mirrors change over time and may need updating

## License

MIT
