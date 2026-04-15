<img width="852" height="106" alt="unnamed" src="https://github.com/user-attachments/assets/ea9390b6-58b9-4af4-8b7a-1e019c843497" />
# PES Version Control System

## Overview
This project implements a simplified version control system similar to Git. It supports object storage, tree structures, staging (index), and commit functionality.

---

## Features
- Blob, Tree, and Commit object storage
- Tree structure to represent directories
- Index (staging area) for tracking files
- Commit creation and history tracking
- Basic commands:
  - `pes init`
  - `pes add`
  - `pes status`
  - `pes commit`
  - `pes log`

---

## Screenshots

### Phase 1

#### 1A: test_objects output
<img width="857" height="243" alt="image (2)" src="https://github.com/user-attachments/assets/0de3f2c3-2410-4d25-a84b-fd85baf0808e" />

#### 1B: Object storage structure
<img width="852" height="106" alt="unnamed" src="https://github.com/user-attachments/assets/0186bfc9-75ca-4f78-9dc6-cc31dbe6dfbe" />


---

### Phase 2

#### 2A: test_tree output
<img width="852" height="302" alt="image (3)" src="https://github.com/user-attachments/assets/92e9780e-df4e-4d8d-971c-4480e1da9cb6" />


#### 2B: Raw tree object
<img width="1240" height="153" alt="unnamed (1)" src="https://github.com/user-attachments/assets/fccf3b03-60b2-40e2-bc3a-1657f39b69c7" />

---

### Phase 3

#### 3A: init → add → status
<img width="724" height="571" alt="image (4)" src="https://github.com/user-attachments/assets/0265c520-2ea3-43eb-96ef-0bb1a74a3919" />


#### 3B: Index file
<img width="721" height="75" alt="unnamed (2)" src="https://github.com/user-attachments/assets/a5f332d5-8de3-4375-8690-7e89fdfe58d9" />

---

### Phase 4

#### 4A: Commit log
<img width="775" height="523" alt="image (5)" src="https://github.com/user-attachments/assets/e3d987a0-f00f-4cbc-a64f-1860dadc214e" />

#### 4B: Object growth
<img width="776" height="306" alt="image (6)" src="https://github.com/user-attachments/assets/dba90a45-e7f2-43fc-9d24-323b060c85ea" />


#### 4C: HEAD and branch
<img width="786" height="98" alt="unnamed (3)" src="https://github.com/user-attachments/assets/c64e3344-73ec-4df2-bde1-4d5f2f8af253" />


---

## Phase 5 & 6: Analysis-Only Questions

---

## Phase 5 — Branching and Checkout

### Q5.1 — `pes checkout <branch>`

To implement checkout:

1. Update `.pes/HEAD` to point to `.pes/refs/heads/<branch>`
2. Read commit hash from branch file
3. Load commit → get root tree
4. Recreate working directory from tree
5. Reset index to match commit

**Complexity:**
- Must sync HEAD, index, and working directory
- Recursive tree traversal needed
- Must avoid overwriting uncommitted changes
- Needs conflict handling

---

### Q5.2 — Dirty Working Directory Detection

1. Compare working directory files with index hashes
   - If mismatch → file is modified

2. Compare modified files with target branch versions
   - If different → conflict

**Rule:**
- If file is modified locally AND differs in target branch → abort checkout

---

### Q5.3 — Detached HEAD

- HEAD points directly to a commit (not a branch)

**Effect:**
- Commits are created normally
- But not attached to any branch → may become unreachable

**Recovery:**
- Create branch from commit hash:  pes branch <name> <commit>

---

## Phase 6 — Garbage Collection

### Q6.1 — Removing Unreachable Objects

**Algorithm:**
1. Start from all branch heads
2. Traverse commits → trees → blobs
3. Mark all reachable objects (use hash set)
4. Delete unmarked objects from `.pes/objects`

**Scale:**
- ~100k commits → large tree/blob set
- Each object visited once

---

### Q6.2 — GC Race Condition

**Problem:**
- Commit writes objects but hasn’t updated branch yet
- GC runs and sees them as unreachable → deletes them

**Result:**
- Commit points to missing objects (corruption)

**Fix (Git-style):**
- Write objects first
- Then update references atomically
- Use locking + delayed deletion (grace period)

---

## Conclusion
This project helped me understand how version control systems manage objects, track changes, and maintain history efficiently.
