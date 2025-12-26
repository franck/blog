---
title: "How to Set Up an AI-Managed Hugo Blog with Cloudflare Pages"
date: 2025-12-26
draft: false
categories: ["workflow"]
description: "I used to dread publishing blog posts. Not the writing—that part I loved. But after finishing a draft, I'd spend 30 minutes on tedious work..."
---

I used to dread publishing blog posts.

Not the writing—that part I loved. But after finishing a draft, I'd spend 30 minutes on tedious work: reformatting front matter, hunting for internal linking opportunities, manually updating old posts to reference new ones.

Most posts never made it out of my drafts folder. The friction killed them.

Now? One command. I type `/publish`, and my AI handles everything—formatting, internal links in both directions, archiving the draft. The post goes live while I move on to the next idea.

Here's how to build the same system.

## What We're Building

Before this system, I had [drafts collecting dust](/posts/25-years-of-failing-to-get-organized/). After? I publish consistently.

The workflow:

1. Write drafts in Obsidian (where I already take notes)
2. AI refines through versions (v1 → v2 → final)
3. Run `/publish`—AI transforms and publishes to Hugo
4. AI adds internal links *in both directions*
5. Push to GitHub, Cloudflare deploys in seconds

Each new post strengthens your entire site's SEO. Your blog becomes a web, not a list of disconnected pages.

## Prerequisites

You'll need:

- **Obsidian vault**

- **Hugo** installed:

```bash
# macOS
brew install hugo

# Linux (Debian/Ubuntu)
sudo apt install hugo

# Linux (snap)
sudo snap install hugo

# Windows
choco install hugo-extended

# Or: https://gohugo.io/installation/
```

Verify:

```bash
hugo version
```

- **GitHub account**

- **Cloudflare account** (free tier)

- **AI CLI** — I use [OpenCode](https://opencode.ai). Works with Claude Code, Cursor, or any AI that reads/writes files.

With these tools installed, let's set up the folder structure that makes this workflow possible.

## Folder Structure

```
vault/
├── Projects/Blog/
│   ├── article-ideas.md      # Backlog
│   └── drafts/
│       ├── my-post-v1.md     # Rough
│       ├── my-post-v2.md     # Refined
│       ├── my-post-final.md  # Ready
│       └── published/        # Archived
└── blog/                     # Hugo (separate git repo)
    ├── config.toml
    ├── content/posts/
    └── themes/
```

The `blog/` folder is a **separate git repository**. Your messy drafts stay private. Your polished posts go public.

Now let's initialize Hugo.

## Hugo Setup

```bash
cd your-vault
hugo new site blog
cd blog
git init
```

Create `config.toml`:

```toml
baseURL = "https://your-blog.pages.dev/"
languageCode = "en-us"
title = "Your Blog Title"
theme = "your-theme"

[pagination]
  pagerSize = 10

[params]
  description = "Your blog description"
  author = "Your Name"

[markup]
  [markup.highlight]
    style = "github"
    lineNos = false
  [markup.goldmark.renderer]
    unsafe = true

[taxonomies]
  category = "categories"
  tag = "tags"

[menu]
  [[menu.main]]
    name = "Home"
    url = "/"
    weight = 1
  [[menu.main]]
    name = "Posts"
    url = "/posts/"
    weight = 2
```

Add a theme:

```bash
git submodule add https://github.com/theNewDynamic/gohugo-theme-ananke themes/ananke
```

Test:

```bash
hugo server
```

Hugo is ready. But we're not writing directly in Hugo—we're writing in Obsidian, where ideas flow naturally. Here's how drafts should be formatted.

## Draft Format

Obsidian drafts use this front matter:

```yaml
---
title: "Your Article Title"
status: draft
pillar: productivity
date: 2025-12-26
versions:
  - v1: rough draft
  - v2: clarity pass
  - final: ready to publish
---

# Your Article Title

Content here...
```

Version progression:
- **v1** — Get ideas down
- **v2** — AI improves clarity and structure
- **final** — Hook refined, ready to publish

When a draft reaches `-final.md`, it's ready.

The version progression is where the magic happens. Each version gets better with AI assistance. But the real power comes from the publish command.

## The Publish Command

This is the heart of the system—the single command that eliminates all the tedious work.

Create `.opencode/command/publish.md`:

```markdown
---
description: Publish a blog draft to Hugo with SEO-friendly internal linking
---

# Publish Blog Command

Publish from `Projects/Blog/drafts/` to `blog/content/posts/` with automatic internal linking.

## Process

### Step 1: List Available Drafts

1. Scan for files ending in `-final.md`
2. Read front matter (title, pillar, date)
3. Display options:

Ready to publish:

1. my-post-final.md
   "My Post Title"
   Category: productivity | Date: 2025-12-26

Select draft (1-3), or 'q' to quit:

### Step 2: Prepare Hugo Post

1. Generate slug from title:
   - Lowercase, hyphens for spaces
   - Remove special characters
   - Max 50 characters
   - Example: "My Great Post!" → `my-great-post`

2. Auto-generate description:
   - First paragraph, ~150 characters
   - End at word boundary, add ellipsis

3. Transform front matter:

**From Obsidian:**
```yaml
---
title: "Your Article Title"
status: ready-for-review
pillar: productivity
date: 2025-12-26
versions:
  - v1: rough draft
---
```

**To Hugo:**
```yaml
---
title: "Your Article Title"
date: 2025-12-26
draft: false
categories: ["productivity"]
description: "First paragraph truncated..."
---
```

4. Remove: `status`, `pillar`, `based_on`, `versions`
5. Map `pillar` → `categories`
6. Remove H1 if it duplicates title

### Step 3: SEO Internal Linking

The key differentiator.

#### 3a. Analyze Existing Content

1. Read all posts in `blog/content/posts/`
2. Extract: title, slug, categories, headings, bold terms
3. Build topic map:
   ```
   "daily note" → /posts/daily-note/
   "obsidian" → /posts/obsidian-setup/
   ```

#### 3b. Link FROM New Post TO Existing

Scan for natural mentions of existing topics.

Rules:
- First mention only (not in headers)
- Max 3-5 links per post
- Natural anchor text
- Never link same target twice

Example:
```markdown
# Before
I typed a task I'd postponed for the fourth time.

# After
I typed a task I'd postponed for the fourth time. This was before I discovered [daily notes](/posts/daily-note/).
```

#### 3c. Link FROM Existing Posts TO New

For each existing post:
- Same category? Add "Related Reading"
- Mentions new post's topic? Add inline link
- Not related? Skip

Strategies:

1. **Inline link** where topic is mentioned:
   ```markdown
   I've tried many productivity systems. My [25 years of failed attempts](/posts/25-years-failing/) taught me what works.
   ```

2. **Related Reading** at end:
   ```markdown
   ---
   **Related:** [New Post Title](/posts/new-post/)
   ```

3. **No link** if unrelated

#### 3d. Display Plan

```
Internal Linking Plan
=====================

NEW POST → existing:
  Line 61: "daily note" → /posts/daily-note/

EXISTING → new post:
  hello-world.md → Related Reading section

Total: 1 outbound, 1 inbound

Proceed? (y/n)
```

### Step 4: Write Files

1. Write new post to `blog/content/posts/{slug}.md`
2. Update existing posts with new links
3. Archive drafts:
   - Move all versions to `drafts/published/`
   - Update final's status to `published`

### Step 5: Commit

```bash
git -C blog add content/posts/*.md
git -C blog commit -m "Publish: {title}"
```

### Step 6: Summary

```
Published: blog/content/posts/my-post.md
  - 2 internal links added

Updated:
  - hello-world.md (Related Reading)

Archived: drafts/published/

Deploy: cd blog && git push
```
```

## SEO Linking Rules

I learned these the hard way—through traffic drops and wasted effort. Save yourself the pain:

1. **Anchor text variety** — I once linked every mention of "productivity" to the same post with the same text. Google noticed. Use natural variations.

2. **Contextual relevance** — Links must help *readers*, not just search engines. If the link feels forced, delete it.

3. **Link to cornerstone content** — Newer posts should link to your pillar articles. This distributes authority where it matters.

4. **Reciprocal linking** — Post A links to B, B links back. This creates topic clusters that Google rewards.

5. **First mention only** — Multiple links to the same page dilute value and annoy readers.

6. **No header links** — Keep links in body text. Header links look spammy and get ignored.

7. **Link density** — One link per 300-500 words. More than that feels aggressive.

## Cloudflare Pages Deployment

### Connect Repository

1. Push `blog/` to GitHub
2. Cloudflare Dashboard → Pages → Create project
3. Connect to Git, authorize GitHub
4. Select your blog repository

### Build Settings

- **Branch:** `main`
- **Build command:** `hugo --minify`
- **Output directory:** `public`

Save and Deploy.

### Update Base URL

Cloudflare assigns something like `your-blog-abc.pages.dev`. Update `config.toml`:

```toml
baseURL = "https://your-blog-abc.pages.dev/"
```

Commit, push. Cloudflare redeploys automatically.

### Custom Domain

Add in Cloudflare Pages settings. SSL handled automatically.

## Complete Workflow

```
1. Capture idea → article-ideas.md

2. Write draft → my-post-v1.md

3. AI refines → my-post-v2.md

4. Final polish → my-post-final.md

5. /publish
   → Transform front matter
   → Add internal links
   → Commit

6. git push → Live
```

## The Real Win

Setup takes an hour. Maybe two if you're customizing themes.

But here's what changes:

Every post you publish makes your entire site smarter. Links flow automatically. Old content gets refreshed. Your blog becomes a living system, not a graveyard of disconnected pages.

I used to publish rarely—when I could stomach the tedium.

Now I publish consistently. The friction is gone. The ideas flow.

Traditional blogging: write, publish, forget.

This system: write, publish, watch your site get smarter.

---

One command. `/publish`. Your words are live.

What will you publish first?
