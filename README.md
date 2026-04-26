# The `set-up-django-gdpr-cookie-consent` Skill

This guide explains how to install the **set-up-django-gdpr-cookie-consent** skill into any AI coding assistant that supports Claude skills (Claude Code, Augment Code, Cursor, and others).

## Why Use This Skill?

Setting up GDPR-compliant cookie consent in Django involves multiple steps — installing the right package, wiring up templates, configuring cookie categories, and ensuring tracking scripts only load after user consent. Done manually, this is time-consuming and easy to get wrong. This skill automates the entire process: the AI follows a precise, tested workflow that correctly installs and configures [django-gdpr-cookie-consent](https://websightful.gumroad.com/l/django-gdpr-cookie-consent) in your project in minutes, not hours. You get a fully functional cookie consent banner, properly categorised cookies, and GDPR-safe script loading — without having to read through documentation or debug integration issues yourself.

## What is a Skill?

A skill is a folder containing a `SKILL.md` file (plus optional scripts, references, and assets) that teaches your AI editor how to handle a specific task — in this case, setting up GDPR-compliant cookie consent in a Django project.

## Prerequisites

- An AI editor with Claude skills support (Claude Code, Augment Code, etc.)
- [Git](https://git-scm.com/) installed
- [django-gdpr-cookie-consent package](https://websightful.gumroad.com/l/django-gdpr-cookie-consent)

## Installation Steps

### 1. Locate your skills directory

Skills are typically stored in a `.claude/skills/` or `.ai/skills/` folder, either in your **home directory** (global) or in your **project root** (project-local).

Common locations:

| Scope          | Path                                      |
|----------------|-------------------------------------------|
| Global (user)  | `~/.claude/skills/`                       |
| Project-local  | `<your-project>/.claude/skills/`          |

> Check your editor's documentation if you're unsure which path it uses.

### 2. Copy the skill to your skills directory

Check out (or download and unzip) the repo and copy the skill to your `skills` directory:  

```bash
$ git clone https://github.com/websightful/set-up-django-gdpr-cookie-consent.git \
~/projects/django-gdpr-cookie-consent-skill
$ cp -r ~/projects/django-gdpr-cookie-consent-skill/set-up-django-gdpr-cookie-consent \
~/.claude/skills/set-up-django-gdpr-cookie-consent
```

The result should be a folder named `set-up-django-gdpr-cookie-consent` containing a `SKILL.md` file:

```
.claude/skills/
└── set-up-django-gdpr-cookie-consent/
    ├── SKILL.md
    └── ... (any additional scripts or reference files)
```

### 3. Verify the structure

Make sure the `SKILL.md` file is directly inside the cloned folder:

```bash
# Should print the SKILL.md path
$ ls ~/.claude/skills/set-up-django-gdpr-cookie-consent/SKILL.md
```

### 4. Restart your AI editor (if required)

Some editors pick up new skills automatically; others require a restart or a settings reload. Restart your editor to be safe.

## Verifying the Skill is Active

Open a new chat in your AI editor and try a prompt like:

> "Set up GDPR cookie consent in my Django project."

or

> /set-up-django-gdpr-cookie-consent

If the skill is loaded correctly, the AI will follow the structured workflow defined in `SKILL.md` rather than giving a generic answer.

## Troubleshooting

| Problem | Fix |
|---|---|
| Skill not triggering | Confirm the folder name matches exactly: `set-up-django-gdpr-cookie-consent` |
| "Skill not found" error | Check the skills directory path — some editors use a different default |
| Extra nesting after clone | Move the inner folder up one level so `SKILL.md` is directly inside the skill folder |

## Uninstalling

Simply delete the skill folder:

```bash
$ rm -rf ~/.claude/skills/set-up-django-gdpr-cookie-consent
```
