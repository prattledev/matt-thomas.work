---
date: 2026-04-02T01:00:00Z
draft: false
categories:
  - Broadcast Technology
tags:
  - Tools
  - Automation
  - Software
  - Audio Engineering
title: "Git Basics for Audio Engineers"
description: "As broadcast audio moves toward software-defined workflows, the tools we build and configure deserve the same rigour we apply to signal chains. Git is how you stop losing work, track changes, and collaborate on the scripts and configs that keep productions running."
---

Broadcast audio engineering has always involved a degree of systems thinking. Routing matrices, gain structures, signal flow - these are disciplines built around understanding how components interact and what happens when something changes. Software-defined workflows extend that thinking into a new domain, and with it comes a new category of things that can go wrong: a script that worked last week stops working, a configuration change breaks something, a colleague overwrites your work.

Git is the tool the software world uses to manage this. It is not complicated at its core, and the investment in learning it pays back quickly once you are maintaining anything more complex than a single file.

## Why Git Matters for Audio Engineers

The shift toward software-defined broadcast infrastructure means audio engineers are increasingly responsible for things that look like software. Routing automation scripts. Python tools that query NMOS registries or calculate stream bandwidth. SDP files for AES67 receivers/senders or SRT contribution encoders. Macros and show files for consoles that support scripting.

All of these share a characteristic with code: they change over time, multiple people may work on them, and a bad change can break something in production. Without version control, the typical response is to keep copies - `routing_v2_FINAL.py`, `routing_v2_FINAL_actually_final.py` - which solves nothing and creates its own problems.

Git gives you a structured history of every change, the ability to go back to any previous state, a safe way to experiment without risking the working version, and a mechanism for collaboration that does not involve emailing files around.

## How Git Works

Git tracks changes to files in a **repository** - a directory that Git is managing. Every time you want to save a snapshot of your work, you create a **commit**. A commit is a permanent record of what changed, when, and why. The collection of commits is the history of the project.

You do not have to commit every small change. The discipline is to commit when you have reached a meaningful state - a working version of a new feature, a fix for a specific bug, a configuration update that has been tested.

Git is **distributed**, which means every copy of a repository contains the full history. There is no single server that holds the authoritative version - though in practice most teams designate one (typically GitHub, GitLab, or a self-hosted equivalent) as the shared remote.

## Basic Commands

These are the commands you will use most of the time.

**Initialise a repository**

```bash
git init
```

Run this once in a directory to start tracking it with Git. Creates a hidden `.git` folder that stores the history.

**Check what has changed**

```bash
git status
```

Shows which files have been modified, which are new and untracked, and which are staged and ready to commit. Run this frequently - it is your source of truth for what Git knows about.

**Stage changes for commit**

```bash
git add filename.py
git add .
```

Staging is a deliberate step between making a change and committing it. `git add filename.py` stages a specific file. `git add .` stages everything in the current directory. Being selective about what you stage lets you make clean, focused commits.

**Commit staged changes**

```bash
git commit -m "Add gain compensation to downmix script"
```

The message should describe what changed and why, not what the code does. "Fix centre channel level calculation" is useful. "Update script" is not. You will thank yourself for good commit messages when you are trying to find where something broke six months later.

**View the history**

```bash
git log
git log --oneline
```

`git log` shows the full commit history with author, date, and message. `--oneline` gives a compact one-line-per-commit view, which is easier to scan.

**See what changed in a file**

```bash
git diff
git diff filename.py
```

Shows the line-by-line difference between the current state and the last commit. Useful before staging to review what you are about to commit.

**Undo changes to a file**

```bash
git restore filename.py
```

Discards uncommitted changes to a file and restores it to the last committed state. Useful when an experiment has gone wrong and you want to start over.

**Connect to a remote repository**

```bash
git remote add origin https://github.com/yourname/yourrepo.git
```

Adds a remote called `origin` - the conventional name for the primary remote. You only need to do this once per repository.

**Push commits to the remote**

```bash
git push origin main
```

Sends your local commits to the remote repository. If the remote does not have the branch yet, you may need `git push -u origin main` the first time to set the tracking relationship.

**Pull changes from the remote**

```bash
git pull
```

Fetches commits from the remote and merges them into your local branch. Run this before starting work if others may have pushed changes.

**Clone an existing repository**

```bash
git clone https://github.com/yourname/yourrepo.git
```

Creates a local copy of a remote repository, including its full history. The starting point when working with an existing project.

## Branches

Branches let you work on something new without touching the main working version. The convention is that `main` (or `master` in older repositories) always contains working, tested code. New work happens on a separate branch and is merged back when it is ready.

```bash
git branch feature/add-srt-output    # create a branch
git checkout feature/add-srt-output  # switch to it
```

Or in one step:

```bash
git checkout -b feature/add-srt-output
```

When the work is done and tested:

```bash
git checkout main
git merge feature/add-srt-output
```

For audio engineering work, branches are useful any time you are making a significant change to something that is actively in use. Develop the new version on a branch, test it, then merge it. The main branch always has the version you can go back to if the show is about to start and something is wrong.

## What to Put in Git

For broadcast audio workflows, good candidates include:

- Python or Bash scripts for automation and monitoring
- NMOS configuration and registration tools
- SDP file templates
- Console macro sets and show file templates (where the format is text-based)
- Documentation for systems and signal flow

What not to put in Git: large binary files, credentials and API keys, output files generated by scripts. A `.gitignore` file in the repository root tells Git which files and directories to ignore.

```
# Example .gitignore
*.log
credentials.json
output/
```

## A Practical Starting Point

If you have a directory of scripts or configuration files that you are currently managing manually, the path to getting them into Git is straightforward:

```bash
cd /path/to/your/scripts
git init
git add .
git commit -m "Initial commit - existing scripts"
```

Create a repository on GitHub, connect it, and push:

```bash
git remote add origin https://github.com/yourname/yourrepo.git
git push -u origin main
```

From that point, every change you make is tracked. You can always see what changed, when, and go back to any previous state. For tools that production workflows depend on, that is a meaningful improvement over hoping the last working version is somewhere on the network drive.
