sheet -> https://git-scm.com/cheat-sheet 
https://git-scm.com/cheat-sheet.pdf 

---

### **Scenario 1: Resolving Merge Conflicts**

**Description:**  
You and your teammate are working on the same file in different branches. Both of your changes overlap and cause a conflict when merging. Solve the conflict manually to ensure all changes are properly incorporated.

**Tasks:**  
1. Create a new branch `feature-one`. 2. Modify the same section of `index.html` file in `main` branch and `feature-one` branch. 3. Merge `feature-one` into `main` and resolve the conflict manually.  
2. Commit the resolution with a proper message.

---

### **Scenario 2: Handling Detached HEAD**

**Description:**  
You switch to a specific commit using its commit hash. You realize you're in a detached HEAD state and need to exit it while retaining your changes.

**Tasks:**  
1. Use `git log` to find a specific commit hash and switch to it. 2. Make some changes to a file (e.g., `README.md`) in the detached HEAD state. 3. Create a new branch `save-detached` from this state to preserve the changes.

---

### **Scenario 3: Rewriting History with Rebase**

**Description:**  
Your team asks you to rewrite commits in a branch to clean up the commit history before merging.

**Tasks:**  
1. Create a new branch `feature-cleanup` and make 3 separate commits with dummy changes. 2. Use `git rebase -i` to squash all commits into a single commit with a new message. 3. Merge `feature-cleanup` into `main`.

**Also Check**

## 1. git commit --amend

### **Scenario 4: Undoing Mistakes**

**Description:**  
You accidentally committed a sensitive file into the repository. Remove the file from the Git history entirely while ensuring it does not appear in any previous commits.

**Tasks:**  
1. Add and commit a sensitive file `config.json` to the repository. 2. Use `git filter-repo` or `git filter-branch` to remove the sensitive file completely. 3. Verify the file no longer exists in the repository's history.

---

### **Scenario 5: Handling Staging Area**

**Description:**  
You accidentally staged changes that you don't want to commit. Unstage those changes or selectively stage only the desired files while retaining your work in the working directory.

**Tasks:**  
1. Modify two files: `app.js` and `style.css`. 2. Stage both files using `git add .`. 3. Unstage `style.css` while keeping the changes in the working directory. 4. Commit only `app.js` and leave the remaining file unstaged.

---

### **Scenario 6: Recovering Deleted Branches**

**Description:**  
You accidentally delete a branch from your local repository. Recover the branch using Git history.

**Tasks:**  
1. Create a branch `experimental` and make a few commits. 2. Delete the `experimental` branch locally. 3. Recover the branch using Git command (`git reflog`) and switch back to it.

---

Sure! Here are some more Git and GitHub workflow scenarios focused on topics like force-pushing, squashing commits, and cherry-picking:

---

### **Scenario 7: Force Push to Update Remote History**

**Description:**  
You overwrite the history of a branch and want to push the changes to the remote repository. Use a force push to update the remote branch after rewriting your local history.

**Tasks:**  
1. Create and push a branch `force-update` to the remote repository with 3 commits.  
2. Locally, use an interactive rebase (`git rebase -i`) to rewrite the commit history (e.g., edit messages or squash commits).  
3. Push the rewritten history to the remote branch using `git push --force`.  
4. Verify on the remote repository (e.g., GitHub) that the commit history reflects the changes.

---

### **Scenario 8: Squashing Commits Before a Pull Request**

**Description:**  
Your branch contains multiple commits, and your team requires a clean commit history before merging a Pull Request, with all related changes in a single commit.

**Tasks:**  
1. Create a branch `squash-feature` and make 4 separate commits while working on a feature.  
2. Create a Pull Request but before merging, squash all 4 commits into one commit locally.  
3. Force push the squashed commit (`git push --force`) to update the Pull Request.  
4. Document the steps you followed to perform the squash.

---

### **Scenario 9: Cherry-Picking Specific Commits to Another Branch**

**Description:**  
You have some changes in a branch that you want to replicate on another branch without merging or rebasing. Use `git cherry-pick` to copy over specific changes.

**Tasks:**  
1. Create two branches: `feature-branch` and `hotfix-branch`.  
2. In `feature-branch`, create 3 commits with different changes to `file1.txt`, `file2.txt`, and `file3.txt`. Push this branch.  
3. Switch to `hotfix-branch` and cherry-pick only the commit related to `file2.txt` from `feature-branch`.  
4. Verify that only the desired commit has been applied to `hotfix-branch`.

---

### **Scenario 10: Accidentally Force Push and Recover Changes**

**Description:**  
You accidentally used `git push --force` and overwrote changes on the remote repository, causing you to lose your previous commits. Recover the deleted commits using `git reflog` and restore them.

**Tasks:**  
1. Create a branch `force-error` with 3 commits and push it to the remote.  
2. Use `git reset --hard` to move back to the 1st commit and intentionally force-push it to overwrite the other two commits on the remote.  
3. Use `git reflog` to find the lost commits.  
4. Restore all the commits using the `git reset` or `git cherry-pick` commands.
--- 
- **When to rebase vs merge**
    
    - Rebase: local feature branches before pushing (clean linear history)
        
    - Merge: public branches, integrating long-lived branches, preserving exact timeline
        
- **How to recover from a bad rebase**
    
    - `git reflog` → find old HEAD → `git reset --hard <commit>`
        
- **How to split a commit**
    
    - `git rebase -i` → mark commit as `edit` → `git reset HEAD^` → `git add -p` → `git commit` → `git rebase --continue`
        
- **How to move a commit to another branch**
    
    - `git cherry-pick` → `git reset --hard HEAD~1` on source
        
- **How to find when a line was deleted**
    
    - `git log -S"deleted line" -- <file>` or `git blame --reverse <file>`
--- 

neiche commands  of git : 
- --amend
- reflog -> `git log` shows the official project history, `git reflog` shows the history of where your local pointer has been, including commits that are no longer part of any branch. 
- filter-repo
- filter-branch
- bisect 
- cherry pick
---

verisoning nomenclature

1. SEMVER naming conventions. 
		MAJOR.MINOR.PATCH semantic versioning. 
2. CalVer 
		this is the calender versioning. 
3. 
#### 

