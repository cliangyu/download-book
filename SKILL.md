---
name: download-book
description: >
  Download books from LibGen with automatic mirror fallback, 4-layer file
  verification (HTTP status, file type, size, stub detection), and batch
  support. Handles dead links, stub PDFs, and broken mirrors automatically.
---

# Download Book from LibGen

Download a book by searching LibGen mirrors and finding a working download link. Guarantees a verified, readable file or reports failure with reasons.

## Input

$ARGUMENTS = book title, author, or ISBN to search for. Multiple books separated by semicolons.

**Examples:**
- `/download-book The Art of War by Sun Tzu`
- `/download-book Principles by Ray Dalio`
- `/download-book The Prince; 48 Laws of Power; Mastery`

## Procedure

### Step 1: Search

Use curl (never a browser — curl is faster and avoids cert/protocol issues):

```
curl -sL --max-time 20 "https://libgen.li/index.php?req=QUERY&search_mode=&column=def&page=1"
```

Extract ALL MD5 hashes from results: `grep -o 'ads.php?md5=[a-f0-9]*' | sort -u`. Save them as a **candidate list** — you will need fallbacks when downloads fail.

Also extract book metadata (title, author, year, format, size) from the table rows.

**Language filtering:** Filter results to English editions by default. Non-English editions (Russian, Hindi, etc.) appear frequently in results — skip them unless the user explicitly requests another language.

**If the search returns 0 results or times out:** Try `libgen.com.im` as a search-only fallback (its search works, but its download server is often broken — use the MD5 hashes it returns with the `libgen.li` download chain instead). If all mirrors fail, report that the service may be temporarily down.

**Mirror note:** LibGen mirrors change over time. If `libgen.li` stops working, check for current mirrors before assuming the book doesn't exist.

### Step 2: Present options

Show up to 5 matching English results, sorted by format preference (EPUB > PDF > MOBI):
- Title, author, year, format, size

**Skip this step** if the user already specified a clear book+author or if running in batch mode. In those cases, auto-select the best English EPUB (or PDF if no EPUB exists).

### Step 3: Download loop (auto-retry with candidate list)

**Before downloading:** Check if a file with the same name already exists in the target directory. If it does, warn the user and skip unless they confirm overwrite.

For each MD5 candidate (up to 5 attempts), run the full download-and-verify cycle:

#### 3a. Get the download key

The download chain on `libgen.li` requires a session-specific key. You must fetch `ads.php` first to get it — direct `get.php` calls without a key will fail.

```
curl -sL --max-time 20 "https://libgen.li/ads.php?md5=CHOSEN_MD5"
```
Extract the `get.php?md5=...&key=...` link from the response (look for `href="get.php?md5=`).

If no key found, skip to next MD5 candidate.

#### 3b. Download
```
curl -sL --max-time 120 -o "$HOME/Downloads/FILENAME.ext" -w "%{http_code} %{size_download}" "https://libgen.li/get.php?md5=...&key=..."
```

Name files: `Author-Title.ext` with underscores for spaces, no special characters. The default download directory is `$HOME/Downloads/` — use a different path if the user specifies one.

#### 3c. Verify (ALL checks must pass)

Run these checks in order. If ANY check fails, delete the bad file and try the next MD5 candidate.

**Check 1: HTTP status**
- `200` = proceed to next check
- `307` or `0 bytes` = dead link (this is common — some MD5s are simply gone), next candidate

**Check 2: File type**
```
file "$HOME/Downloads/FILENAME.ext"
```
- Must report "EPUB document", "Mobipocket E-book", or "PDF document"
- "Mobipocket E-book" on an `.epub` file is normal and not an error — it opens fine in most e-readers
- If it says "HTML document", "ASCII text", or "empty", the download returned an error page, not a book — next candidate

**Check 3: Minimum file size**
- Books should be > 50KB. A "book" under 50KB is almost certainly a stub, metadata-only file, or error page.
- Exception: very short public domain works can be legitimately small (e.g., 100-200KB). Use judgment.

**Check 4: Not a stub/preview**
- If PDF: check page count with `mdls -name kMDItemNumberOfPages` (macOS) or `pdfinfo` (Linux). A real book should have > 10 pages. A 2-page "book" is a stub — next candidate.
- If EPUB/MOBI: file size > 100KB is generally sufficient. The format check in Check 2 covers most issues.

If all 4 checks pass, the download is **verified successful** — stop trying candidates.

If all 5 candidates fail, report failure to user with what was tried and why each failed.

### Step 4: Final report

For each book, report:
- Filename and path
- Format and size
- Verification status (PASS/FAIL)

For failed books, report which MD5s were tried and why they failed (307, HTML, stub, etc.), so the user can decide next steps.

## Batch downloads

When downloading multiple books (semicolon-separated input), parallelize aggressively:
- Step 1 (search) can run in parallel for different books
- Step 3 (download loop) can run in parallel for different books
- For large batches (5+ books), split across multiple parallel agents, 2-3 books per agent
- Each agent follows this same procedure independently

**Example:** `/download-book The Prince; 48 Laws of Power; Mastery; Art of War; Influence` will launch 2-3 parallel agents, each handling 2-3 books.

## Fallback mirrors

If `libgen.li` is completely unreachable, try these in order:

1. **Anna's Archive** — `https://en.annas-archive.gl/md5/HASH` (these links also appear in `libgen.li` search results as badge links)
2. **libgen.com.im** — search works but download server often returns 502. Use it for search only, then try Anna's Archive links for the actual download.
3. **libgen.rs / libgen.is** — frequently timeout but worth a try as last resort

## Limitations

- Depends on LibGen mirror availability — if all mirrors are down, the skill cannot download
- No audiobook support (LibGen hosts text formats only)
- Format preference is EPUB > PDF > MOBI, but some books only exist in one format
- PDF page count check uses macOS `mdls` by default; on Linux, install `poppler-utils` for `pdfinfo`
