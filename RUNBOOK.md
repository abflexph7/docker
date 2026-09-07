Symptom: Job 1 (test) shows red in Actions
Causes:
  - A unit test is failing
  - Lint errors in the code
  - npm ci fails (package-lock.json out of sync)
  Fix:
  1. Click the failed job → expand the failed step → read the error
  2. Reproduce locally: npm test or npm run lint
  3. Fix the code, git push — pipeline retries automatically
  4. If npm ci fails: delete node_modules, run npm install,
     commit the new package-lock.json

Symptom: Job 2 (build) fails with "CRITICAL vulnerability found"

Causes:
  - A base image or npm dependency has a known CVE with a fix available

Fix:
  1. Read the Trivy output — it names the package, CVE ID, and fixed version
  2. Update the base image: change FROM node:20-alpine to FROM node:22-alpine
  3. Or update the npm package: npm update PACKAGE-NAME
  4. Rebuild and push — if Trivy still fails, check if a fix is available at nvd.nist.gov
  5. If no fix exists yet, add to .trivyignore:
     CVE-2024-XXXXX  # no fix available as of YYYY-MM-DD

DO NOT: set exit-code: "0" to silence the scan. Fix the vulnerability.

Symptom: Job 3 (deploy) runs for 10+ minutes then fails with
"service did not reach a steady state"
Causes:
  - New tasks start but fail health checks
  - Container crashes on startup
  - Wrong port mapping in task definition
🔒 Private Layer
Fix:
  1. ECS → production-cluster → myapp-service → Tasks tab
  2. Find a task with status STOPPED → click it → read "Stopped reason"
  3. CloudWatch → /ecs/myapp → find log stream for the failing task
  4. Read the last 20 log lines — the crash reason is usually there
Common causes and fixes:
  - "CannotPullContainerError": ECR URI wrong, or execution role missing ECR permissions
  - App crashes: missing environment variable → add to task definition
  - Health check fails: wrong port in target group, or /health returns non-200
Emergency rollback (while you investigate):
  ECS → myapp-service → Update service
  Task definition: choose the previous revision number
  Force new deployment: checked → Update

Symptom: "Error: Could not assume role" in Job 2 or Job 3
Cause:
  The IAM role trust policy condition does not match the repository path
Fix:
  1. IAM → Roles → github-actions-deploy-role → Trust relationships
  2. Find the Condition block:
     "token.actions.githubusercontent.com:sub": "repo:OWNER/REPO:*"
  3. Verify OWNER and REPO exactly match your GitHub username and repo name
     (case-sensitive)
  4. Update and retry the pipeline