---
name: ripgrep-first
description: Prefer ripgrep (rg) over grep, fd over find, ast-grep for structural code search and refactors, sd over sed for find-and-replace, and jq/yq for JSON/YAML, whenever working from the shell. Use this skill any time a task involves searching a codebase, finding where a symbol, string, route, config key or error message is defined or used, listing files by name or type, renaming or migrating an API across many files, inspecting package.json or locale files, or running grep, egrep, find, sed -i or `grep -r` in a bash command, especially in Ruby/Rails and JavaScript/TypeScript/React projects. Also use it when exploring an unfamiliar repo, doing a refactor sweep, or auditing for usages, even if the user never mentions these tools.
---

# Fast search and edit toolkit

Use `rg` instead of `grep` and `fd` instead of `find` for searching from the shell; reach for `ast-grep` when the question is about code structure, `sd` for bulk replacements, and `jq`/`yq` for structured data. They are faster, and more importantly they behave better by default in a real project: they respect `.gitignore`, skip `.git/`, `node_modules/`, `tmp/`, `log/`, `vendor/` and build output, and skip binary files. That means less noise in the results and fewer wasted tokens reading matches from generated or vendored code.

If the environment has a dedicated search tool (for example a built-in Grep tool), that is fine to use. This skill is about what to reach for in a bash command.

## Check availability once

Run `command -v rg fd ast-grep sd jq yq` before relying on them. If `rg` is missing, fall back to `git grep -n <pattern>` inside a git repo (it also respects `.gitignore`), or `grep -rn --exclude-dir={.git,node_modules,tmp,log,vendor,coverage,dist,build} <pattern> .` (in zsh quote flag globs such as `'--include=*.rb'`, or it errors with "no matches found"; macOS has no `timeout` command, so don't wrap searches in it). Without those excludes plain `grep -r` can be dozens of times slower (3.4s vs 0.06s in one Rails repo, because it walks `log/`, `tmp/` and binaries). Mention to the user that installing ripgrep would speed things up; do not try to install tools unprompted, but do offer the install command (see the install notes at the end). Each tool below degrades gracefully: no `ast-grep` means use rg plus reading the files; no `sd` means `perl -pi -e` or `sed -i`; no `jq` means a short `ruby -rjson` or `node -e` one-liner.

## Basic usage

**Always pass a path (`rg 'pattern' .`) in scripts and agent shells.** When stdin is not a TTY (any non-interactive shell) and no path is given, rg reads stdin instead of the working directory and hangs forever waiting for input. If a path is impossible, redirect with `</dev/null`.

```bash
rg 'pattern' .                   # recursive from cwd, respects .gitignore (path needed in non-TTY shells, see above)
rg -n 'pattern' app/             # with line numbers (rg adds them only on a TTY, so pass -n when piping)
rg -F 'literal.string(' lib/     # fixed string, no regex escaping needed
rg -w 'User' app/                # whole word only
rg -S 'userid'                   # smart case: case-insensitive unless the pattern has uppercase
rg -l 'pattern'                  # list matching files only
rg -c 'pattern'                  # match count per file
rg -C 3 'pattern' file.rb        # 3 lines of context (-A after, -B before)
rg --files-without-match 'x'     # files that do NOT match
```

Start narrow and widen: search the most likely directory first (`app/`, `src/`, `lib/`), then the whole repo if nothing turns up.

## Filter by file type, not by guessing extensions

```bash
rg -t ruby 'has_many :orders'
rg -t js -t ts 'useEffect'            # .js/.jsx/.mjs/.cjs and .ts/.tsx
rg -T ruby 'render'                   # everything except ruby
rg -g '*.erb' 'form_with'             # glob when there is no built-in type
rg -g '!*.test.*' -g '!spec/**' 'pattern'   # exclude with a leading !
rg --type-list                        # see what types exist before assuming
```

Run `rg --type-list | grep -i <name>` if unsure whether a type covers a file kind (for example ERB, Haml, or Stimulus controllers). Add a custom type for the session with `--type-add 'view:*.{erb,haml,slim}' -t view`.

## Habits from grep that do the wrong thing in rg

These flags exist in both tools but mean different things. Getting them wrong gives confusing output rather than an error, so watch for them:

| You wrote (grep habit) | What rg does | Use instead |
|---|---|---|
| `rg -r` | `-r` is *replace*, not recursive. rg is already recursive | omit it |
| `rg -R` | no such flag | omit; add `-L` only if you need to follow symlinks |
| `rg -L` | follow symlinks | `--files-without-match` for grep's `-L` |
| `rg -h` | prints help | `--no-filename` (or `-I`) |
| `rg -E 'a\|b'` | `-E` is *encoding* | just `rg 'a\|b'`; rg regex is extended by default |
| `rg --include='*.rb'` | no such flag | `-g '*.rb'` or `-t ruby` |
| `rg '\(foo\)'` basic-regex escapes | rg uses Rust regex: `(foo)` groups, `\(` is a literal paren | write extended regex |

Rust regex has no lookaround or backreferences. When you need them, add `-P` (PCRE2): `rg -P '(?<!#)\bbinding\.pry\b'`. For patterns spanning lines use `-U` (add `--multiline-dotall` if `.` should match newlines): `rg -U 'def create\n.*render'`.

## Searching things that are ignored

By default rg hides what `.gitignore` hides. Widen deliberately:

```bash
rg --hidden 'pattern'        # include dotfiles/dirs (still honours .gitignore); add -g '!.git' to skip .git/
rg -u 'pattern'              # also ignore .gitignore rules
rg -uu 'pattern'             # also search hidden files
rg -uuu 'pattern'            # also search binary files; last resort
rg -g '!node_modules' --no-ignore 'pattern'   # ignore rules off, but keep one exclusion
```

If a search that should obviously hit returns nothing, suspect an ignore rule before assuming the code is absent, and retry with `--no-ignore --hidden`. Searching `node_modules` or gems is occasionally useful (reading a library's source); target the specific package path rather than the whole tree.

## File finding with fd

```bash
fd -e rb -e erb app/views            # by extension
fd -t f 'user' app/                  # files whose name matches 'user'
fd -t d 'components' src/            # directories
fd -H -E node_modules 'config'       # include hidden, exclude a path
fd -e rb -x wc -l                    # run a command per result
```

On Debian/Ubuntu the binary may be called `fdfind`. `rg --files -g '*.rb'` also lists files and works when fd is absent.

## Project-shaped examples

Rails:
```bash
rg -t ruby 'before_action' app/controllers
rg -n 'def (create|update)\b' app/controllers -t ruby
rg -w 'Order' -g '!spec/**' -t ruby -l          # which non-test files touch the Order model
rg 'user_path|users_path' app/views app/helpers  # route helper usage
rg -g '*.erb' 'render partial:'
rg 'ActiveSupport::Notifications' -t ruby
```

JavaScript / React:
```bash
rg -t ts -t js "from '@/components/Button'"      # importers of a module
rg -g '*.{jsx,tsx}' 'useContext\('
rg -g '*.{jsx,tsx}' '<Modal\b' -l                # components that render <Modal>
rg 'process\.env\.[A-Z_]+' -o --no-filename | sort -u   # every env var referenced
```

## When text search is the wrong tool: ast-grep

rg matches text, not syntax. It will match inside comments and strings, and it cannot tell a method call from a definition or see through formatting differences. When the question is structural, use `ast-grep` (binary name `ast-grep`; the short alias `sg` collides with a Linux system command, so avoid it).

Patterns are real code in the target language, with metavariables: `$NAME` matches one node, `$$$ARGS` matches zero or more. Use `-l` to set the language (`ruby`, `tsx` for both TS and JSX, `js`, `ts`, `html`).

```bash
# Search
ast-grep run -l ruby -p 'before_action :$NAME' app/controllers
ast-grep run -l ruby -p 'has_many :$ASSOC, through: :$THROUGH' app/models
ast-grep run -l tsx  -p 'useEffect($$$ARGS)' src/
ast-grep run -l tsx  -p '<Button $$$PROPS />' src/              # JSX usages regardless of prop order/line breaks
ast-grep run -l tsx  -p 'fetch($URL, { method: "POST", $$$REST })' src/

# Rewrite: preview first (default prints a diff), then apply with -U
ast-grep run -l ruby -p 'update_attributes($$$A)' -r 'update($$$A)' app/ lib/
ast-grep run -l ruby -p 'update_attributes($$$A)' -r 'update($$$A)' app/ lib/ -U
ast-grep run -l tsx  -p 'console.log($$$A)' -r '' src/ -U       # delete debug logging (check the diff for statement-level removal)
```

Tips:
- A pattern must parse as valid code on its own. If it won't, wrap it in the surrounding construct (for example give a method call a receiver or write the whole `def ... end`).
- `--debug-query` shows how a pattern parses when it doesn't match what you expect.
- Prefer ast-grep over rg when text search would produce false positives (comments, strings, similarly named identifiers) or when a rewrite needs to survive multi-line formatting. Prefer rg for plain "where is this string mentioned" questions; it is faster to write and enough most of the time.
- Always show the user the preview diff or `git diff` after applying a rewrite, and run the project's tests or linter.

Other precise tools beat text search for their own domains: `bin/rails routes -g users`, `bin/rails runner`, `tsc --noEmit`, ESLint, Ruby LSP/Solargraph. Say which tool you used so the user can judge the scope of the result.

## Bulk find-and-replace: sd

`sd` is a simpler `sed`: regular expressions by default, `$1`/`${1}` for capture groups, no escaping gymnastics, and the same syntax on macOS and Linux (unlike `sed -i`).

```bash
sd -p 'OldName' 'NewName' app/models/user.rb          # -p previews the change without writing
sd 'OldName' 'NewName' app/models/user.rb             # edits in place
sd -F 'foo.bar(' 'foo.baz(' lib/thing.rb              # literal strings, no regex
sd '(\w+)_path\b' '${1}_url' app/views/mailers/*.erb   # capture groups
rg -l 'OldName' -t ruby | xargs sd 'OldName' 'NewName'    # repo-wide: find files with rg, rewrite with sd
```

`sd` edits files in place with no backup, so make sure `git status` is clean (or commit/stash first), preview with `-p` on a representative file, and review `git diff` afterwards. For renames that must respect syntax (not touch comments, strings, or unrelated identifiers), use ast-grep's `-r` instead. Rename files themselves with `git mv`, so history follows.

## Structured data: jq and yq

Read JSON and YAML with a parser rather than grepping them; it avoids false matches and handles nesting.

```bash
jq -r '.dependencies | keys[]' package.json                  # list dependencies
jq '.scripts' package.json
jq -r '.. | .version? // empty' package.json | head          # recursive lookup
yq '.en.activerecord.models' config/locales/en.yml           # mikefarah/yq (Go) syntax
yq -i '.defaults.timeout = 30' config/settings.yml           # in-place edit; clean git state first
```

Two different programs are both called `yq` (mikefarah's Go version, and a Python wrapper around jq). Run `yq --version` if a command errors; the examples above assume the Go version.

## Output hygiene

- Cap noisy searches: `rg ... | head -50`, or `-m 3` to stop after 3 matches per file. Prefer `-l` or `-c` first when the hit count may be huge, then drill into specific files.
- Add `--max-columns 200 --max-columns-preview` when minified or generated files might slip through.
- Use `--no-heading -n` (or `--vimgrep`) when you want one `path:line:col:text` per line for further processing; use `--json` for programmatic parsing.
- Quote patterns with single quotes so the shell doesn't expand `$`, `*`, or `!`.

## Install notes (offer, don't run unprompted)

If tools are missing and the user wants them, offer the one-liner for their platform:

```bash
# macOS (Homebrew)
brew install ripgrep fd ast-grep sd jq yq

# Debian/Ubuntu (fd installs as `fdfind`; alias it: ln -s $(which fdfind) ~/.local/bin/fd)
sudo apt install ripgrep fd-find jq
npm install -g @ast-grep/cli          # or: cargo install ast-grep --locked
cargo install sd                      # or: apt install sd on newer releases

# Any platform with Rust
cargo install ripgrep fd-find sd ast-grep --locked
```

In sandboxed or network-restricted environments, installs will fail; say so plainly and continue with the fallbacks listed under availability rather than retrying.
