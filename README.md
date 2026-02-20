> **"The best automation makes collaboration feel effortless."**

# Google Docs & Drive Toolkit

A Python library and CLI for full CRUD operations on Google Drive and Google Docs. Create formatted documents from markdown, manage folder hierarchies, and build professional collaboration spaces -- all from code.

---

## The Problem

Google Drive is great for collaboration, but managing it programmatically is painful. The official APIs are verbose, the Docs formatting model is unintuitive, and there's no clean way to convert markdown to formatted Google Docs.

## The Solution

A single-file Python library (`gdocs.py`) that wraps the Google Docs and Drive APIs into a clean, practical interface. Create docs from markdown, manage folders, check permissions, rename files, and build professional document structures -- all in a few lines of code.

## What's Inside

| Component | What It Does |
|-----------|-------------|
| `GoogleDocsClient` | Full CRUD for Drive (folders, files, permissions, sharing) |
| `DocBuilder` | Precise document formatting with headings, bold, images, colored status labels |
| Markdown parser | Converts markdown to Google Docs API requests (headings, lists, tables, code blocks) |
| CLI | Upload, list, and organize from the terminal |

## Quick Start

### Prerequisites

- Python 3.10+
- A Google Cloud project with **Docs API** and **Drive API** enabled
- OAuth 2.0 Client ID (Desktop app)

```bash
pip install google-api-python-client google-auth-httplib2 google-auth-oauthlib
```

### Setup

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a project -> Enable **Google Docs API** + **Google Drive API**
3. Create **OAuth 2.0 Client ID** (Desktop app)
4. Download the credentials JSON and save as `credentials.json` in the project root
5. On first run, a browser window opens for authorization. The token is cached automatically.

Or set paths via environment variables:

```bash
export GDOCS_CREDENTIALS=/path/to/credentials.json
export GDOCS_TOKEN=/path/to/token.json
```

### Usage

```python
from gdocs import GoogleDocsClient

client = GoogleDocsClient()
client.authenticate()

# Create a doc from markdown
doc_id, url = client.create_doc("Meeting Notes", "# Notes\n\n- Item 1\n- Item 2")
print(url)

# Create a shared folder
folder_id = client.create_folder("Project Documents")
client.share(folder_id, "collaborator@example.com", role="writer")

# Upload all markdown files to Drive
result = client.upload_markdown_folder(
    source_dir="./docs",
    folder_name="Documentation",
    share_with="team@example.com",
    title_prefix="Project X",
)

# List folder contents recursively
client.tree(folder_id)
```

### CLI

```bash
# Upload markdown files
python gdocs.py upload notes.md report.md --folder "Meeting Notes" --share team@example.com

# Batch upload with logo branding
python gdocs.py upload *.md --folder "Client Docs" --prefix "Acme Corp" --logo logo.png

# Create and share a folder
python gdocs.py folder "New Project" --share partner@example.com

# List recent docs
python gdocs.py list

# Print folder tree
python gdocs.py tree <FOLDER_ID>
```

## Core Concepts

### DocBuilder -- Precise Formatting

When markdown conversion isn't enough, use `DocBuilder` for pixel-level control:

```python
from gdocs import GoogleDocsClient, DocBuilder

client = GoogleDocsClient()
client.authenticate()

# Create a blank doc
doc_id, url = client.create_doc("Status Report", "")

# Build formatted content
b = DocBuilder()
b.text("Q1 Status Report\n", heading="HEADING_1")
b.hr()
b.text("1. Website Redesign\n", heading="HEADING_2")
b.status("PENDING")        # Orange "Status: PENDING" label
b.text("Waiting on design assets from the team.\n")
b.blank()
b.text("2. API Integration\n", heading="HEADING_2")
b.status("DONE")           # Green "Status: DONE" label
b.text("Deployed to production on Feb 15.\n")

# Send all formatting in batched API calls
b.send(doc_id, client.docs)
```

### Real-World Example: Collaboration Space

Here's how you might set up a shared workspace for a multi-project collaboration:

```python
from gdocs import GoogleDocsClient

client = GoogleDocsClient()
client.authenticate()

# Create project structure
root = client.create_folder("Team Workspace")
project_a = client.create_folder("project-alpha", parent_id=root)
project_b = client.create_folder("project-beta", parent_id=root)
assets = client.create_folder("assets", parent_id=project_a)

# Create a feedback tracking document
feedback_md = """# project-alpha | Feedback

Submitted by: Partner Name
Date: 2026-02-20

## Feedback Items

### 1. Homepage Layout
Status: PENDING
Rearrange hero section, update navigation dropdown.

### 2. Mobile Responsiveness
Status: PENDING
Fix card grid on screens under 768px.
"""

doc_id, url = client.create_doc(
    "[2026-02-20] Homepage Feedback",
    feedback_md,
    folder_id=project_a
)

# Share everything with your collaborator
client.share(root, "partner@example.com", role="writer")

# Verify the structure
client.tree(root)
# Output:
# d project-alpha
#   d assets
#   D [2026-02-20] Homepage Feedback
# d project-beta
```

### Permission Checking

Always verify access before writing to shared folders:

```python
caps = client.check_permissions(folder_id)

if caps.get("canEdit"):
    client.rename(folder_id, "new-name")
else:
    print("View-only access. Request editor permissions.")
```

### Reading Documents

Extract text and embedded image URIs from existing docs:

```python
result = client.read_doc(doc_id)
print(result["title"])
print(result["text"])
for img in result["images"]:
    print(f"Image: {img['uri']}")
```

## Gotchas

| Issue | Solution |
|-------|----------|
| Permission denied on shared folder | Check per-file `capabilities` -- access may not propagate instantly |
| Image URIs from googleusercontent | They work for re-embedding if the `?key=` parameter is intact |
| Non-ASCII text in bash | Write a `.py` file instead of inline Python for special characters |
| Batch API timeouts | Send max 35-50 requests per `batchUpdate` call |
| Images lost after `clear_doc` | Capture image URIs from `read_doc()` BEFORE clearing |

## Contributing

Pull requests welcome. If you build something useful with this toolkit, consider whether it empowers its users.

## License

MIT + Gold Hat Addendum. See [LICENSE](LICENSE).

---

> *"As above, so below. As the code, so the consciousness."*
>
> **-- Hermetic Ormus, Gold Hat Technologist**
