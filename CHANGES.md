# Changes in this fork

This is a modified fork of **[lakiw/compiled-pcfg](https://github.com/lakiw/compiled-pcfg)**
by Matt Weir. The original work and this fork are licensed under the
**GNU General Public License, version 2** (see `LICENSE`). The original
copyright and author attribution are retained in all source files.

Per GPLv2 §2(a), the modifications below are documented with the date of change.

## 2026-11-13 — Hiraishi Tasuku (Kyoto University)

Fixes that make `compiled-pcfg` build and run correctly on Linux (glibc, LF
line endings) against rulesets produced by the Python
[`lakiw/pcfg_cracker`](https://github.com/lakiw/pcfg_cracker) trainer.
The upstream code only worked with CRLF rulesets and crashed on Linux.

1. **`src/config_parser.c` — `get_key()`: newline stripping assumed CRLF.**
   The code removed a fixed 2 trailing characters (`result[result_len-2] = '\0'`),
   which corrupts every value by one character on LF files
   (e.g. `utf-8` → `utf-`, so the encoding check rejected valid rulesets).
   Now strips trailing `\r`/`\n` in a loop, handling LF, CRLF and a missing
   final newline.

2. **`src/config_parser.c` — `get_key()`: 1-byte stack buffer overflow.**
   `section_name` / `key_name` were not allocated room for the terminating
   `'\0'`, so `strncat` wrote out of bounds (SIGSEGV; confirmed with
   AddressSanitizer). Buffers are now sized correctly and built with
   `snprintf`.

3. **`src/grammar_io.c` / `src/pcfg_guesser.c` — empty terminal files were fatal.**
   A category with no terminals (e.g. the trainer emits an empty
   `Context/1.txt` when no context-sensitive strings were seen) made
   `load_term_from_file()` return `NULL`, aborting the whole load.
   `load_terminal()` now skips empty terminal files, and the `PcfgGrammar`
   struct in `main()` is zero-initialized (`= {0}`) so unloaded terminal
   slots are well-defined `NULL`.

4. **`src/grammar_io.c` — zero-length terminal (`id == 0`) was rejected.**
   On large datasets the trainer emits a length-0 terminal (e.g.
   `Other/0.txt` holding the empty string). `load_terminal()` treated the
   filename id `0` as invalid and aborted (`"Invalid File name found in
   rules"`). The base-structure grammar never references a length-0 segment
   and the generator indexes terminals by positive length, so such files are
   now skipped. Without this, generating from a full rockyou.txt ruleset
   (~14.3M passwords) failed to load.

To reproduce the build and run:

```sh
make                      # production build (gcc, pure C; no libJudy needed)
make DEBUG=2              # AddressSanitizer build for debugging
./pcfg_guesser -r <rulename>   # reads ./Rules/<rulename>/config.ini
```
