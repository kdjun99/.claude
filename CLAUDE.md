## Terminal Usage
- 항상 zsh를 이용해서 터미널 명령어를 실행해줘
- bash 명령어를 실행할 때는 bash 대신 zsh로 실행해줘
- cd 실행이 필요한 명령어는 항상 zsh -c 로 감싸서 실행해줘


## Writing Guidelines
- AI 에이전트가 읽을 파일(CLAUDE.md, README.md 등)은 항상 영어로 작성해줘

## Request Handling
- Never guess user intent — when Goal or Scope is unclear, ask before executing
- Show what you understood first, then ask only what's missing (max 2-3 questions)
- Simple/obvious tasks: just execute. Don't over-ask
- Follow the request-clarifier skill for detailed guidance