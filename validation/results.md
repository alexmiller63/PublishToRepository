# PublishToRepository Validation Results

This log records observed validation outcomes for the staged validation plan.

## Test 1 — Smoke test: single create

**Result:** PASS

**Target:** `validation/smoke-test-1.txt`

**Expected contents:**
```text
Validation Test 1
Kilroy was here.
```

**Observed:** The publisher created the file with the exact expected contents.

**Verified Git blob SHA:** `f3764cb533e98a4c999bd9d90d8f26732c4339df`

**Mailbox lifecycle:** The successful request remained in the mailbox until repository verification. The mailbox was then cleared and verified truly empty (`paragraphs: []`).

## Test 2 — Exact replacement

**Result:** PASS

**Target:** `validation/smoke-test-1.txt`

**Expected replacement contents:**
```text
Validation Test 2
Kilroy was here again.
Old contents replaced.
```

**Observed:** The existing file was replaced completely with the exact expected contents. No Test 1 content remained.

**Verified Git blob SHA:** `e2f5f399288d83d1258b28a3bca4d9c2923b72c1`

**Mailbox lifecycle:** Repository replacement was verified before mailbox cleanup.

## Test 3 — Single delete

**Result:** PASS

**Target:** `validation/smoke-test-1.txt`

**Observed:** The publisher deleted the target file. A direct repository fetch returned 404 after publication.

**Unrelated-file check:** `validation/results.md` remained present and unchanged during the delete test, with Git blob SHA `4ee4db2f50151a705e2a9ad372245247aedc50a2`.

**Mailbox lifecycle:** Repository deletion was verified before mailbox cleanup. The mailbox was then cleared and verified truly empty (`paragraphs: []`).

## Running observations

- Strict mailbox JSON and repository verification are functioning for simple create, replacement, and delete operations.
- Revision-controlled mailbox clearing worked after Tests 1, 2, and 3.
- Earlier in the session, Google Drive connector safety intermittently blocked `deleteContentRange`, including a modest deletion. Later exact revision-controlled clears succeeded again. Treat this as intermittent connector behavior; do not weaken the mailbox protocol or use replacement tricks to evade the guard.
- `.github/workflows/` remains outside mailbox write authority and must stay protected throughout validation.
