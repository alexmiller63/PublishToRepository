# ChatGPT - RTFM!

## Purpose

This document is the operating manual for ChatGPT when using the PublishToRepository mailbox. Read and follow it before changing the mailbox, workflow, or repository structure.

## Mailbox security

The Google Doc used as the publish mailbox is intentionally configured for **Anyone with the link → Viewer** access so the GitHub Actions workflow can fetch it as plain text.

The repository stores the mailbox document ID in `tools/id.txt`. That ID is an identifier, not a secret. Anyone who obtains the ID can use it to read the mailbox while the Google Doc has public-read access.

Therefore:

- Treat the mailbox as **publicly readable**.
- Never put passwords, API keys, access tokens, credentials, private personal information, or other secrets in the mailbox.
- Do not assume that Base64 encoding makes mailbox contents secret. Base64 is an encoding, not encryption.
- The mailbox should contain only deliberately publishable repository instructions and file content.

## Mailbox protocol

The mailbox contains either one JSON publish request or nothing.

An empty mailbox is the normal idle state. A workflow run against an empty mailbox is a safe no-op.

A new request must replace the entire mailbox contents. Never append one JSON request to another.

## Successful publish lifecycle

Use this sequence for an ordinary publication:

1. Verify the exact repository paths and read the current versions of files that will be changed.
2. Construct the complete desired UTF-8 contents.
3. Base64-encode those exact bytes and prepare one JSON request.
4. Replace the entire mailbox contents with that request.
5. Tell the user the mailbox is ready.
6. The user manually runs **Publish to Repository**.
7. Verify the workflow result and the resulting repository state.
8. If verification succeeds, clear the mailbox and return it to the idle state.

A green workflow is not the final proof by itself. Verify the repository result.

## Failure rule

If the workflow fails, do **not** automatically clear the mailbox.

Keep the failed request intact until the cause is understood. Correct the underlying problem before rerunning when the same input would reproduce the same failure.

This is deliberate: clearing a failed request can destroy the exact input needed for diagnosis and recovery.

## Preserve source text

File contents are carried in the JSON request as Base64 so that Google Docs does not reinterpret or reformat the source text. The workflow decodes the Base64 and verifies that the resulting bytes are valid UTF-8 text before writing them.

Do not replace this mechanism with ordinary pasted source text unless the workflow is deliberately redesigned and tested.

## Repository safety

- Never invent an existing repository path or filename. Verify it first.
- Only propose a new path or filename when creating a genuinely new file.
- When changing an existing file, preserve its complete contents except for the requested change.
- Be especially careful with workflow files, mailbox handling, and cleanup logic.
- Do not delete files merely because they appear unused. Confirm the intended deletion first.

## Workflow files are special

Under the current GitHub authorization, the mailbox publisher can write ordinary repository files but cannot reliably create or modify `.github/workflows/*`.

GitHub can reject the final push when the GitHub App/token lacks the separate permission required to modify workflow files. This failure occurs after the mailbox has been fetched and the local repository has been changed, so it can look as though publication nearly succeeded.

Therefore:

- Do not send `.github/workflows/*` changes through the mailbox under the current permission model.
- Edit and commit workflow files outside the mailbox publisher.
- Do not rerun an unchanged workflow-file payload after this permission failure.
- If the authorization model is changed later, test workflow-file publishing separately before treating it as supported.

The publisher must not be used to perform unsafe self-surgery on its own workflow.

## ChatGPT behavior

When working on this repository:

1. Read this document first.
2. Inspect the existing implementation before changing it.
3. Prefer fixing the underlying problem over adding a no-op or workaround merely to make a test pass.
4. Keep the mailbox deliberately small, simple, and disposable.
5. Document important architectural and security decisions in the repository.
6. Test changes incrementally and distinguish a successful workflow run from a successful end-to-end publication.
7. After verified success, clear the mailbox.
8. After failure, preserve the mailbox until diagnosis is complete.

## Tested behavior

The current system has been verified to support an empty-mailbox no-op, ordinary UTF-8 text file creation/replacement, and file deletion. Each successful operation was checked against repository state.

## Why this matters

The mailbox is a communication channel between ChatGPT and GitHub, not a secure storage system. Its convenience depends on public readability, so its contents must always be safe to expose. Its reliability depends on treating the repository as the source of truth and clearing the mailbox only after success is verified.
