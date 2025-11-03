# How I vibe - a software engineers take on agentic AI development

As software engineers, we're used to a systematic approach to development.

We're breaking software down into epics and issues, or whatever you call it in your methodoligy.

This repo is part of a series of articles and videos, where I explain my approach.

## Getting Started

This repository uses Git submodules. Follow one of the methods below to get the complete repository with all submodules.

### Option 1: Clone with submodules (recommended)

```bash
git clone --recurse-submodules git@github.com:codewithpassion/how-i-vibe.git
cd how-i-vibe
```

### Option 2: Clone first, then fetch submodules

```bash
git clone git@github.com:codewithpassion/how-i-vibe.git
cd how-i-vibe
git submodule update --init --recursive
```

### Option 3: Update submodules in an existing clone

If you've already cloned the repo without submodules, run:

```bash
git submodule update --init --recursive
```

## Coding agents

These set of prompts are generally applicaple for most AI coding agents and environments. 

Although, I'm generally using `claude-code`.

In subsequent modules, I will use more `claude-code` specific things, like sub-agents and skills. 

## 01-manual

This is all about the basics. 
We take a very short project brief `worktracking-project.md` and create a deatailed project documentation prompt (PRD). 
Then we break it down into phases, and into tasks. 

These tasks then can be implemented using an AI coding agent.