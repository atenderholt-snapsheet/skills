---
name: user-runs-tests
description: Never run tests, rubocop, or Docker yourself; ask the user to run them. Applies in every session for this user, whenever you are tempted to run rspec, rake test, rails test, bundle exec rspec, rubocop (including autocorrect), a test watcher, or any docker / docker compose command to verify a change, check a failure, or see results, or when a test or DB connection error suggests a service is down. Instead, assume Docker services are up and, when you need results, ask the user to run a specific command and paste the output. Only run tests or rubocop if the user explicitly asks you to in that message.
---

# User runs tests and rubocop; Docker is assumed up

Adam wants control over when tests and linters run. Test and rubocop runs are slow, depend on his local environment, and he often already knows the result. Docker commands are a waste here because the services are up unless proven otherwise.

## Tests and rubocop

- Do not run rspec, rake test, rails test, rubocop (with or without `-a`/`-A`), or any other test or lint command on your own, including "just a quick check" after an edit and including a single file.
- Run them only when he explicitly asks in his message (e.g. "run the specs", "run rubocop").
- When you want results, ask him to run it. Give him:
  - the exact command in its own `bash` block (one command per block),
  - what to paste back (full failure output, or just the summary and failures),
  - what you're hoping to learn (e.g. "to confirm the invalid-base64 case now raises `ArgumentError`").
- Stop and wait for his output rather than guessing at results.

## Docker

- Do not run any docker or docker compose command (ps, up, logs, exec, anything), including to check whether services are running.
- Assume the DB and other services are up.
- If you hit a connection error (e.g. MySQL unreachable), don't investigate with Docker or port probes. Tell him what you saw and let him decide, e.g. "this looks like the DB isn't reachable; can you check it's up and re-run?"

## Still fine to do

Reading code and logs he points you to, and static reasoning about what a test will do or what rubocop would likely flag, are fine. Anything that would execute the test suite, rubocop, or Docker (including a plain `ruby -c` style check on his behalf, if in doubt) should be asked about instead.
