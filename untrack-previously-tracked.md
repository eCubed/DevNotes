# Untracking Previously - Tracked Files

Sometimes, a file may have made it to the repository that we didn't intend to. When that happens,
we must perform the following steps:

1. Edit the .gitignore file and add an entry or entries that correspond[s] to the file[s] we
don't want to make it to the repo from this point onward.
2. Commit.
3. Run `git rm -r --cached .` This will remove everything from the repo. It does NOT delete any
of the files.
4. Run `git add .` This will re-add everything back to the repo, except those files stated in
the most recently-committed .gitignore.
5. Commit. Now, those files most recently unwanted will no longer make it to the repo if we
make edits and save them.