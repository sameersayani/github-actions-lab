# 🚀 GitHub Actions Lab

> **Learn GitHub Actions by doing!** This repository is your hands-on guide to automating CI/CD workflows using GitHub's powerful automation platform.

---

## 📚 Table of Contents
1. [What are GitHub Actions?](#what-are-github-actions)
2. [Quick Concepts](#quick-concepts)
3. [Our Workflow: `.NET CI`](#our-workflow-net-ci)
4. [Key Components Explained](#key-components-explained)
5. [Running Tests with GitHub Actions](#running-tests-with-github-actions)
6. [Advanced Features](#advanced-features)
7. [Workflow Diagram](#workflow-diagram)

---

## 🤔 What are GitHub Actions?

**GitHub Actions** is an automation engine built directly into GitHub. Think of it as a **robot 🤖 that watches your repository and automatically performs tasks** whenever certain events happen.

### The Concept in Simple Terms:

```
📝 You push code to GitHub
           ↓
🔔 GitHub Actions detects the push
           ↓
⚙️  Automatic workflow starts
           ↓
🔍 Code is built and tested
           ↓
✅ Results are reported
```

### Real-World Analogy:
- **Without Actions:** You manually build code, run tests, check for errors *(tedious!)*
- **With Actions:** Push code → Automatic build & test → Get results instantly *(efficient!)*

---

## 💡 Quick Concepts

### **Workflows** 🔄
A workflow is a **set of automated tasks** defined in a YAML file. 
- Located in `.github/workflows/`
- Triggered by events (push, pull request, schedule, etc.)
- Contains one or more jobs

### **Jobs** 👷
Jobs are **logical groups of steps** that run in sequence or parallel.
- Each job runs on its own runner (virtual machine)
- Jobs can depend on other jobs with `needs:`

### **Steps** 📋
Individual **commands or actions** that execute sequentially.
- Can run shell commands: `run: echo "Hello!"`
- Can use pre-built actions: `uses: actions/checkout@v4`

### **Actions** 🎯
Reusable **units of code** that do specific tasks.
- Official actions: `actions/checkout`, `actions/upload-artifact`
- Community actions: Available in GitHub Marketplace
- Custom actions: Build your own!

### **Runners** 🏃
The **machines that execute workflows**.
- GitHub-hosted: `ubuntu-latest`, `windows-latest`, `macos-latest`
- Self-hosted: Run on your own servers

---

## 🔧 Our Workflow: `.NET CI`

Let's dissect the **dotnet-ci.yml** workflow in this repository:

### 📍 **Trigger Events**
```yaml
on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main
```
⚡ **This means:** The workflow runs whenever:
- Code is pushed to the `main` branch
- A pull request targets the `main` branch

### 🌍 **Environment Variables**
```yaml
env:
  DOTNET_VERSION: '8.0.x'
  BUILD_CONFIGURATION: Release
```
📌 **What's this?** Global variables used across all jobs. Like setting up your toolbox before starting work!

### 🏗️ **The Build Job**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4
```

| Step | What It Does | Why It Matters |
|------|-------------|----------------|
| **Checkout** | Clones your code to the runner | Without this, the runner has no code to build! |
| **Setup .NET** | Installs .NET 8.0 | Required to build C# projects |
| **Cache NuGet** | Saves dependencies locally | Speeds up future builds ⚡ |
| **Restore** | Downloads project dependencies | `dotnet restore` = install libraries |
| **Build** | Compiles the code | Checks for syntax errors |
| **Upload Artifact** | Saves build output | You can download it later! |

#### 🎨 Visual Flow:

```
┌──────────────────────────────────────────┐
│        BUILD JOB (ubuntu-latest)         │
├──────────────────────────────────────────┤
│ 1. 📥 Checkout                           │
│    └─> Download code from GitHub        │
│                                          │
│ 2. 🛠️ Setup .NET 8.0                     │
│    └─> Install .NET SDK                 │
│                                          │
│ 3. 💾 Cache NuGet Packages               │
│    └─> Use cached dependencies           │
│                                          │
│ 4. 🔍 Restore Dependencies               │
│    └─> dotnet restore                   │
│                                          │
│ 5. 🏗️ Build                              │
│    └─> dotnet build --configuration...  │
│                                          │
│ 6. 📦 Upload Artifact                    │
│    └─> Save compiled output              │
└──────────────────────────────────────────┘
          ✅ Build Complete!
                 │
                 ↓ (needs: build)
          🧪 TEST JOB STARTS
```

### 🧪 **The Test Job**

```yaml
test:
  needs: build          # ← Only runs AFTER build succeeds!
  runs-on: ubuntu-latest
  steps:
    - name: Run tests
      run: dotnet test --configuration Release
```

**Key Insight:** `needs: build` creates a **dependency**!
- 🚫 If build fails → tests don't run
- ✅ If build succeeds → tests run automatically

---

## 🔑 Key Components Explained

### 1️⃣ **Checkout Action**
```yaml
- uses: actions/checkout@v4
```
👉 Downloads your code from GitHub to the runner. Without this, nothing works!

### 2️⃣ **Setup .NET Action**
```yaml
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: ${{env.DOTNET_VERSION}}
```
👉 Installs the exact .NET version you need. The `${{ }}` syntax accesses variables.

### 3️⃣ **Conditional Steps**
```yaml
- name: Main branch message
  if: github.ref == 'refs/heads/main'
  run: echo "This is the main branch"
```
👉 Only runs when the condition is true. Super useful for branch-specific logic!

### 4️⃣ **Caching**
```yaml
- uses: actions/cache@v4
  with:
    path: ~/.nuget/packages
    key: ${{ runner.os }}-nuget-${{ hashFiles('**/*.csproj') }}
```
👉 Saves NuGet packages between runs. Makes builds **3-5x faster!** ⚡

### 5️⃣ **Artifacts**
```yaml
- uses: actions/upload-artifact@v4
  with:
    name: dotnet-build
    path: |
      **/bin/Release/**
```
👉 Saves the compiled output. You can download it from the Actions tab!

---

## 🧪 Running Tests with GitHub Actions

### What Happens:

```
1. Developer pushes code
         ↓
2. GitHub detects push
         ↓
3. Build job runs:
   • Checkout code
   • Setup .NET
   • Build project
         ↓
4. Test job starts (only if build succeeds):
   • Run: dotnet test --configuration Release
         ↓
5. Results appear in GitHub UI:
   ✅ All tests passed → Green checkmark
   ❌ Tests failed → Red X + error details
```

### Our Test File:
```csharp
public class UnitTest1
{
    [Fact]
    public void Test1()
    {
        Assert.True(true);  // ✅ This test passes
    }
}
```

### Where to See Results:

| Location | What You See |
|----------|-------------|
| **Pull Request Page** | Green ✅ or Red ❌ checkmark |
| **Commits Page** | Status next to each commit |
| **Actions Tab** | Detailed logs of each step |
| **Job Summary** | Duration, status, job output |

---

## 🚀 Advanced Features

### ⏰ **Scheduled Workflows**
Run workflows on a timer:
```yaml
on:
  schedule:
    - cron: '0 0 * * *'  # Run daily at midnight
```

### 🔐 **Secrets & Sensitive Data**
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```
→ Store API keys securely in GitHub Settings!

### 📤 **Matrix Builds**
Test on multiple versions:
```yaml
strategy:
  matrix:
    dotnet-version: [6.0, 7.0, 8.0]
```
→ Automatically runs build on each version!

### 🔔 **Notifications**
```yaml
- name: Notify Slack
  run: curl -X POST $SLACK_WEBHOOK
```
→ Send alerts when builds fail!

### 📊 **Status Checks**
Require workflows to pass before merging PRs:
1. Go to **Settings** → **Branch Protection Rules**
2. Require `.NET CI` to pass
3. Now PRs can't be merged if tests fail! 🛡️

---

## 📊 Workflow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                        GitHub Repository                             │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │  .github/workflows/dotnet-ci.yml                              │   │
│  │  ═══════════════════════════════════════════════════════════  │   │
│  │  on: [push to main, pull_request]                            │   │
│  └──────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                          │
                    🔔 EVENT TRIGGERED
                          │
                ┌─────────┴─────────┐
                │                   │
        ┌───────▼────────┐  ┌──────▼──────────┐
        │   BUILD JOB    │  │   BUILD JOB     │
        │  (ubuntu-latest)  (windows-latest) │
        ├────────────────┤  └─────────────────┘
        │ ✅ Checkout   │
        │ ✅ Setup .NET │
        │ ✅ Restore    │
        │ ✅ Build      │
        │ ✅ Artifacts  │
        └────────┬───────┘
                 │
         🎯 BUILD SUCCEEDS
                 │
        ┌────────▼────────┐
        │    TEST JOB     │
        │ (ubuntu-latest) │
        ├─────────────────┤
        │ ✅ Checkout    │
        │ ✅ Setup .NET  │
        │ ✅ Run Tests   │
        └────────┬────────┘
                 │
       ┌─────────┴──────────┐
       │                    │
   ✅ ALL PASS         ❌ FAILURE
       │                    │
  GREEN ✓              RED ✗
  on PR                on PR
```

---

## 🎯 Quick Reference

### Common Commands Used:

```bash
# Restore dependencies
dotnet restore

# Build project
dotnet build --configuration Release

# Run tests
dotnet test --configuration Release

# Publish artifact
dotnet publish -c Release -o ./output
```

### Common Workflow Patterns:

```yaml
# Run only on specific branches
on:
  push:
    branches: [main, develop]

# Run on schedule
on:
  schedule:
    - cron: '0 9 * * 1'  # Every Monday at 9 AM

# Manual trigger
on:
  workflow_dispatch:  # Press a button in GitHub UI!
```

---

## 🎓 Learning Path

1. **Read** this README (you are here! 📖)
2. **Explore** the `.github/workflows/dotnet-ci.yml` file
3. **Make a PR** and watch the workflow run in real-time 👀
4. **Modify** the workflow and see what changes
5. **Add** more jobs (e.g., code analysis, security scanning)
6. **Automate** your own projects! 🚀

---

## 📚 Resources

| Resource | Link |
|----------|------|
| GitHub Actions Docs | https://docs.github.com/actions |
| Workflow Syntax | https://docs.github.com/actions/using-workflows/workflow-syntax-for-github-actions |
| Actions Marketplace | https://github.com/marketplace?type=actions |
| .NET Workflows | https://github.com/actions/setup-dotnet |

---

## 🎯 Key Takeaways

✨ **GitHub Actions allows you to:**
- ✅ Automate testing on every push/PR
- ✅ Build multiple configurations in parallel
- ✅ Deploy code automatically
- ✅ Cache dependencies for speed
- ✅ Require quality gates before merging
- ✅ Notify teams about build status

🚀 **Why GitHub Actions?**
- 🏗️ Built into GitHub (no external tools needed)
- 💰 Free tier: 3,000 minutes/month per account
- 🔒 Secure (secrets management built-in)
- 📦 Large marketplace of pre-built actions
- 🎯 Powerful conditional logic

---

## 💬 Got Questions?

- Check workflow logs in the **Actions** tab
- Read error messages carefully (they're usually helpful!)
- Search [GitHub Community](https://github.community) for similar issues
- Create an issue in this repo for discussion!

---

## 🎉 Let's Automate!

Happy automation! 🚀 Push code, watch it build and test automatically, and celebrate your CI/CD success! 

**Remember:** Every successful workflow run is a step towards better code quality and faster development! 💪

---

<div align="center">

**Made with ❤️ for learning GitHub Actions**

*Go forth and automate!* 🤖✨

</div>
