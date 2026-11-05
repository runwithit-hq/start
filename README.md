# Run With It: start here

For new Run With It teammates. This sets up Claude Code on your Mac with the team's skills.

1. Open Terminal (Applications → Utilities) and paste:
   ```
   claude plugin marketplace add runwithit-hq/start && claude plugin install rwi-start@rwi-start
   ```
   No Claude Code on this Mac yet? Install it first with `curl -fsSL https://claude.ai/install.sh | bash`, then run `claude` once to sign in.
2. Open Claude Code and type `/rwi-start:setup`.

Claude walks you through the rest: your Mac's tools, GitHub, joining the team, and installing the skills. You type your own passwords; Claude never asks for them.

Nothing in this repository is private. The team's skills and client folders live in private repositories you get access to during setup.
