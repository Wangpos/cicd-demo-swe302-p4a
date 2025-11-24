# Step-by-Step Guide: Implementing Snyk Security Scanning

## 🎯 Learning Objectives

By the end of this guide, you will understand:

1. How to set up Snyk account and get API tokens
2. How GitHub Actions workflows work
3. How to configure security scanning
4. How to interpret and fix vulnerabilities
5. How to implement advanced security features

---

## 📋 **STEP 1: Understanding Your Current Setup** (5 minutes)

### What You Have Now:

Let's check what's already in your project:

```bash
# Navigate to your project
cd /Users/tsheringwangpodorji/Desktop/cicd-demo_swe302_p4

# Check the current workflow
cat .github/workflows/maven.yml
```

### What This Workflow Does:

```yaml
jobs:
  test: # Job 1: Runs unit tests
  security: # Job 2: Scans for security vulnerabilities
```

**Key Concept:** GitHub Actions runs these jobs automatically when you push code or create a pull request.

### ✅ Action Items:

- [ ] Open `.github/workflows/maven.yml` in VS Code
- [ ] Read through the file and identify the two jobs
- [ ] Notice the `SNYK_TOKEN` secret being used

---

## 📋 **STEP 2: Setting Up Snyk Account** (10 minutes)

### Why Snyk?

Snyk is a security tool that:

- 🔍 Scans your code for vulnerabilities
- 📦 Checks dependencies (libraries you use)
- 🐳 Scans Docker containers
- 💡 Suggests fixes automatically

### Create Your Snyk Account:

1. **Visit Snyk Website:**

   ```
   https://snyk.io
   ```

2. **Sign Up (Choose ONE method):**

   - ✅ **Recommended:** Sign up with GitHub (easier integration)
   - Alternative: Use email/password

3. **Connect GitHub:**
   - After signing up, go to "Integrations" → "GitHub"
   - Click "Connect to GitHub"
   - Authorize Snyk to access your repositories

### Get Your API Token:

1. **Navigate to Settings:**

   ```
   Click your profile icon → Account Settings
   ```

2. **Find API Token:**

   ```
   Go to: General → Auth Token
   Click "Show" to reveal your token
   ```

3. **Copy the Token:**
   ```
   It looks like: 12345678-abcd-1234-efgh-1234567890ab
   ```

⚠️ **IMPORTANT:** Keep this token secret! Never commit it to your code.

### ✅ Action Items:

- [ ] Create Snyk account
- [ ] Connect your GitHub account
- [ ] Copy your API token (save it temporarily in a secure note)

---

## 📋 **STEP 3: Configuring GitHub Secrets** (5 minutes)

### What Are GitHub Secrets?

GitHub Secrets are secure variables that:

- Store sensitive information (like API tokens)
- Are encrypted and hidden
- Can be used in GitHub Actions workflows

### Add Your Snyk Token:

1. **Go to Your Repository on GitHub:**

   ```
   https://github.com/Wangpos/cicd-demo-swe302-p4a
   ```

2. **Navigate to Settings:**

   ```
   Click "Settings" tab (top navigation)
   ```

3. **Access Secrets:**

   ```
   Left sidebar: Secrets and variables → Actions
   ```

4. **Create New Secret:**

   ```
   Click "New repository secret"

   Name: SNYK_TOKEN
   Value: [paste your token from Step 2]

   Click "Add secret"
   ```

### Verify It Worked:

You should see `SNYK_TOKEN` listed with a green checkmark ✅

### ✅ Action Items:

- [ ] Add SNYK_TOKEN to GitHub repository secrets
- [ ] Verify the secret appears in the list

---

## 📋 **STEP 4: Understanding the Basic Workflow** (15 minutes)

### Current Workflow Analysis:

Open `.github/workflows/maven.yml` and let's break it down:

#### **Part 1: When Does It Run?**

```yaml
on:
  push:
    branches: ["master"] # Runs when you push to master
  pull_request:
    branches: ["master"] # Runs when you create a PR
```

#### **Part 2: The Test Job**

```yaml
jobs:
  test:
    name: build and test
    runs-on: ubuntu-latest # Uses Ubuntu server

    steps:
      - uses: actions/checkout@v4 # Downloads your code
      - name: Set up JDK 17 # Installs Java 17
        uses: actions/setup-java@v4
      - name: Perform the Unit Test Cases
        run: mvn -B test # Runs tests
```

**What happens here:**

1. GitHub creates a virtual Ubuntu machine
2. Downloads your repository code
3. Installs Java 17
4. Runs your unit tests with Maven

#### **Part 3: The Security Job**

```yaml
security:
  needs: test # Only runs AFTER test job succeeds
  name: SA scan using snyk
  runs-on: ubuntu-latest

  steps:
    - uses: actions/checkout@master
    - name: Run Snyk to check for vulnerabilities
      uses: snyk/actions/maven@master # Runs Snyk scanner
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }} # Uses your secret
```

**What happens here:**

1. Waits for tests to pass
2. Downloads your code again
3. Runs Snyk to scan for vulnerabilities
4. Uses your SNYK_TOKEN to authenticate

### ✅ Action Items:

- [ ] Read through maven.yml and identify each section
- [ ] Understand the flow: trigger → test → security
- [ ] Notice how secrets are referenced: `${{ secrets.SNYK_TOKEN }}`

---

## 📋 **STEP 5: Testing Your Basic Setup** (10 minutes)

### Let's Trigger the Workflow:

1. **Make a Small Change:**

   ```bash
   cd /Users/tsheringwangpodorji/Desktop/cicd-demo_swe302_p4

   # Edit README to trigger workflow
   echo "Testing Snyk integration" >> README.md

   # Commit and push
   git add README.md
   git commit -m "test: trigger Snyk workflow"
   git push origin main
   ```

2. **Watch the Workflow Run:**

   ```
   Go to: https://github.com/Wangpos/cicd-demo-swe302-p4a/actions

   You should see a new workflow running!
   ```

3. **Check the Results:**
   - Click on the workflow run
   - You'll see two jobs: "build and test" and "SA scan using snyk"
   - Click on each to see the logs

### Understanding the Output:

**If Successful (Green ✅):**

```
✅ build and test - All tests passed
✅ SA scan using snyk - No critical vulnerabilities found
```

**If Failed (Red ❌):**

```
❌ SA scan using snyk - Vulnerabilities detected!
```

### ✅ Action Items:

- [ ] Make a small change and push to GitHub
- [ ] Watch the workflow run in the Actions tab
- [ ] Click through the logs to understand what's happening

---

## 📋 **STEP 6: Understanding Vulnerability Reports** (15 minutes)

### What Snyk Tells You:

When Snyk finds a vulnerability, it shows:

1. **Severity Level:**

   - 🔴 **Critical** - Fix immediately!
   - 🟠 **High** - Fix soon
   - 🟡 **Medium** - Fix when possible
   - 🟢 **Low** - Nice to fix

2. **Vulnerability Details:**

   ```
   Package: com.fasterxml.jackson.databind
   Version: 2.9.8
   Issue: Remote Code Execution
   CVE: CVE-2019-12345
   ```

3. **How to Fix:**
   ```
   Upgrade to version 2.15.2 or higher
   ```

### Example Vulnerability Report:

```
✗ High severity vulnerability found in spring-core
  Description: Improper Input Validation

  Path: pom.xml
  From: spring-boot-starter-web@3.1.2 > spring-core@6.0.11

  Remediation:
  ✓ Upgrade spring-boot-starter-web to 3.1.3
```

### ✅ Action Items:

- [ ] Go to your Snyk dashboard: https://app.snyk.io
- [ ] Look for your repository
- [ ] Review any vulnerabilities found
- [ ] Read the descriptions and understand the risks

---

## 📋 **STEP 7: Implementing Enhanced Security** (20 minutes)

### Why Enhance the Workflow?

The basic workflow only does dependency scanning. Let's add:

- ✅ Severity thresholds (don't fail on low-severity issues)
- ✅ SARIF upload (show results in GitHub Security tab)
- ✅ Code scanning (find security issues in YOUR code)
- ✅ Monitoring (track vulnerabilities over time)

### Let's Upgrade the Workflow:

I'll show you the improvements step by step:

#### **Improvement 1: Add Severity Threshold**

**Before:**

```yaml
- name: Run Snyk to check for vulnerabilities
  uses: snyk/actions/maven@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
```

**After:**

```yaml
- name: Run Snyk to check for vulnerabilities
  uses: snyk/actions/maven@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  with:
    args: --severity-threshold=high # Only fail on high/critical
```

**Why:** This prevents the build from failing on low-priority issues.

#### **Improvement 2: Upload Results to GitHub Security**

**Add this step:**

```yaml
- name: Upload Snyk results to GitHub Security
  uses: github/codeql-action/upload-sarif@v3
  if: always() # Run even if Snyk finds vulnerabilities
  with:
    sarif_file: snyk.sarif
```

**Why:** This shows vulnerabilities in your GitHub Security tab for easy tracking.

#### **Improvement 3: Add JDK Setup**

**Add this before Snyk scan:**

```yaml
- name: Set up JDK 17
  uses: actions/setup-java@v4
  with:
    java-version: "17"
    distribution: "temurin"
    cache: maven

- name: Build project
  run: mvn clean compile -DskipTests
```

**Why:** Snyk needs the project compiled to do accurate scanning.

### ✅ Action Items:

- [ ] Compare basic vs enhanced workflow
- [ ] Understand why each improvement is useful
- [ ] Decide which features you want to implement

---

## 📋 **STEP 8: Creating a .snyk Configuration File** (10 minutes)

### What is .snyk File?

The `.snyk` file lets you:

- Ignore specific vulnerabilities (if you accept the risk)
- Configure language-specific settings
- Set project policies

### Your Current .snyk File:

You already have one! Let's understand it:

```yaml
version: v1.0.0

# Ignore specific vulnerabilities
ignore:
  # Example format:
  # 'SNYK-JAVA-ORGAPACHECOMMONS-1234567':
  #   - '*':
  #       reason: 'False positive - not exploitable'
  #       expires: '2024-12-31T23:59:59.999Z'
```

### When to Use Ignore Rules:

✅ **Good Reasons to Ignore:**

- False positive (not actually exploitable in your context)
- Test-only dependency
- Vulnerability doesn't affect your usage
- Fix is not available yet (with expiration date)

❌ **Bad Reasons to Ignore:**

- "I don't want to fix it"
- "It's too much work"
- Without understanding the risk

### Example: Ignoring a Test Dependency Issue:

```yaml
ignore:
  "SNYK-JAVA-COMGITHUBJAVAFAKER-123456":
    - "*":
        reason: "JavaFaker is only used in tests, not production"
        expires: "2025-12-31T23:59:59.999Z"
```

### ✅ Action Items:

- [ ] Open your `.snyk` file
- [ ] Read the example configurations
- [ ] Don't add ignore rules yet - understand them first!

---

## 📋 **STEP 9: Fixing a Vulnerability** (20 minutes)

### Let's Practice with a Real Example:

#### Step 9.1: Add a Vulnerable Dependency

Edit `pom.xml` and add this BEFORE the closing `</dependencies>` tag:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.8</version>  <!-- Intentionally OLD version -->
</dependency>
```

#### Step 9.2: Commit and Push

```bash
git add pom.xml
git commit -m "test: add vulnerable dependency for learning"
git push origin main
```

#### Step 9.3: Watch Snyk Detect It

1. Go to GitHub Actions
2. Wait for the workflow to complete
3. You should see Snyk report vulnerabilities!

#### Step 9.4: Fix the Vulnerability

Update the version in `pom.xml`:

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.15.2</version>  <!-- UPDATED to secure version -->
</dependency>
```

#### Step 9.5: Verify the Fix

```bash
git add pom.xml
git commit -m "fix: upgrade jackson-databind to secure version"
git push origin main
```

Watch the workflow run again - it should pass! ✅

### ✅ Action Items:

- [ ] Add the vulnerable dependency
- [ ] See Snyk detect the vulnerability
- [ ] Update to the secure version
- [ ] Verify the issue is resolved

---

## 📋 **STEP 10: Understanding Advanced Features** (15 minutes)

### Feature 1: Scheduled Scans

**Why:** Dependencies get new vulnerabilities discovered over time.

**How to add:**

```yaml
on:
  push:
    branches: ["master"]
  pull_request:
    branches: ["master"]
  schedule:
    - cron: "0 2 * * 1" # Every Monday at 2 AM
```

### Feature 2: Matrix Strategy (Multiple Scan Types)

**Why:** Scan dependencies AND your code separately.

**How it works:**

```yaml
strategy:
  matrix:
    scan-type: [dependencies, code] # Run job twice

steps:
  - name: Scan dependencies
    if: matrix.scan-type == 'dependencies'
    # ... dependency scan

  - name: Scan code
    if: matrix.scan-type == 'code'
    # ... code scan
```

### Feature 3: Monitoring

**Why:** Track vulnerabilities in your production application.

**How to add:**

```yaml
- name: Monitor with Snyk
  uses: snyk/actions/maven@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  with:
    command: monitor # Instead of 'test'
```

This creates a project in your Snyk dashboard that continuously monitors for new vulnerabilities.

### ✅ Action Items:

- [ ] Understand what each advanced feature does
- [ ] Think about which ones would be useful for your project
- [ ] Review the enhanced-security.yml file to see them in action

---

## 📋 **STEP 11: Choosing Your Implementation Path** (Decision Point)

You have two options:

### Option A: Keep It Simple (Recommended for Learning)

- Use the basic `maven.yml` workflow
- Add small enhancements one at a time:
  1. Add severity threshold
  2. Add SARIF upload
  3. Add JDK setup in security job

**Pros:** Easier to understand, gradual learning
**Cons:** Missing some advanced features

### Option B: Use Full Enhanced Workflow

- Switch to `enhanced-security.yml`
- Get all features at once

**Pros:** Complete implementation, best security
**Cons:** More complex, harder to troubleshoot

### My Recommendation:

Start with **Option A** - enhance the basic workflow step by step. Once you understand each piece, you can switch to the full version.

---

## 📋 **STEP 12: Practical Exercise** (30 minutes)

### Exercise: Enhance Your Basic Workflow

Let's improve `maven.yml` together:

1. **Add JDK Setup to Security Job:**

Find this in `maven.yml`:

```yaml
security:
  needs: test
  name: SA scan using snyk
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@master
```

Add JDK setup:

```yaml
security:
  needs: test
  name: SA scan using snyk
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Set up JDK 17
      uses: actions/setup-java@v4
      with:
        java-version: "17"
        distribution: "temurin"
        cache: maven

    - name: Build project
      run: mvn clean compile -DskipTests
```

2. **Add Severity Threshold:**

Find the Snyk step and add `with:` section:

```yaml
- name: Run Snyk to check for vulnerabilities
  uses: snyk/actions/maven@master
  env:
    SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
  with:
    args: --severity-threshold=high --sarif-file-output=snyk.sarif
```

3. **Add SARIF Upload:**

Add a new step after Snyk scan:

```yaml
- name: Upload Snyk results to GitHub Security
  uses: github/codeql-action/upload-sarif@v3
  if: always()
  with:
    sarif_file: snyk.sarif
```

4. **Test Your Changes:**

```bash
git add .github/workflows/maven.yml
git commit -m "feat: enhance Snyk security scanning"
git push origin main
```

5. **Verify Results:**
   - Check GitHub Actions
   - Look for the new build step
   - Check Security tab for SARIF results

### ✅ Action Items:

- [ ] Make the three improvements above
- [ ] Commit and push your changes
- [ ] Verify the enhanced workflow runs successfully
- [ ] Check GitHub Security tab for results

---

## 📋 **STEP 13: Interpreting Results** (10 minutes)

### Where to Find Results:

1. **GitHub Actions Tab:**

   ```
   Shows: Workflow execution logs
   URL: /actions
   ```

2. **GitHub Security Tab:**

   ```
   Shows: Detailed vulnerability reports
   URL: /security/code-scanning
   ```

3. **Snyk Dashboard:**
   ```
   Shows: All projects, trends, recommendations
   URL: https://app.snyk.io
   ```

### Reading a Vulnerability Report:

```
Critical: Deserialization of Untrusted Data
Package: jackson-databind
Introduced by: pom.xml

Details:
- CVE-2019-12345
- CVSS Score: 9.8 (Critical)
- Published: 2019-05-01

Impact:
Remote Code Execution (RCE) - An attacker could execute
arbitrary code on your server.

Fix:
Upgrade to jackson-databind@2.15.2 or higher
```

### What Each Part Means:

- **CVE ID:** Official vulnerability identifier
- **CVSS Score:** 0-10 scale (10 = worst)
- **Impact:** What an attacker could do
- **Fix:** How to resolve the issue

### ✅ Action Items:

- [ ] Navigate to all three result locations
- [ ] Compare the information shown in each
- [ ] Practice reading vulnerability reports

---

## 📋 **STEP 14: Best Practices Checklist**

### Security Scanning Best Practices:

- ✅ **Run scans on every push/PR**
- ✅ **Set appropriate severity thresholds** (high/critical for production)
- ✅ **Review and triage vulnerabilities regularly**
- ✅ **Keep dependencies up to date**
- ✅ **Use .snyk file judiciously** (document why you ignore issues)
- ✅ **Monitor production applications** (use `snyk monitor`)
- ✅ **Educate your team** (share vulnerability reports)
- ✅ **Set up notifications** (get alerted on critical issues)
- ✅ **Schedule regular scans** (weekly security reviews)
- ✅ **Integrate with Security tab** (use SARIF uploads)

### Workflow Optimization:

- ✅ **Use caching** (Maven dependencies)
- ✅ **Run scans in parallel** (when possible)
- ✅ **Continue on error** (for reporting steps)
- ✅ **Use conditional execution** (skip unnecessary scans)

### ✅ Action Items:

- [ ] Review this checklist
- [ ] Identify which practices you're following
- [ ] Plan to implement missing ones

---

## 📋 **STEP 15: Troubleshooting Common Issues**

### Issue 1: "SNYK_TOKEN is not set"

**Symptoms:**

```
Error: Input required and not supplied: SNYK_TOKEN
```

**Solutions:**

1. Check secret exists: Settings → Secrets → Actions
2. Verify exact name: `SNYK_TOKEN` (case-sensitive)
3. Check token is valid in Snyk dashboard

---

### Issue 2: "No supported manifests found"

**Symptoms:**

```
Error: Could not detect supported target files
```

**Solutions:**

1. Ensure `pom.xml` is in repository root
2. Add build step before Snyk scan:
   ```yaml
   - name: Build project
     run: mvn clean compile
   ```

---

### Issue 3: Workflow Fails on Vulnerabilities

**Symptoms:**

```
❌ Security scan failed
Process completed with exit code 1
```

**This is NORMAL!** Snyk fails the build when it finds vulnerabilities.

**Solutions:**

1. Review the vulnerabilities
2. Fix them by updating dependencies
3. OR adjust severity threshold:
   ```yaml
   args: --severity-threshold=critical
   ```

---

### Issue 4: SARIF Upload Fails

**Symptoms:**

```
Error: SARIF file not found
```

**Solutions:**

1. Ensure Snyk outputs SARIF:
   ```yaml
   args: --sarif-file-output=snyk.sarif
   ```
2. Use `if: always()` on upload step
3. Check file exists:
   ```yaml
   - name: List files
     run: ls -la *.sarif
   ```

---

## 🎓 **Final Review and Next Steps**

### What You've Learned:

✅ How to set up Snyk account and get API tokens
✅ How to configure GitHub Secrets
✅ How GitHub Actions workflows work
✅ How to run basic security scanning
✅ How to interpret vulnerability reports
✅ How to fix vulnerabilities
✅ How to implement advanced security features
✅ Best practices for security scanning

### Your Implementation Status:

**Basic (Complete):**

- [x] Snyk account created
- [x] GitHub secrets configured
- [x] Basic workflow running
- [x] Vulnerability detection working

**Enhanced (Optional):**

- [ ] Severity thresholds configured
- [ ] SARIF upload to Security tab
- [ ] Scheduled scans
- [ ] Matrix strategy
- [ ] Monitoring enabled
- [ ] Notifications configured

### Next Steps:

1. **Immediate (This Week):**

   - Complete the practical exercise in Step 12
   - Fix any existing vulnerabilities
   - Enable SARIF upload

2. **Short-term (This Month):**

   - Add scheduled scans
   - Set up monitoring
   - Configure notifications

3. **Long-term (Ongoing):**
   - Regular vulnerability reviews
   - Keep dependencies updated
   - Expand to container scanning
   - Train team members

---

## 📚 Additional Resources

### Documentation:

- [Snyk Documentation](https://docs.snyk.io/)
- [GitHub Actions Security](https://docs.github.com/en/actions/security-guides)
- [Maven Dependency Management](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

### Learning:

- [Snyk Learn](https://learn.snyk.io/) - Free security training
- [OWASP Top 10](https://owasp.org/www-project-top-ten/) - Common vulnerabilities
- [GitHub Skills](https://skills.github.com/) - GitHub Actions tutorials

### Tools:

- [Snyk CLI](https://docs.snyk.io/snyk-cli) - Local scanning
- [Snyk IDE Plugins](https://docs.snyk.io/integrations/ide-tools) - VS Code integration
- [Dependency-Check](https://owasp.org/www-project-dependency-check/) - Alternative scanner

---

## ❓ Quick Reference Commands

### Local Testing:

```bash
# Install Snyk CLI
npm install -g snyk

# Login to Snyk
snyk auth

# Test current project
snyk test

# Monitor project
snyk monitor

# Test with threshold
snyk test --severity-threshold=high

# Output as JSON
snyk test --json
```

### Git Commands:

```bash
# Check workflow files
git diff .github/workflows/

# Push changes
git add .
git commit -m "feat: enhance security scanning"
git push origin main

# View logs
git log --oneline -10
```

### Maven Commands:

```bash
# List dependencies
mvn dependency:tree

# Update dependencies
mvn versions:display-dependency-updates

# Run tests
mvn test

# Build project
mvn clean compile
```

---

## 🎯 Success Criteria

You've successfully completed this guide when you can:

- [ ] Explain what SAST is and why it's important
- [ ] Set up a Snyk account and integrate with GitHub
- [ ] Configure GitHub Secrets securely
- [ ] Understand how GitHub Actions workflows execute
- [ ] Read and interpret vulnerability reports
- [ ] Fix vulnerabilities by updating dependencies
- [ ] Configure severity thresholds appropriately
- [ ] Upload results to GitHub Security tab
- [ ] Implement at least one advanced feature
- [ ] Troubleshoot common issues independently

---

**Congratulations! You now understand Snyk security scanning! 🎉**

Remember: Security is a continuous process, not a one-time task. Keep learning and improving!

---

_Created for NUS-ISS SWE302 Practical 4_
_Last Updated: November 24, 2025_
