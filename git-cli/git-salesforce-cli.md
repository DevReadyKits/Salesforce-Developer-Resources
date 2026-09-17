# Git + Salesforce CLI Basics

A practical collection of Git and Salesforce CLI commands for everyday Salesforce development.

This guide focuses on the commands you are most likely to use while working with source control, Salesforce projects, sandboxes, and deployments.

---

# Git Basics

## Check Your Working Tree

Before switching branches, pulling changes, or preparing a commit, check the current state of your repository.

```bash
git status
```

This shows:

- modified files
- staged files
- untracked files
- current branch

---

## Inspect Changes

See changes that have not been staged yet:

```bash
git diff
```

See changes already staged for the next commit:

```bash
git diff --staged
```

A good habit is to review your diff before every commit.

---

## Create a Branch

Create and switch to a new branch:

```bash
git switch -c feature/my-change
```

Switch to an existing branch:

```bash
git switch main
```

Use meaningful branch names such as:

```text
feature/customer-validation
fix/opportunity-trigger
chore/update-api-version
```

---

## Stage Changes

Stage a specific file:

```bash
git add force-app/main/default/classes/MyClass.cls
```

Stage multiple selected files:

```bash
git add file1 file2 file3
```

If you want more control over what gets staged:

```bash
git add -p
```

This allows you to review changes in smaller sections before adding them to the commit.

---

## Commit Changes

Create a commit:

```bash
git commit -m "fix: bulkify account trigger"
```

Useful commit prefixes include:

```text
feat:
fix:
docs:
test:
refactor:
chore:
```

Example:

```bash
git commit -m "feat: add customer validation"
```

---

## Pull Changes

Fetch and integrate remote changes:

```bash
git pull
```

If your team prefers a rebase-based workflow:

```bash
git pull --rebase
```

Always run:

```bash
git status
```

before pulling so you know whether you have local changes.

---

## Push a Branch

Push a new branch for the first time:

```bash
git push -u origin feature/my-change
```

After the upstream branch exists:

```bash
git push
```

---

# Salesforce CLI Basics

Salesforce CLI commands use the `sf` command.

Check that Salesforce CLI is installed:

```bash
sf --version
```

---

## Log In to a Sandbox

```bash
sf org login web \
  --alias UAT \
  --instance-url https://test.salesforce.com
```

This opens the Salesforce authentication flow in your browser.

---

## Log In to Production

```bash
sf org login web \
  --alias PROD \
  --instance-url https://login.salesforce.com
```

Use clear aliases such as:

```text
DEV
QA
UAT
PROD
```

This makes commands easier to read and reduces the risk of using the wrong environment.

---

## List Authorized Orgs

```bash
sf org list
```

Use this before sensitive operations to confirm which environments are available.

---

## Inspect an Org

```bash
sf org display --target-org UAT
```

This helps confirm the organization associated with an alias.

---

## Open an Org

```bash
sf org open --target-org UAT
```

Salesforce opens the org directly in your browser.

---

# Retrieve Metadata

## Retrieve a Specific Apex Class

```bash
sf project retrieve start \
  --metadata ApexClass:MyClass \
  --target-org UAT
```

---

## Retrieve Using package.xml

```bash
sf project retrieve start \
  --manifest manifest/package.xml \
  --target-org UAT
```

After retrieving metadata, always review:

```bash
git status
```

and:

```bash
git diff
```

Do not commit unrelated metadata just because it was retrieved.

---

# Deploy Metadata

## Deploy Using package.xml

```bash
sf project deploy start \
  --manifest manifest/package.xml \
  --target-org UAT
```

---

## Deploy a Specific Directory

```bash
sf project deploy start \
  --source-dir force-app/main/default/classes \
  --target-org UAT
```

Use targeted deployments whenever possible so you understand exactly what is being changed.

---

# Validate Before Deploying

For production releases, validation can help detect problems before the real deployment.

Example:

```bash
sf project deploy validate \
  --manifest manifest/package.xml \
  --test-level RunLocalTests \
  --target-org PROD
```

Validation can catch issues such as:

- Apex compilation errors
- failing tests
- missing dependencies
- metadata problems

---

# Run Apex Tests

Run a specific test class:

```bash
sf apex run test \
  --class-names MyClassTest \
  --result-format human \
  --wait 20 \
  --target-org UAT
```

Include code coverage information:

```bash
sf apex run test \
  --class-names MyClassTest \
  --code-coverage \
  --result-format human \
  --wait 20 \
  --target-org UAT
```

---

# Execute Anonymous Apex

Create a file such as:

```text
scripts/apex/test.apex
```

Example content:

```apex
List<Account> accounts = [
    SELECT Id, Name
    FROM Account
    LIMIT 5
];

System.debug(accounts);
```

Execute it with:

```bash
sf apex run \
  --file scripts/apex/test.apex \
  --target-org UAT
```

---

# Query Salesforce Data

Run a SOQL query directly from the terminal:

```bash
sf data query \
  --query "SELECT Id, Name FROM Account LIMIT 20" \
  --target-org UAT
```

Another example:

```bash
sf data query \
  --query "SELECT Id, Name, IsActive FROM User WHERE IsActive = true" \
  --target-org UAT
```

---

# Debug Logs

Tail Apex logs:

```bash
sf apex tail log --target-org UAT
```

List available Apex logs:

```bash
sf apex list log --target-org UAT
```

Logs can help investigate:

- exceptions
- trigger execution
- SOQL usage
- DML usage
- Flow execution
- callouts
- governor limit consumption

---

# A Simple Development Workflow

A typical workflow might look like:

```text
1. Pull latest changes
        ↓
2. Create a branch
        ↓
3. Retrieve required metadata
        ↓
4. Make your changes
        ↓
5. Run tests
        ↓
6. Review git diff
        ↓
7. Commit
        ↓
8. Push
        ↓
9. Validate
        ↓
10. Deploy
```

Example:

```bash
git switch main
git pull

git switch -c fix/account-trigger

sf project retrieve start \
  --metadata ApexClass:AccountTriggerHandler \
  --target-org UAT

git status
git diff

git add force-app/main/default/classes/AccountTriggerHandler.cls

git commit -m "fix: bulkify account trigger"

git push -u origin fix/account-trigger
```

---

# Quick Git Checklist

Before pushing:

- [ ] Run `git status`
- [ ] Review `git diff`
- [ ] Confirm only intended files changed
- [ ] Avoid committing secrets or credentials
- [ ] Use a meaningful commit message
- [ ] Make sure the branch contains only related work

---

# Quick Salesforce CLI Checklist

Before deploying:

- [ ] Confirm the target org
- [ ] Review the metadata being deployed
- [ ] Check dependencies
- [ ] Run relevant Apex tests
- [ ] Review permissions when applicable
- [ ] Confirm environment-specific configuration
- [ ] Validate before production deployment

---

## DevReady Tip

Never run a retrieve, deploy, or data command against an environment just because the alias looks familiar.

Verify it first:

```bash
sf org display --target-org PROD
```

Especially before production operations.

---

## Want the Complete Toolkit?

The **Salesforce Developer Survival Kit** goes further with:

- a complete Salesforce Developer Survival Guide
- SOQL Cheat Sheet
- Apex Bulkification Checklist
- Salesforce Debugging Checklist
- Salesforce Deployment Checklist
- full Git + Salesforce CLI Cheat Sheet
- Practical Apex & SOQL Code Examples
- downloadable Apex, Trigger, Test, Callout, and SOQL source files

[Get the Salesforce Developer Survival Kit](https://payhip.com/DevReadyKits)

---

## About DevReady Kits

**DevReady Kits** creates practical resources for real-world software development.

**Code. Debug. Test. Deploy. Learn.**

DevReady Kits is an independent resource provider and is not affiliated with, endorsed by, or sponsored by Salesforce.
