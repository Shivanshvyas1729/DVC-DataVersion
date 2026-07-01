# YT-MLOPS-DVC-DataVersion
This repo impliments the idea of data versioning using DVC tool.
<details>
  <summary>why what and  how it work</summary>
  # DVC + Git Theory (Easy Notes)
<img width="1024" height="1536" alt="ChatGPT Image Jul 2, 2026, 02_33_31 AM" src="https://github.com/user-attachments/assets/56bf6495-f411-4fa1-9d6f-ed4e34227b3f" />

## Why do we need DVC?

Git is excellent for tracking **code**, but it struggles with **large datasets** because:

* Large files make the repository huge.
* Every data change creates another large version.
* Cloning and pushing become very slow.

**Solution:** Use **DVC (Data Version Control)** alongside Git.

---

# The Main Idea

Think of **Git and DVC as two systems working together.**

| Git                          | DVC                    |
| ---------------------------- | ---------------------- |
| Tracks code                  | Tracks datasets        |
| Stores text files            | Stores large data      |
| Keeps history of code        | Keeps history of data  |
| Tracks `.dvc` files (tokens) | Stores actual datasets |

> **Git never stores the actual large dataset.**
> It only stores a small reference (token) to it.

---

# Analogy 1: Locker System

Imagine you have a huge box.

### Without DVC

You carry the box everywhere.

```
Git Repository
---------------
Code
Dataset (5 GB)
More datasets
```

Problems:

* Heavy
* Slow
* Repository becomes huge

---

### With DVC

Instead of carrying the box, you keep it inside a locker.

```
Locker Room (DVC Storage)

Locker A → Dataset V1
Locker B → Dataset V2
Locker C → Dataset V3
```

Each locker has a unique **claim token (MD5 hash)**.

Example:

```
Locker A
↓

Token:
8fd9ab234c...
```

Now you only keep the token in Git.

```
Git Repository

Code
README
train.py
data.dvc  ---> Token (8fd9ab234c...)
```

Git stores only this tiny file.

---

# Analogy 2: Cupboard (From the Temple/Shoes Example)

Imagine your house has two cupboards.

### Cupboard 1 (Git)

Contains:

* Phone
* Wallet
* Bag
* Small slips (tokens)

```
Git Cupboard

Phone
Wallet
Bag
Token ID-2
```

---

### Cupboard 2 (DVC)

Contains heavy items.

```
DVC Cupboard

Shoes
Books
Clothes
Large Boxes
```

Instead of keeping shoes inside Cupboard 1, you keep only a **token**.

```
Git

Token ID-2
```

The token tells DVC:

> "The shoes are stored over there."

---

# How Versioning Works

Suppose your dataset changes over time.

```
Version 1 → Dataset D1

Version 2 → Dataset D2

Version 3 → Dataset D3

Version 4 → Dataset D4
```

DVC stores all these versions.

```
DVC Storage

D1
D2
D3
D4
```

Each dataset gets a different token.

```
D1 → Hash A

D2 → Hash B

D3 → Hash C

D4 → Hash D
```

Git stores only the corresponding token.

```
Commit 1

train.py
data.dvc
Hash A
```

```
Commit 2

train.py
data.dvc
Hash B
```

```
Commit 3

train.py
data.dvc
Hash C
```

---

# What Happens During `git commit`?

Git commits:

```
✔ train.py
✔ README.md
✔ requirements.txt
✔ data.dvc
```

Notice:

Git **does not commit**

```
❌ data.csv (5 GB)
```

Instead, `data.dvc` contains something like:

```
md5: 8fd9ab234...
size: 5GB
path: data.csv
```

This tiny file is committed.

---

# Where is the Actual Data?

The actual dataset is stored by DVC.

It can be:

* Local cache
* Google Drive
* AWS S3
* Azure Blob
* Remote server
* SSH server

Example:

```
Git Repo

train.py
data.dvc
README.md
```

↓

`data.dvc`

↓

```
Hash:
8fd9ab234...
```

↓

```
DVC Remote Storage

Dataset
```

---

# Retrieving an Older Version

Suppose six months later you want Version 2.

### Step 1

Checkout the old Git commit.

```
git checkout commit_of_v2
```

Now Git restores:

```
train.py
data.dvc
```

Inside `data.dvc` is:

```
Hash B
```

---

### Step 2

Run

```bash
dvc pull
```

DVC reads:

```
Hash B
```

Searches its storage:

```
Hash B

↓

Dataset D2
```

Downloads the exact dataset used when that commit was created.

Now both your **code and data** match perfectly.

---

# Relationship Between Git and DVC

```
               Git
        ----------------
        train.py
        app.py
        README.md
        data.dvc
             |
             |
         (MD5 Hash)
             |
             ▼
        ----------------
            DVC
        ----------------
        Dataset V1
        Dataset V2
        Dataset V3
```

Git stores the pointer.

DVC stores the actual data.

---

# Why Can't We Use DVC Without Git?

DVC needs Git because Git versions the **pointer (`.dvc` file)**.

Without Git:

* No history of which dataset belongs to which code.
* No mapping between code versions and data versions.
* Difficult to reproduce experiments.

Git provides the timeline, while DVC provides the data.

---

# Parallel Workflow

```
Edit Code
     │
     ▼
Git tracks code
     │
     ▼
Add Dataset
     │
     ▼
DVC stores dataset
     │
Creates Hash (Token)
     │
     ▼
Git commits the token (.dvc file)
     │
     ▼
Later...
     │
Git restores token
     │
     ▼
DVC reads token
     │
     ▼
Downloads exact dataset
```

---

# Key Takeaways

* **Git** is responsible for versioning **code** and **small metadata files** (`.dvc` files).
* **DVC** is responsible for storing and versioning **large datasets**.
* Every dataset version is represented by a unique **MD5 hash (token)**.
* Git stores only the **token**, not the actual dataset.
* DVC uses the token to retrieve the correct version of the dataset.
* Using Git and DVC together ensures that **the exact code and the exact data version are always reproducible**.
* **Git + DVC work in parallel**: Git manages the project history, while DVC manages the data history.
<img width="1024" height="1536" alt="ChatGPT Image Jul 2, 2026, 02_33_31 AM" src="https://github.com/user-attachments/assets/dc945a94-d928-4fa6-ab59-f61c55694324" />

</details>
<details><summary>basic code</summary>
1. Create git repo and clone it in local.
2. Create mycode.py and add code to it. (it will save a csv file to a new "data" folder)
3. Do a git add-commit-push before initializing dvc.
# pip install dvc
4. Now we do "dvc init" (creates .dvcignore, .dvc)
5. Now do "mkdir S3" (creates a new S3 directory)
6. Now we do "dvc remote add -d myremote S3"
7. Next "dvc add data/" 
   Now it will ask to do: ("git rm -r --cached 'data'" and "git commit -m "stop tracking data"")
   Because initially we were tracking data/ folder from git so now we remove it for DVC to handle.
8. Again we do "dvc add data/" (creates data.dvc) then "git add .gitignore data.dvc"
9. Now - "dvc commit" and then "dvc push"
9. Do a git add-commit-push to mark this stage as first version of data.
10. Now make changes to mycode.py to append a new row in data, check changes via "dvc status"
11. Again - - "dvc commit" and then "dvc push"
12. Then git add-commit-push (we're saving V2 of our data at this point)
13. Check dvc/git status, everything should be upto date.
14. Now repeat step 10-12 for v3 of data.
</details>
