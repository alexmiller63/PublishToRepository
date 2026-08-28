# PublishToRepository

PublishToRepository is a small mailbox that lets ChatGPT prepare UTF-8 text files and publish them to a GitHub repository without requiring the user to paste file contents into GitHub.

## How it works

ChatGPT prepares a JSON publish request and writes it to a designated Google Doc mailbox. The repository stores that document's ID in `tools/id.txt`.

When the **Publish to Repository** GitHub Actions workflow is run manually, it:

1. Reads the Google Doc ID from `tools/id.txt`.
2. Exports the Google Doc as plain text.
3. Treats an empty or whitespace-only document as an idle mailbox and exits successfully.
4. Parses a non-empty document as JSON.
5. Base64-decodes each file's contents.
6. Verifies that the decoded bytes are valid UTF-8 text.
7. Writes the files at their specified repository paths.
8. Deletes any files listed in the request's `delete` array.
9. Commits and pushes the resulting changes.

The workflow has no parameters. The Google Doc is the mailbox between ChatGPT and GitHub.

## Mailbox lifecycle

The mailbox contains either one JSON publish request or nothing.

After a request has been successfully processed, the mailbox should be cleared. An empty mailbox is the normal idle state; running the workflow with an empty mailbox is safe and produces no repository commit.

## Publish request format

The mailbox contains one JSON object:

```json
{
  "commit_message": "Example repository update",
  "files": [
    {
      "path": "example.txt",
      "content_base64": "S2lscm95IHdhcyBoZXJlIQ=="
    }
  ],
  "delete": [
    "old-example.txt"
  ]
}