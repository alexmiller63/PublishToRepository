# PublishToRepository

PublishToRepository is a small bridge that lets ChatGPT prepare UTF-8 text files and publish them to a GitHub repository without requiring the user to paste file contents into GitHub.

## How it works

ChatGPT prepares a JSON publish request and writes it to a designated Google Doc. The repository stores that document's ID in `tools/id.txt`.

When the **Publish to Repository** GitHub Actions workflow is run manually, it:

1. Reads the Google Doc ID from `tools/id.txt`.
2. Exports the Google Doc as plain text.
3. Parses the document as JSON.
4. Base64-decodes each file's contents.
5. Verifies that the decoded bytes are valid UTF-8 text.
6. Writes the files at their specified repository paths.
7. Commits and pushes the resulting changes.

The workflow has no parameters. The Google Doc acts as a bridge between ChatGPT and GitHub.

## Publish request format

The bridge document contains one JSON object:

```json
{
  "commit_message": "Example repository update",
  "files": [
    {
      "path": "example.txt",
      "content_base64": "S2lscm95IHdhcyBoZXJlIQ=="
    }
  ]
}
```

`content_base64` is the Base64 representation of the exact UTF-8 bytes to be written. Using Base64 prevents Google Docs or the transport path from altering source formatting, whitespace, line endings, quotes, or other significant text.

## Safety checks

The workflow rejects empty file lists, missing paths, missing or invalid Base64, files that do not decode as UTF-8, and paths that escape the repository root.

This publisher is intentionally limited to UTF-8 text files. Binary files should be transferred by another mechanism.

## Typical use

Discuss the desired repository changes with ChatGPT. ChatGPT prepares the complete file contents, encodes them, and writes the publish request to the bridge document. Then manually run **Actions → Publish to Repository → Run workflow**.

GitHub Actions performs the write, commit, and push.

## Installation in another repository

Copy the publishing workflow into the target repository and create `tools/id.txt` containing the Google Doc ID used as that repository's publish bridge. The workflow requires `contents: write` permission so GitHub Actions can commit the generated changes.

The bridge Google Doc must be accessible to the workflow through its plain-text export URL.

## Design goal

The goal is a simple division of labor: ChatGPT composes repository changes; the bridge carries an exact, preservation-safe payload; GitHub Actions performs the authenticated repository write.
