---
description: Morning startup routine for navigator-frontend-next
---

// turbo-all

1. Navigate to the project directory
   cd /home/jesuslara/proyectos/navigator/navigator-frontend-next

2. Check for local modifications
   git status

3. Pull latest changes from remote
   git pull --no-rebase

4. Install new dependencies
   npm install

5. Verify code integrity (Svelte Check)
   npm run check || echo "WARNING: Svelte check failed with errors"

6. Show current HEAD commit and status
   echo "=== Current Commit ===" && git log -1 --format='%h - %s (%cr) <%an>' && echo "=== Status ===" && git status -s
