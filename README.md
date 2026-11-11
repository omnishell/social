# Omni Social Media Manager (`social`)

# About

**`social`** is a social media content, template, and publishing manager for the [Omni Shell Framework](../omni/README.md). It enables drafting, organizing, previewing, and publishing posts from Markdown files with YAML frontmatter directly to **LinkedIn** (supporting both personal profiles and company/organization pages).

## Key Features

- **Full LinkedIn API Integration (`mods/linkedin`)**:
  - Interactive **OAuth 2.0 Authorization Code flow** with a built-in local HTTP callback listener (`nc`) that automatically exchanges authorization codes for access tokens.
  - Supports text posts, **article/link shares**, and **image/thumbnail uploads** (automatic asset initialization, binary upload, and URN attachment).
  - Supports publishing as a personal profile (`urn:li:person:...`) or an organization/company page (`urn:li:organization:...`) with visibility controls (`PUBLIC`, `CONNECTIONS`, `LOGGED_IN`).
  - Automatic LinkedIn REST API version negotiation with `/v2/ugcPosts` fallback and a local post history index (`~/.omni/social/linkedin_posts.tsv`) for listing and deleting published posts.
- **Markdown Content & Draft Workflow (`social content`)**: Organize posts across named content directories, inspect `[DRAFT]` vs. `[READY]` status in terminal listings, and prevent accidental publication of posts marked `draft: true`.
- **Reusable Templates (`social template`)**: Includes built-in `default` and `article` templates and supports custom global or platform-specific templates.

## Prerequisites

- `curl`, `awk`, `base64`, `cksum`
- `nc` / netcat (optional, used to automatically capture the local OAuth 2.0 redirect callback)
- `xdg-open` (optional, opens the OAuth authorization URL in your browser)

---

# Install

## Via the Official `omni` Provider

```bash
omni providers install omni
omni tools use social --provider omni
```

## From a Local Directory

```bash
omni tools use -p /path/to/omni/social
```

## Directly from GitHub

```bash
omni tools use -u https://github.com/omnishell/social.git
```

---

# Usage

```bash
social <platform|content|template|publish> [subcommand] [options]
```

## 1. Platform Configuration (`social platform`)

Configure one or more social platform accounts (persisted per Omni environment in `$OMNIRC`):

```bash
# List configured platforms in the current environment (or -a/--all environments)
social platform ls [-e|--env <env>] [-a|--all]

# Configure a LinkedIn platform using an existing OAuth access token
social platform set <name> -t|--type linkedin \
  --token <access_token> \
  [-u|--urn <urn:li:person:...>] \
  [--org-id <organization_id>] \
  [-v|--version <YYYYMM>] \
  [-e|--env <env>]

# Configure a LinkedIn platform via interactive OAuth 2.0 flow (client ID + secret)
social platform set <name> -t|--type linkedin \
  --client-id <client_id> \
  --client-secret <client_secret> \
  [-r|--redirect <http://localhost:8080/callback>] \
  [-s|--scopes <"openid profile email w_member_social">]

# Set the active default platform for the current environment
social platform use <name> [-e|--env <env>]

# Inspect platform details, profile info (`me` / `profile`), or fetch a specific post/org URN
social platform get <name> [<me|profile|urn>] [-e|--env <env>]

# Remove a configured platform
social platform rm <name> [-e|--env <env>]
```

### Supported Environment Variables

- `LINKEDIN_ACCESS_TOKEN`: Fallback OAuth access token if not stored in `$OMNIRC`
- `LINKEDIN_CLIENT_ID` / `LINKEDIN_CLIENT_SECRET`: Fallback OAuth application credentials
- `LINKEDIN_API_VERSION`: Override LinkedIn REST API version header (`YYYYMM`)

---

## 2. Managing Content & Drafts (`social content`)

Register content directories and manage Markdown post files:

```bash
# Register a named content directory
social content set <dir_name> <directory_path>

# List registered content directories
social content ls --dirs

# List all posts inside a content directory (or across all directories) with [DRAFT]/[READY] badges
social content ls [<dir_name>]

# Create a new Markdown post from a template and open it in $EDITOR
social content new [<post_name>] -d|--dir <dir_name> [-t|--template <template>] [-p|--platform <platform>]

# Edit an existing post in $EDITOR
social content edit <post_name> [-d|--dir <dir_name>]

# Print the raw contents of a post
social content get <post_name> [-d|--dir <dir_name>]

# Delete a post file (with confirmation) or unregister a content directory
social content rm <post_or_dir_name> [-d|--dir <dir_name>]
```

### Post Markdown & Frontmatter Format

Posts use YAML frontmatter followed by the post body:

```markdown
---
title: Launching Omni v0.1.0
thumb: ./assets/banner.png
url: https://github.com/omnishell/omni
description: A speed-optimized modular shell framework for Bash
visibility: PUBLIC
account: me
draft: false
---
Excited to share **Omni**, our modular shell framework for Bash!

#bash #linux #opensource #devtools
```

| Frontmatter Key | Values | Description |
| :--- | :--- | :--- |
| `title` | String | Optional headline/title for the post or shared article |
| `thumb` | File path | Optional local image path (`png`, `jpg`, `gif`) to upload and attach |
| `url` | URL | Optional external article URL to attach as an article share |
| `description` | String | Optional article description when `url` is specified |
| `visibility` | `PUBLIC`, `CONNECTIONS`, `LOGGED_IN` | Audience visibility setting (default: `PUBLIC`) |
| `account` | `me` or Organization ID / URN | Author identity (`me` for personal profile, or numeric org ID / `urn:li:organization:...`) |
| `draft` | `true` / `false` | When `true`, `social publish` blocks publishing until set to `false` |

---

## 3. Managing Templates (`social template`)

`social` ships with two built-in templates (`default` and `article`) and lets you register custom template directories or files:

```bash
# List available templates (built-in and custom) or template directories
social template ls [<template_dir>] [-p|--platform <platform>] [-t|--templates] [--dirs]

# Register a custom template directory or single template file
social template set <name> <path> [-p|--platform <platform>] [-f|--file <filename>]

# Create a new template inside a registered template directory
social template new <template_name> -d|--dir <template_dir> [-p|--platform <platform>]

# Edit or view a template
social template edit <template_name> [-d|--dir <template_dir>] [-p|--platform <platform>]
social template get <template_name> [-d|--dir <template_dir>] [-p|--platform <platform>]

# Remove a template or unregister a template directory
social template rm <name> [-d|--dir <template_dir>] [-p|--platform <platform>]
```

---

## 4. Publishing & Managing Remote Posts (`social publish`)

```bash
# Publish a saved content post (shows preview and prompts for confirmation unless -y/--yes is passed)
social publish <content_name> [-d|--dir <dir_name>] [-p|--platform <platform>] [-e|--edit] [-y|--yes]

# Compose a one-off post in $EDITOR from a template and publish immediately
social publish [-p|--platform <platform>] [-t|--template <default|article|custom>]

# List recently published posts on the platform (queries API with local history fallback)
social publish ls [-p|--platform <platform>] [-c|--count <N>]

# Delete a published post from LinkedIn by its URN (e.g., urn:li:share:... or urn:li:ugcPost:...)
social publish rm <post_urn> [-p|--platform <platform>]
```
