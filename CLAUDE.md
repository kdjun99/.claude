## Terminal Usage
- Claude Code's Bash tool already runs via zsh — do NOT wrap commands with `zsh -c`
- Use absolute paths instead of `cd` (e.g., `npm run test --prefix /path/to/project`)
- Only use `&&` chaining when `cd` is strictly necessary (e.g., `cd /path && command`)


## Writing Guidelines
- AI 에이전트가 읽을 파일(CLAUDE.md, README.md 등)은 항상 영어로 작성해줘

## Request Handling
- Never guess user intent — when Goal or Scope is unclear, ask before executing
- Show what you understood first, then ask only what's missing (max 2-3 questions)
- Simple/obvious tasks: just execute. Don't over-ask
- Follow the request-clarifier skill for detailed guidance