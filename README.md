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
(Add screenshot here)

---

### Phase 2

#### 2A: test_tree output
(Add screenshot here)

#### 2B: Raw tree object
(Add screenshot here)

---

### Phase 3

#### 3A: init → add → status
(Add screenshot here)

#### 3B: Index file
(Add screenshot here)

---

### Phase 4

#### 4A: Commit log
(Add screenshot here)

#### 4B: Object growth
(Add screenshot here)

#### 4C: HEAD and branch
(Add screenshot here)

---

## Analysis Questions

### Q5.1 — pes checkout <branch>

- A branch is just a file inside `.pes/refs/heads/` that stores a commit hash  
- I would update `.pes/HEAD` to point to the selected branch  
- Then read the commit hash from that branch file  
- Load the commit and get its tree  
- Recreate the working directory using that tree  
- Update the index to match  

Why this is complex:
- Multiple components (HEAD, index, working directory) must stay in sync  
- Tree traversal can be recursive  
- Must avoid overwriting user changes  
- Conflict handling adds complexity  

---

### Q5.2 — Dirty working directory detection

- I would go through each file in the index  
- Compute hash of file from disk  
- Compare with stored hash in index  

If mismatch → file is modified  

Before checkout:
- Check if file is also different in target branch  
- If yes, switching would overwrite changes  

In that case:
- Abort checkout  
- Show error  

---

### Q5.3 — Detached HEAD

- HEAD directly points to a commit instead of a branch  
- Commits created here are not referenced by any branch  

So:
- They can become unreachable  
- May be deleted by garbage collection  

To recover:
- Create a new branch pointing to that commit  
- Or use commit hash to access it  

---

### Q6.1 — Garbage Collection

- Start from all branch heads  
- Traverse commits, trees, and blobs  
- Mark all reachable objects  

I would store visited hashes in a hash set  

Then:
- Scan `.pes/objects`  
- Delete unmarked objects  

Estimated:
- Around 100k commits  
- Each commit has tree + blobs  
- Total objects can be a few hundred thousand  

---

### Q6.2 — GC Race Condition

- Commit creates objects not yet referenced  
- GC runs at the same time  
- GC deletes these objects thinking they are unused  

Result:
- Commit points to missing objects  
- Repository becomes inconsistent  

Git avoids this by:
- Writing objects first  
- Updating references later  
- Using locks  
- Delaying deletion of unreachable objects  

---

## Conclusion
This project helped understand how version control systems manage objects, track changes, and maintain history efficiently.
