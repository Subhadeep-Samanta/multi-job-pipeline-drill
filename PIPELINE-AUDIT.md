# Pipeline Audit

## Lint

### What it is supposed to do
The lint job checks the project for code-quality and linting errors.

### Current problem
The lint job does not have a `timeout-minutes` value, so it does not have the required execution timeout.

### Correct fix
Add `timeout-minutes: 10` to the lint job.

---

## Unit Tests

### What it is supposed to do
The unit-tests job runs the project's unit test suite.

### Current problem
The job has no `needs:` dependency, so it can start before the lint job finishes. It also does not have the required timeout.

### Correct fix
Make unit-tests depend on lint using `needs: lint` and add `timeout-minutes: 15`.

---

## Build

### What it is supposed to do
The build job creates the production build, including the `dist/` directory.

### Current problem
The build job does not depend on lint and does not upload the generated `dist/` directory as an artifact. It also does not have the required timeout.

### Correct fix
Make build depend on lint, add `timeout-minutes: 20`, and upload `dist/` using `actions/upload-artifact@v4` with the artifact name `app-build`.

---

## Integration Tests

### What it is supposed to do
The integration-tests job runs integration tests using the application build produced by the build job.

### Current problem
The job does not depend on the build job and does not download the `dist/` artifact. It also does not have the required timeout.

### Correct fix
Make integration-tests depend on build, download the `app-build` artifact using `actions/download-artifact@v4`, and add `timeout-minutes: 30`.

---

## Deploy Staging

### What it is supposed to do
The deploy-staging job deploys the tested application to the staging environment.

### Current problem
The job does not wait for both unit-tests and integration-tests. It also has no condition restricting deployment to the main branch and does not have the required timeout.

### Correct fix
Add `needs: [unit-tests, integration-tests]`, add the condition `if: github.ref == 'refs/heads/main'`, and set `timeout-minutes: 15`.

---

## Deploy Production

### What it is supposed to do
The deploy-production job deploys the application to the production environment after staging succeeds.

### Current problem
The job does not depend on deploy-staging. It also has no main-branch condition and no required timeout.

### Correct fix
Add `needs: deploy-staging`, add `if: github.ref == 'refs/heads/main'`, and set `timeout-minutes: 15`.

---

## Notify

### What it is supposed to do
The notify job sends a pipeline completion notification.

### Current problem
The job does not use `if: always()`, so it may be skipped when an earlier job fails. It also does not have a timeout.

### Correct fix
Add `if: always()` and `timeout-minutes: 10` so the notification job runs regardless of the pipeline result.