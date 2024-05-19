# Trunk-based development (TBD)

1. Create a new branch for your feature:
	```bash
	git checkout -b feature/my-feature
	```

2. Make your changes to the codebase and commit them:
	```bash
	git add .
	git commit -m "Implement feature XYZ"
	```

3. Fetch the latest changes from the mainline/trunk branch:
	```bash
	git checkout main
	git pull
	```

4. Merge the changes from the mainline/trunk branch into your feature branch:
	```bash
	git checkout feature/my-feature
	git merge main
	```

5. Resolve any merge conflicts if they occur. Git will indicate the conflicting files, and you can edit them manually to resolve the conflicts.

6. Test your changes locally to ensure they work correctly with the latest codebase.

7. Commit the merge changes to your feature branch:
	```bash
	git add .
	git commit -m "Merge main into feature/my-feature"
	```

8. Push your feature branch to the remote repository:
	```bash
	git push origin feature/my-feature
	```

9. If you're ready to deploy your feature, merge it into the mainline/trunk branch:
	```bash
	git checkout main
	git merge --no-ff feature/my-feature
	git push origin main
	```

10. Delete your feature branch (optional):
	```bash
	git branch -d feature/my-feature
	```

> [!NOTE]
> Remember to adapt these steps to your specific Git workflow and branch naming conventions. The key idea is to frequently merge changes from the mainline/trunk branch into your feature branch and eventually merge your feature branch back into the mainline/trunk branch when it's ready.
