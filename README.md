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

## ChatGPT School

The repository also contains a manually triggered **Send ChatGPT to School** workflow at `.github/workflows/send-chatgpt-to-school.yaml`.

The school workflow is read-only. It checks that `ChatGPT-school.yaml` and `ChatGPT - RTFM!.md` exist, validates `ChatGPT-school.yaml` as YAML, verifies that the operating manual is readable and non-empty, and reports successful graduation when those checks pass.

This workflow does not retrain or modify ChatGPT. It provides a repeatable validation step for the repository's machine-readable curriculum and operating manual. ChatGPT can then read those repository files as instructions before working with the publisher.

The first **Send ChatGPT to School** run was successfully tested after the workflow was created.

## Mailbox lifecycle

The mailbox contains either one JSON publish request or nothing.

The normal operating cycle is:

1. ChatGPT verifies the repository state and prepares one complete request.
2. ChatGPT replaces the mailbox contents with that request.
3. The user manually runs **Publish to Repository**.
4. The workflow validates, applies, commits, and pushes the requested changes.
5. The resulting repository state is verified.
6. Only after successful verification, ChatGPT clears the mailbox.

An empty mailbox is the normal idle state. Running the workflow with an empty mailbox is a safe no-op and produces no repository commit.

If publication fails, do **not** clear the mailbox until the failure has been diagnosed. Preserving the request makes the failure reproducible and prevents loss of the intended change.

## Verified behavior

The current implementation has been tested for:

- an empty-mailbox successful no-op;
- creating or replacing an ordinary UTF-8 text file;
- deleting an existing ordinary file;
- committing and pushing those changes to `main`.

Repository state, not merely a green workflow badge, is the final verification of success.

## Workflow-file limitation

The publisher is intended for ordinary repository files.

Under the current GitHub authorization, a publish request that tries to create or modify a file under `.github/workflows/` can be rejected at `git push` because the GitHub App/token does not have the separate workflow-modification permission.

Therefore, changes to `.github/workflows/*` must currently be edited and committed outside the mailbox publisher, unless the authorization model is deliberately changed and tested.

Do not use the mailbox to self-modify the publishing workflow under the current permission model.

## Mailbox security

The mailbox is publicly readable because the workflow fetches the Google Doc through its plain-text export URL.

Treat the mailbox as public transport, not secure storage. Never put passwords, access tokens, API keys, credentials, private personal information, or other secrets in it. Base64 preserves bytes; it does not provide secrecy.

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
```

`content_base64` is the Base64 encoding of the complete desired UTF-8 file contents. File entries are complete replacements, not patches.

Before modifying or deleting an existing file, verify its exact repository path and current contents. Never guess an existing filename or path.
