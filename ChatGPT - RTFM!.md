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

After a request has been successfully processed, the mailbox should be cleared.

## Preserve source text

File contents are carried in the JSON request as Base64 so that Google Docs does not reinterpret or reformat the source text. The workflow decodes the Base64 and verifies that the resulting bytes are valid UTF-8 text before writing them.

Do not replace this mechanism with ordinary pasted source text unless the workflow is deliberately redesigned and tested.

## Repository safety

- Never invent an existing repository path or filename. Verify it first.
- Only propose a new path or filename when creating a genuinely new file.
- When changing an existing file, preserve its complete contents except for the requested change.
- Be especially careful with workflow files, mailbox handling, and cleanup logic.
- Do not delete files merely because they appear unused. Confirm the intended deletion first.

## ChatGPT behavior

When working on this repository:

1. Read this document first.
2. Inspect the existing implementation before changing it.
3. Prefer fixing the underlying problem over adding a no-op or workaround merely to make a test pass.
4. Keep the mailbox deliberately small, simple, and disposable.
5. Document important architectural and security decisions in the repository.
6. Test changes incrementally and distinguish a successful workflow run from a successful end-to-end publication.

## Why this matters

The mailbox is a communication channel between ChatGPT and GitHub, not a secure storage system. Its convenience depends on public readability, so its contents must always be safe to expose.
