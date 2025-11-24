# 🚀 Quick Start Checklist - Snyk Integration

**Goal:** Get Snyk security scanning working in 30 minutes!

---

## ✅ Phase 1: Setup (10 minutes)

### Task 1: Create Snyk Account

- [ ] Go to https://snyk.io
- [ ] Click "Sign up for free"
- [ ] Choose: **Sign up with GitHub** ← Recommended!
- [ ] Authorize Snyk to access GitHub

### Task 2: Get Your API Token

- [ ] In Snyk: Click your profile icon → Account Settings
- [ ] Go to: General → Auth Token
- [ ] Click "Show" to reveal token
- [ ] **Copy the token** (looks like: `12345678-abcd-1234-efgh-1234567890ab`)

### Task 3: Add Secret to GitHub

- [ ] Go to: https://github.com/Wangpos/cicd-demo-swe302-p4a
- [ ] Click: Settings → Secrets and variables → Actions
- [ ] Click: "New repository secret"
- [ ] Name: `SNYK_TOKEN`
- [ ] Value: [paste your token]
- [ ] Click: "Add secret"
- [ ] ✅ Verify it appears in the list

---

## ✅ Phase 2: Test Basic Setup (10 minutes)

### Task 4: Trigger Workflow

```bash
# Make a small change to trigger the workflow
cd /Users/tsheringwangpodorji/Desktop/cicd-demo_swe302_p4

# Edit README
echo "## Testing Snyk Security Scanning" >> README.md

# Commit and push
git add README.md
git commit -m "test: trigger Snyk workflow"
git push origin main
```

### Task 5: Watch It Run

- [ ] Go to: https://github.com/Wangpos/cicd-demo-swe302-p4a/actions
- [ ] Click on the latest workflow run
- [ ] Wait for both jobs to complete:
  - ✅ "build and test"
  - ✅ "SA scan using snyk"
- [ ] If green: Success! If red: Check logs for errors

### Task 6: Check Results

- [ ] Go to your Snyk dashboard: https://app.snyk.io
- [ ] Look for your repository in the projects list
- [ ] Click on it to see any vulnerabilities found

---

## ✅ Phase 3: Enhance Workflow (10 minutes)

### Task 7: Add Improvements to maven.yml

Open `.github/workflows/maven.yml` and make these changes:

#### Change 1: Update checkout version

```yaml
# Find this line in the security job:
- uses: actions/checkout@master

# Change it to:
- uses: actions/checkout@v4
```

#### Change 2: Add JDK setup

```yaml
# Add these steps AFTER checkout in the security job:
- name: Set up JDK 17
  uses: actions/setup-java@v4
  with:
    java-version: "17"
    distribution: "temurin"
    cache: maven

- name: Build project
  run: mvn clean compile -DskipTests
```

#### Change 3: Add severity threshold

```yaml
# Find the Snyk step and add 'with:' section:
- name: Run Snyk to check for vulnerabilities
  uses: snyk/actions/maven@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  with:
    args: --severity-threshold=high --sarif-file-output=snyk.sarif
```

#### Change 4: Add SARIF upload

```yaml
# Add this new step AFTER the Snyk scan:
- name: Upload Snyk results to GitHub Security
  uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: snyk.sarif
```

### Task 8: Commit Your Changes

```bash
git add .github/workflows/maven.yml
git commit -m "feat: enhance Snyk security scanning with SARIF upload"
git push origin main
```

### Task 9: Verify Enhanced Workflow

- [ ] Check GitHub Actions for new workflow run
- [ ] Verify all steps complete successfully
- [ ] Go to: Security tab → Code scanning
- [ ] ✅ You should see Snyk results!

---

## ✅ Phase 4: Test with a Vulnerability (Optional)

### Task 10: Add a Vulnerable Dependency

Edit `pom.xml` and add this before `</dependencies>`:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.8</version>
</dependency>
```

```bash
git add pom.xml
git commit -m "test: add vulnerable dependency"
git push origin main
```

### Task 11: See Snyk Detect It

- [ ] Watch workflow run in Actions
- [ ] ✅ Snyk should report vulnerabilities
- [ ] Check Security tab for details
- [ ] Read the vulnerability report

### Task 12: Fix the Vulnerability

Update the version in `pom.xml`:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>
</dependency>
```

```bash
git add pom.xml
git commit -m "fix: upgrade jackson-databind to secure version"
git push origin main
```

- [ ] ✅ Verify workflow passes now

---

## 🎉 Success Checklist

You're done when you can check all these boxes:

- [ ] Snyk account created and connected to GitHub
- [ ] SNYK_TOKEN secret configured in GitHub
- [ ] Basic workflow runs successfully
- [ ] Enhanced workflow with SARIF upload working
- [ ] Can view results in GitHub Security tab
- [ ] Understand how to add/fix vulnerabilities
- [ ] Can read vulnerability reports

---

## 🆘 Troubleshooting

### Problem: "SNYK_TOKEN is not set"

**Solution:**

1. Check Settings → Secrets → Actions
2. Verify secret name is exactly: `SNYK_TOKEN`
3. Re-create the secret if needed

### Problem: Workflow fails on Snyk step

**Solution:**

1. This is NORMAL if vulnerabilities are found
2. Check the logs to see what was found
3. Either fix the vulnerabilities OR lower the threshold

### Problem: SARIF upload fails

**Solution:**

1. Make sure Snyk step includes: `--sarif-file-output=snyk.sarif`
2. Ensure upload step has: `if: always()`
3. Check that file path is correct: `snyk.sarif`

### Problem: No results in Security tab

**Solution:**

1. Wait a few minutes for processing
2. Ensure SARIF upload step completed successfully
3. Check you're looking at the right repository
4. Refresh the page

---

## 📚 Next Steps

After completing this checklist:

1. **Read the full guide:** `STEP_BY_STEP_GUIDE.md`
2. **Review practical4.md:** Understand all the concepts
3. **Explore Snyk dashboard:** Learn about all features
4. **Try advanced features:** Scheduled scans, monitoring, etc.

---

## 💡 Quick Commands Reference

```bash
# Navigate to project
cd /Users/tsheringwangpodorji/Desktop/cicd-demo_swe302_p4

# Check workflow file
cat .github/workflows/maven.yml

# Make a change and push
git add .
git commit -m "your message"
git push origin main

# View recent commits
git log --oneline -5

# Check current branch
git branch

# Pull latest changes
git pull origin main
```

---

## 🔗 Important Links

- **Your Repository:** https://github.com/Wangpos/cicd-demo-swe302-p4a
- **GitHub Actions:** https://github.com/Wangpos/cicd-demo-swe302-p4a/actions
- **Security Tab:** https://github.com/Wangpos/cicd-demo-swe302-p4a/security
- **Snyk Dashboard:** https://app.snyk.io
- **Snyk Docs:** https://docs.snyk.io

---

**Time to Complete:** ~30 minutes  
**Difficulty:** Beginner  
**Prerequisites:** GitHub account, basic Git knowledge

---

_Start with Phase 1 and work your way through each task!_
