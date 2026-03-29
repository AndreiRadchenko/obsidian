---
date_created: 2025-09-28
date_modified: 2025-09-28
document_type: resource
tags:
  - resource
  - domain/software-dev
  - "#git"
---
[[Resources/Resources Dashboard|Resources]] / **[[Resources/Git help|Git help]]**
# Git help
**Overview**
Description:: Git command help

## Git commands:

- Create a new repository on the command line:

        echo "# WebMicGain" >> README.md
        git init
        git add README.md
        git commit -m "first commit"
        git remote add origin git@github.com:AndreiRadchenko/WebMicGain.git
        git push -u origin master

- Start your own project from cloned repo:

        git clone https://github.com/...somerepo.git // clone to current folder
        git remote set-url origin <url> // url to your remote repo

### Remote repository:

<details><summary> Git commands</summary>

- Show `origin` repository:

        git remote -v

- View pc ssh key for copy to github:

        cat ~/.ssh/id_rsa.pub

- Clone repo from github:

        git clone https://github.com/...somerepo.git path/to/local/folder
        git clone https://github.com/...somerepo.git // clone to current folder

- Link local repository with empty repo on github:

        git remote add origin git@github.com:AndreiRadchenko/RF-Gate.git

- Substitute `origin` repository with `<url>`:

        git remote set-url origin <url>

- **Fetch** all changes from remote to local:

        git fetch

- **Pull**: fetch + merge from remote to the current local branch :

        git pull

- **Push** new branch to the github:

        git push origin feat/new-feature
        git push -u origin feat/new-feature //set upstream branch

- **Delete** branch on the remote repo:

        git push origin --delete branch/to-delete

</details>

### Git local folder

<details><summary>Git Commands</summary>

- Add version control to the folder:

        git init

- Remove version control from folder (run this command in folder with git):

        rm -rf .git

- Add files and folders to local git:

        git add FreeRTOS/*
        git add readme.md
        git add . //add changes in all files

- View files under git:

        git ls-files

- View current changes from last commit:

        git status
        git diff

- Show branches:

        git branch    // local branches
        git branch -r // remote branches
        git branch -a // all branches

- Show commit history

        git log
        git log --pretty=oneline
        git log --oneline
        git log master

- Commit changes to current branch:

        git commit -a -m "fix: change max load to 3A"

- Apply changes to existed commit

        git commit --amend

- **Create new branch** and switch to this branch:

        git checkout -b current_sensors

- Switch to branch <current_sensors>:

        git checkout current_sensors

- Delete local branch:

        git branch -d branch/name

- **Merge** `branch_name` into current local branch

        git merge branch_name
        git merge origin/branch_name

- Rename current branch:

        git branch -m new/branch-name

- Revert particular commit changes and create new commit. (adds a new commit that undoes its changes. )

        git revert b1c2345

- Reset to previous commit without saving changes:

        git reset --hard HEAD~1

- Reset to 2 commit with saving changes (you can create new commit with staged changes):

        git reset --soft HEAD~2
        git reset --soft b1c2345

</details>

### Git stash

<details><summary>Git Command</summary>

- Temporally save changes that are not ready to be committed (cleared working tree):

        git stash

- Show all stash list:

        git stash list

- Apply last saved stash to the current branch:

        git stash apply

- Apply particular stash:

        git stash apply stash@{1}

- Apply last saved stash and remove it from stash list:

        git stash pop
        git stash pop stash@{1}

- Remove particular stash from stash list:

        git stash drop stash@{1}

- Delete all stashes:

        git stash clear

</details>

### Git cherry-pick

<details><summary>Git Command</summary>

Cherry-pick is used to apply particular changes only from one branch to the other. Unlike merge, apply all changes.

- Create branch for test features:

        git checkout master
        git pull
        git checkout -b deploy/15.05.2024-front

- Apply certain commit to current branch:

        git cherry-pick c7fdcf1

- After resolve conflicts (if emerge)

        git add .
        git cherry-pick --continue

- Push test branch to github and create pull request:

        git push origin deploy/15.05.2024-front
        git push -u origin deploy/15.05.2024-front

</details>

### Github authenticate setup

<details><summary>Git Command</summary>

After github access token has been updated, you need to run next command in bash:

- Clear previous credentials:

        git config --global credential.helper ""

- Perform some operation with remote repo:

        git push origin master

- On credential request enter `name` and `pass`:

        name: andreiradchenko
        pass: api-token

- Run command:

        git config credential.helper store

</details>

---

# Git Commands Summary - Home Assistant Config Repo

**Purpose**: Clean Home Assistant config repo from leaked secrets (Google credentials, SSH keys, Amazon OAuth) that blocked GitHub pushes due to push protection, while making `.gitignore` properly exclude sensitive files.

**Problem Solved**:
- GitHub push blocked: `GH013: Repository rule violations found` (secrets in commit history)
- `.gitignore` not working (tracked secrets.yaml, SERVICE_ACCOUNT.json, .ssh/, databases)
- Maintain config files while removing all sensitive data from git

***

### Add new pc to github 

1. Create ssh key on local pc

```bush
ssh-keygen -t ed25519 -C "d4100kx@gmail.com"
```
2. Add key to the github ssh keys
```bush
cat ~/.ssh/id_ed25519.pub
```
3. Change https github access to ssh
```bush
ssh -T git@github.com
git remote set-url origin git@github.com:AndreiRadchenko/Homeassistant/
git remote -v
git push --set-upstream origin master
```


## 1. Check Current State

```bash
git ls-files                          # List all tracked files
git status                            # Show staged/unstaged files
git check-ignore -v secrets.yaml      # Verify ignore rule works
git log --oneline                     # View commit history
```

## 2. Fix .gitignore (Corrected Version)

```gitignore
# Folders
.storage/ .cloud/ .ssh/ tts/ custom_components/ www/ image/ deps/ scans/ blueprints/

# Databases
*.db *.db-shm *.db-wal home-assistant_v2.db* zigbee.db*

# Logs
*.log *.log.* home-assistant.log*

# Secrets
secrets.yaml ip_bans.yaml known_devices.yaml SERVICE_ACCOUNT.json *.json .google.token .HA_VERSION .ha_run.lock .shopping_list.json

# Keep
!.gitignore !README.md !Readme.md !configuration.yaml !automations.yaml !scripts.yaml !scenes.yaml !cameras.yaml !covers.yaml !templates.yaml !templates-old.yaml
```

## 3. Apply .gitignore to Already Tracked Files

```bash
git rm -r --cached .    # Remove ALL from tracking (files stay on disk)
git add .               # Re-add according to .gitignore
git status              # Verify
git commit -m "Apply .gitignore"
```

## 4. Remove Secrets from History (Fresh Start)

```bash
# Create orphan branch (no history)
git checkout --orphan fresh-start

# Stage clean files
git add .

# Verify clean staging
git status
git ls-files | grep -E "secrets|SERVICE|\.ssh|\.db|\.log"

# Commit clean state
git commit -m "Initial commit - clean history"

# Replace main
git branch -D main
git branch -m main

# Force push to GitHub
git push -f origin main
```

## 5. Verify Success

```bash
git log --oneline                    # 1 commit only
git ls-files | wc -l                 # Reasonable file count
git ls-files | grep secrets           # Should be empty
```

## 6. Pre-Commit Checklist

```bash
git status                           # Check staged files
git diff --cached                    # Review changes
git ls-files | grep -E "secrets|db|log|ssh"  # No secrets
```

***

**Key Points:**
- `.gitignore` only works on **untracked** files
- Use `git rm --cached` to untrack files without deleting them  
- Orphan branch (`--orphan`) = fresh history with current files
- `git push -f` overwrites GitHub history (coordinate with collaborators)
- **Rotate all leaked credentials** after cleanup (Google keys, SSH keys, Amazon OAuth)

Sources




---
[[Resources/Resources Dashboard|Resources]] / **[[Resources/Git help|Git help]]**