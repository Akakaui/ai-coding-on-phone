# Code From Your Phone
## The Complete Guide to Building Real Apps Without a Laptop

> This guide shows you how to set up a professional AI-powered development environment using only your phone. No laptop required. No expensive subscriptions. Just your phone, a free server, and an AI that codes for you.

---

# BEFORE YOU START

## What You Are Building

```
Your Phone (Termius)
      ↕ SSH
Oracle VPS — your free permanent server
      ↓
   GitHub CLI → manage all your repos and codespaces
      ↓
GitHub Codespace — powerful cloud computer (4 cores, 8GB RAM)
      ↓
   Gemini CLI → your AI coding agent
   Your Project → Next.js, React, Python, anything
```

**The idea is simple:**
- Oracle VPS is your permanent gateway — always on, always yours, costs nothing
- GitHub Codespace is the powerful computer that runs your code
- Gemini CLI is the AI that writes, fixes, and runs code for you
- Termius is the app on your phone that connects everything

## What You Need

- An Android or iOS phone
- A Google account (for Gemini)
- A GitHub account (free at github.com)
- An Oracle account (free at cloud.oracle.com)
- The Termius app (free on Play Store or App Store)

## Honest Limitations

- GitHub free accounts get **60 hours of Codespace time per month**. GitHub Pro gives 180 hours for $4/month. Plan your usage accordingly.
- Gemini CLI is **free** with a Google account — 1,000 requests per day
- This guide uses Gemini CLI but the same setup works with **Claude Code, Aider, or any other AI CLI tool**

---

# PART 1 — GET YOUR FREE SERVER (ORACLE VPS)

Oracle Cloud gives you a free server that runs 24/7 forever. No charges as long as you stay on the free tier.

## 1.1 — Create Your Oracle Account

1. Go to **cloud.oracle.com**
2. Click **Start for free**
3. Fill in your name, email, country
4. Add card details — this is for identity verification only, you will not be charged
5. Complete the sign up and verify your email

## 1.2 — Create Your Free Server

1. Log into cloud.oracle.com
2. Click the menu (top left) → **Compute** → **Instances**
3. Click **Create Instance**
4. Configure it like this:

```
Name:       dev-server
Image:      Ubuntu 22.04 LTS
Shape:      VM.Standard.E2.1.Micro  ← this is the always-free one
OCPUs:      1
RAM:        1 GB
Storage:    47 GB
Public IP:  Enabled ← make sure this is ON
```

5. Under **SSH Keys** → click **Generate SSH Key Pair**
6. Download both the private key and public key to your phone
7. Click **Create**
8. Wait 2-3 minutes until status shows **Running**
9. Copy your **Public IP Address** — you will need this

> **Save your IP address and private key somewhere safe. You will need them every time.**

## 1.3 — Open the SSH Port

By default Oracle blocks incoming connections. Fix this:

1. OCI Menu → **Networking** → **Virtual Cloud Networks**
2. Click your VCN → **Security Lists** → **Default Security List**
3. Click **Add Ingress Rules**
4. Fill in:

```
Source CIDR:      0.0.0.0/0
IP Protocol:      TCP
Destination Port: 22
Description:      SSH Access
```

5. Click **Add Ingress Rules** to save

---

# PART 2 — CONNECT FROM YOUR PHONE (TERMIUS)

## 2.1 — Install Termius

- **Android:** Play Store → search **Termius** → Install
- **iOS:** App Store → search **Termius** → Install

Termius is a professional SSH client. The free version is everything you need.

## 2.2 — Add Your VPS to Termius

1. Open Termius
2. Tap **+** → **New Host**
3. Fill in:

```
Label:    Oracle VPS
Hostname: YOUR-ORACLE-IP
Port:     22
Username: ubuntu
```

4. Tap **SSH Keys** → **Add Key**
5. Paste your Oracle private key or import the file
6. Tap **Save**

## 2.3 — Connect to Your VPS

Tap your saved **Oracle VPS** host. You should see a terminal with:

```
ubuntu@your-instance:~$
```

You are now inside your free server. Every time you want to work, just open Termius and tap your host.

---

# PART 3 — SET UP YOUR VPS

Do this once after connecting for the first time.

## 3.1 — Update the Server

```bash
sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y
```

Wait for it to finish.

## 3.2 — Install GitHub CLI

Copy and run these commands one by one:

```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list

sudo apt update && sudo apt install gh -y
```

Verify it worked:

```bash
gh --version
```

You should see something like `gh version 2.x.x`

## 3.3 — Authenticate GitHub

```bash
gh auth login
```

Follow the prompts:
- **Where:** GitHub.com
- **Protocol:** SSH
- **Generate SSH key:** Yes
- **Key name:** press Enter (keep default)
- **Passphrase:** press Enter (leave empty)
- **Auth method:** Login with a web browser

It will show you a code like `XXXX-XXXX`. Copy it.

Open **github.com/login/device** in your phone browser, paste the code, click **Authorize**.

Come back to Termius — you should see `Logged in as YourUsername`.

```bash
# Add codespace permissions
gh auth refresh -h github.com -s codespace
```

It will ask you to authorize again — same process, copy code, open browser, paste, authorize.

Verify everything worked:

```bash
gh auth status
```

You should see your GitHub username and `Active account: true`

## 3.4 — Keep Sessions Alive with tmux

Without tmux, if your phone disconnects your session dies. tmux keeps everything running.

```bash
sudo apt install tmux -y
tmux new -s main
```

You are now inside a tmux session. If you ever disconnect, reconnect to Termius and run:

```bash
tmux attach -t main
```

Everything will be exactly where you left it.

> **tmux tip:** Press `Ctrl+B then D` to detach from the session without closing it. Type `tmux attach -t main` to come back.

---

# PART 4 — YOUR FIRST CODESPACE

A GitHub Codespace is a full Linux computer running in the cloud. It has 4 CPU cores, 8GB RAM, and 32GB storage. This is where your actual code runs.

## 4.1 — Create a Codespace on GitHub

1. Go to **github.com** on your phone browser
2. Open any of your repositories
3. Click the green **Code** button
4. Click **Codespaces** tab
5. Click **Create codespace on main**
6. Wait for it to load — takes about 1 minute
7. Once open, copy the URL from your browser

The URL will look like this:
```
https://fuzzy-fortnight-69rp496gpqwg35qx9.github.dev
```

The codespace name is everything after `https://` and before `.github.dev`:
```
fuzzy-fortnight-69rp496gpqwg35qx9
```

## 4.2 — SSH Into Your Codespace From VPS

Back in Termius, in your VPS terminal:

```bash
gh codespace ssh --codespace fuzzy-fortnight-69rp496gpqwg35qx9
```

Replace `fuzzy-fortnight-69rp496gpqwg35qx9` with your actual codespace name.

You should see:

```
@YourUsername ➜ /workspaces/your-project (main) $
```

You are now inside your Codespace — a powerful cloud computer — from your phone.

## 4.3 — Useful Codespace Commands

```bash
# List all your codespaces
gh codespace list

# Start a stopped codespace
gh codespace start --codespace <name>

# Stop a running codespace (saves your hours)
gh codespace stop --codespace <name>

# Create a new codespace for a repo
gh codespace create --repo YourUsername/repo-name

# Delete a codespace you no longer need
gh codespace delete --codespace <name>
```

---

# PART 5 — GEMINI CLI (YOUR AI CODING AGENT)

Gemini CLI is Google's AI coding tool. It reads your entire codebase, writes code, runs commands, and commits to git — all from the terminal. It is free with a Google account.

## 5.1 — Install Gemini CLI

Inside your Codespace terminal:

```bash
npm install -g @google/gemini-cli
```

Verify:

```bash
gemini --version
```

## 5.2 — Authenticate Gemini

### Option A — Sign in with Google (Recommended, Free)

```bash
gemini
```

On first launch select **Sign in with Google**. It will show you a URL. Open it on your phone browser, sign in with your Google account, authorize. Done.

You get **1,000 free requests per day** with Gemini 2.5 Pro — their best model.

### Option B — Use an API Key

If you have a Gemini API key from **aistudio.google.com**:

```bash
nano ~/.env
```

Add:
```
GEMINI_API_KEY=your-key-here
```

Save with `Ctrl+X → Y → Enter` then:

```bash
source ~/.env
gemini
```

### Option C — Use OpenRouter (Access Any Model)

OpenRouter gives you access to Claude, GPT-4, Deepseek, Qwen, and more with one API key. Get a key at **openrouter.ai**

Works with Aider:

```bash
pip install aider-chat

aider --openai-api-base https://openrouter.ai/api/v1 \
      --openai-api-key your-openrouter-key \
      --model openrouter/anthropic/claude-3.5-sonnet
```

## 5.3 — Switch Models or Auth Inside Gemini

```bash
# Switch auth method
/auth

# Switch model
/model

# See all commands
/help
```

## 5.4 — Start Building

```bash
cd /workspaces/your-project
gemini
```

Talk to it like you would talk to a person:

```
build me a login page with email and password using Supabase auth

fix the TypeScript error in src/app/api/users/route.ts

add rate limiting to all my API routes

create a dashboard that shows revenue, active clients, and pending proposals

commit all changes with a meaningful message and push to GitHub
```

Gemini reads your files, writes the code, runs the commands, and commits — all by itself.

---

# PART 6 — DOTFILES (AUTO-SETUP FOR EVERY CODESPACE)

Every time you create a new Codespace you would have to install Gemini CLI again. Dotfiles solves this — it runs a setup script automatically on every new Codespace you create.

## 6.1 — Create the Dotfiles Repo

On GitHub, create a new repository called exactly **dotfiles** — the name matters.

Or from your VPS terminal:

```bash
gh repo create dotfiles --public
mkdir ~/dotfiles && cd ~/dotfiles
```

## 6.2 — Create the Setup Script

```bash
nano install.sh
```

Paste this:

```bash
#!/bin/bash

echo "Setting up Codespace..."

# Install Gemini CLI
npm install -g @google/gemini-cli

# Create gitignore so secrets never get committed
cat > ~/.gitignore_global << EOF
.env
.env.*
.gemini/
*.local
EOF
git config --global core.excludesfile ~/.gitignore_global

# Create empty env file for your API keys
cat > ~/.env << EOF
GEMINI_API_KEY=
OPENROUTER_API_KEY=
EOF

# Auto load env on every session
echo "source ~/.env" >> ~/.bashrc

echo "Done. Run: nano ~/.env to add your keys"
```

Save with `Ctrl+X → Y → Enter`

```bash
chmod +x install.sh
git init
git add .
git commit -m "dotfiles setup"
git remote add origin git@github.com:YourUsername/dotfiles.git
git branch -M main
git push -u origin main
```

## 6.3 — Enable Dotfiles on GitHub

1. Go to **github.com** on your phone
2. **Settings** → **Codespaces**
3. Under **Dotfiles** → toggle **Automatically install dotfiles**
4. Select your **dotfiles** repository
5. Install script: **install.sh**

Every new Codespace you create from now will automatically install Gemini CLI and set up your environment.

---

# PART 7 — CUSTOM AI INSTRUCTIONS

You can give Gemini a set of rules and capabilities so it behaves exactly how you want. This is like giving your AI a job description.

## 7.1 — Create Instructions for Your Codespace

Inside your project create a file called `GEMINI.md` (run this command in your Codespace terminal):

```bash
nano GEMINI.md
```

Example:

```markdown
# Project Instructions

## Stack
- Next.js 14 App Router
- Supabase for database and auth
- TypeScript strict mode
- Tailwind CSS
- Resend for emails
- Upstash for rate limiting

## Rules
- Always write TypeScript, never plain JavaScript
- Always handle errors with try/catch
- Always add RLS policies when creating Supabase tables
- Never use placeholder comments like "add logic here"
- Always commit after completing a feature
- Keep components small and reusable

## What You Can Do
- Read and write any file in this project
- Run terminal commands
- Install npm packages
- Use git to commit and push
- Run the dev server to test changes

## What You Should Not Do
- Never delete the .env file
- Never commit API keys or secrets
- Never modify the database schema without confirming first
```

Gemini reads this file automatically when you start it in the project folder.

## 7.2 — Advanced Example: Browser Automation

You can install tools inside the Codespace and have Gemini use them:

```bash
# Install Chromium browser inside codespace
sudo apt install chromium-browser -y
```

Then tell Gemini:

```
open Chromium, navigate to localhost:3000, take a screenshot of
the dashboard page and tell me if anything looks broken
```

Gemini will launch the browser, navigate, screenshot, and report back.

When you are done:

```
uninstall Chromium, we no longer need it
```

It cleans up after itself.

## 7.3 — MCP Tools (Connect AI to External Apps)

MCP (Model Context Protocol) lets your AI connect to and interact with external tools and services. You can connect Gemini to:

- **GitHub** — manage issues, PRs, repos
- **Supabase** — query and update your database
- **Notion** — read and write docs
- **Slack** — send messages
- **Browser** — navigate websites
- And many more

Add MCP servers to your dotfiles so they are available in every Codespace. When a new Codespace is created, the dotfiles' setup script (install.sh) will automatically install these MCP servers. The agent (Gemini CLI) runs *within* the Codespace and uses the tools installed by dotfiles. To change or add MCP servers, you would update your `dotfiles` repository on GitHub and commit the changes.

```bash
# In your dotfiles install.sh, add:
cat > ~/.gemini/mcp.json << EOF
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": ""
      }
    }
  }
}
EOF
```

Fill in your tokens in `~/.env` — they are gitignored so they never get committed.

---

# PART 8 — YOUR DAILY WORKFLOW

## Starting a Session

1. Open **Termius** on your phone
2. Tap **Oracle VPS**
3. Run: `tmux attach -t main`
4. Run: `gh codespace ssh --codespace <your-codespace-name>`
5. Run: `source ~/.env`
6. Run: `cd /workspaces/your-project`
7. Run: `gemini`
8. Start building

## Switching Projects

```bash
# Exit current codespace
exit

# List all codespaces
gh codespace list

# SSH into a different one
gh codespace ssh --codespace <other-name>

# Or create a new one
gh codespace create --repo YourUsername/new-project
```

## Uploading Files from Your Phone to Your Development Environment

Getting files from your phone to where your code runs (your Codespace) involves a couple of steps, but it's straightforward once you know how. Think of your Oracle VPS as a temporary holding spot for files coming from your phone before they reach your main development environment in the Codespace.

**Step 1: Get Files from Your Phone to Your Oracle VPS (Using Termius)**

First, you need to transfer files from your phone to your always-on Oracle VPS.

1.  **Open Termius:** Launch the Termius app on your phone.
2.  **Connect to your VPS:** Tap on your saved **Oracle VPS** host to establish a connection.
3.  **Access File Manager:** Look for a **Files** tab or button within Termius and tap it. This is Termius's built-in tool for managing files.
4.  **Upload from Phone:** Tap **Upload** and then select the file(s) you want to send from your phone's storage.
5.  **File on VPS:** These files will now appear in your Oracle VPS's home directory (e.g., `/home/ubuntu/`).

**Step 2: Move Files from Your Oracle VPS into Your Codespace**

Once a file is on your Oracle VPS (either uploaded from your phone or created there), you can easily move it into your active Codespace. Think of the `gh codespace cp` command as a special 'copy-paste' tool that works between your VPS and your Codespace. You run this command from your VPS terminal.

*   **Example:** Let's say you uploaded `my_report.pdf` to your VPS's home directory (`~/`), and you want to put it into your project's `documents` folder inside your Codespace (`/workspaces/your-project/documents/`).
    *   You would run this command in your **VPS terminal**:
        ```bash
        gh codespace cp ~/my_report.pdf remote:/workspaces/your-project/documents/
        ```
    *   `~/my_report.pdf`: This is the file on your VPS.
    *   `remote:/workspaces/your-project/documents/`: This is the destination path inside your Codespace.

**Step 3: Organize Files within Your Codespace (Using Gemini CLI)**

Once your files are in the Codespace, you can use Gemini CLI to help manage them.

*   **Example:** If you have a file named `design.jpg` in your Codespace's home directory (`~`) and you want to move it to the `/assets/images/` folder within your project:
    *   You can tell Gemini:
        ```
        move the file design.jpg from my home directory (~/) to /workspaces/your-project/assets/images/
        ```
    *   Gemini will then execute the necessary commands within the Codespace to move the file for you.

This layered approach ensures you can efficiently get your files from your phone into your powerful cloud development environment.

## Saving Codespace Hours

The free tier gives you 60 hours per month. Make them count:

```bash
# Always stop your codespace when done
gh codespace stop --codespace <name>

# Start it again when you need it
gh codespace start --codespace <name>
```

A stopped codespace uses no hours. Your files and everything you installed stays intact.

## Saving Codespace Hours

The free tier gives you 60 hours per month. Make them count:

```bash
# Always stop your codespace when done
gh codespace stop --codespace <name>

# Start it again when you need it
gh codespace start --codespace <name>
```

A stopped codespace uses no hours. Your files and everything you installed stays intact.

---

# QUICK REFERENCE

## Essential Commands

```bash
# Connect to VPS from Termius
# Just tap your saved host

# Reattach session after disconnect
tmux attach -t main

# List all codespaces
gh codespace list

# SSH into codespace
gh codespace ssh --codespace <name>

# Stop codespace (save hours)
gh codespace stop --codespace <name>

# Create new repo
gh repo create project-name --private

# Create new codespace
gh codespace create --repo YourUsername/project-name

# Install Gemini CLI
npm install -g @google/gemini-cli

# Start Gemini
gemini
```

## What Repeats vs What Doesn't

| Task | How Often |
|---|---|
| Add VPS to Termius | Once ever |
| Install gh CLI | Once ever |
| Authenticate GitHub | Once ever |
| Set up dotfiles | Once ever |
| Install Gemini in codespace | Auto via dotfiles |
| Add API key to ~/.env | Once per codespace |
| tmux attach | Every session |
| gh codespace ssh | Every session |

---

# WORKS WITH OTHER AI TOOLS

This setup is not limited to Gemini. The same infrastructure works with:

- **Claude Code** — Anthropic's AI coding agent
- **Aider** — open source, supports any model via OpenRouter
- **Continue** — model agnostic AI coding tool
- **Cline** — VS Code extension (use via browser in codespace)

The Codespace is just a Linux computer. Any CLI tool that runs on Linux works here.

---

*This guide was built and tested on a phone. Everything in it works.*

*The setup costs $0 to start. The only cost is GitHub Pro ($4/month) if you need more than 60 codespace hours per month.*
