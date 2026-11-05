---
name: setup
description: First-day setup for a new Run With It teammate, from nothing to the team's skills installed - the Mac tools (Homebrew, git, GitHub CLI, gitleaks), a GitHub account and login, joining the runwithit-hq organization, checking access to the team's private skills (the brain), installing them, then handing over to /rwi:teammate-setup for the rest. Use when someone says "set me up", "onboard me", "Run With It setup", "install the Run With It skills", "I just joined Run With It", "get me into the brain", or right after installing the rwi-start plugin.
---

# Run With It: first-day setup

The person you're talking to just joined Run With It. By the end of this they can use the team's skills, and the next skill (`/rwi:teammate-setup`) connects their Mac to the client folders so their work and Luis's land in the same place.

Say this once, at the start, in your own words:

> We'll do four things: get your Mac's tools ready, sign you in to GitHub (where the team's skills live), join the Run With It organization, and install the skills. About 15 minutes. I'll tell you exactly what to click; you type your own passwords.

## Rules

- **They type every password and 2FA code themselves**, in GitHub's page or in Terminal. Never ask for one, never type one, never repeat one back. If they paste a password, key or token into the chat, tell them to change it and let Luis know.
- **They click Join on the GitHub invitation themselves.** You check that it worked.
- Never use `sudo` and never change system or security settings. The Homebrew installer asks for their Mac password: they run that one in Terminal themselves.
- One step at a time. After each step, check it with the command given rather than asking "did it work?", and say what changed in one line.
- If a step fails twice, stop and give them the exact error line to send Luis in Slack.
- Commands they run themselves go in Terminal (Applications → Utilities → Terminal). In the Claude Code terminal app they can also type `!` and the command.

## 1. The Mac and its tools

1. `uname -s` must say `Darwin`. If it doesn't, stop: the team's scripts assume a Mac. Tell them to ask Luis.
2. Check what is there: `for t in brew git gh python3 gitleaks claude; do printf '%s: ' $t; command -v $t || echo missing; done`
   Apple Silicon Macs keep Homebrew in `/opt/homebrew/bin`; if `brew` is missing, also try `/opt/homebrew/bin/brew --version` before calling it missing.
3. Fill the gaps, in this order:
   - **git or python3 missing:** they run `xcode-select --install` and click **Install** in the window that opens (about 5 minutes).
   - **brew missing:** they open Terminal, paste the install line from https://brew.sh, enter their Mac password when asked, then run the two `echo … >> ~/.zprofile` and `eval …` lines Homebrew prints under "Next steps". Then they quit and reopen Claude Code so it sees `brew`.
   - **gh or gitleaks missing:** you run `brew install gh gitleaks`.
   - **claude missing** (they use only the desktop app): they run `curl -fsSL https://claude.ai/install.sh | bash` in Terminal. The app and the command line share the same plugins.
4. Run the check from 2 again. Every tool shows a path.

## 2. GitHub

1. `gh auth status`. If it says `Logged in to github.com account <name>`, skip to 4.
2. Ask: "Do you have a GitHub account?" If not, they sign up at https://github.com/signup with their work email, verify the email, and turn on two-factor authentication (Settings → Password and authentication). Wait until they say it's done.
3. They sign in from Terminal:
   ```
   gh auth login --hostname github.com --git-protocol https --web
   ```
   Tell them what they will see: a one-time code. They copy it, press Enter, paste the code in the browser page that opens, and click **Authorize GitHub**. If asked "Authenticate Git with your GitHub credentials?", the answer is **Y**.
4. You run `gh auth setup-git`, so git and Claude Code use this login for the team's private repositories.
5. You run `gh api user --jq .login` and note their username.

## 3. Join runwithit-hq

Run `gh api user/memberships/orgs/runwithit-hq --jq .state`:

| Answer | What to do |
|---|---|
| `active` | They're in. Go to 4. |
| `pending` | They open https://github.com/orgs/runwithit-hq/invitation and click **Join Run With It** (the invitation email has the same button). Check again. |
| `Not Found` (HTTP 404) | No invitation yet. They send Luis their username: "My GitHub username is `<login>`." Luis sends the invite from his side. Stop here and tell them to come back to this step when the email arrives. |

## 4. The brain: the team's skills

1. Check access (this is Luis's first test, so report the answer plainly):
   ```
   gh repo view runwithit-hq/brain --json viewerPermission --jq .viewerPermission
   gh repo view runwithit-hq/clients --json viewerPermission --jq .viewerPermission
   ```
   `WRITE`, `MAINTAIN` or `ADMIN` on both is right. `READ` on either, or an error, means Luis hasn't finished their access: they send him the output and wait.
2. Install the team's skills:
   ```
   claude plugin marketplace add runwithit-hq/brain
   claude plugin install rwi@rwi
   claude plugin install rwi-voices@rwi
   claude plugin install rwi-playbook@rwi
   claude plugin list
   ```
   The list shows `rwi`, `rwi-voices` and `rwi-playbook`. If `marketplace add` says the repository wasn't found or asks for a username, run `gh auth setup-git` again and retry once.
3. Tell them: "You have the team's skills. `rwi-labs` (experimental skills) is optional; skip it for now."

## 5. Hand over

1. They quit Claude Code completely and reopen it: new skills load when a session starts.
2. In the new session they type `/rwi:teammate-setup`. That skill copies the client folders to their Mac, sets up the folders and safety checks, lists the access only Luis can give, and does a check-in that Luis can see from his Mac.
3. Finish with a short table: tools ✓, GitHub `<login>` ✓, runwithit-hq ✓, brain access ✓, skills installed ✓, next: `/rwi:teammate-setup` in a new session.

## When something goes wrong

| Symptom | Fix |
|---|---|
| `command not found: brew` right after installing it | They didn't run Homebrew's "Next steps" lines, or Claude Code was open during the install. Run the lines, reopen Claude Code. |
| `gh auth login` hangs | The browser page wasn't completed. Run it again in Terminal and finish the Authorize click. |
| `Repository not found` on `marketplace add` | Not in the org yet or the invite isn't accepted (step 3), or `gh auth setup-git` wasn't run (step 2.4). |
| `Permission denied (publickey)` | Something used SSH. Run `gh auth login` again and pick HTTPS. |
| `claude plugin` says the plugin doesn't exist | The marketplace add failed. Run `claude plugin marketplace list`; `rwi` must be there. |
