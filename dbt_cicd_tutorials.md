# **🚀 CI/CD Setup for dbt Using GitHub Actions**  

This guide provides a **step-by-step tutorial** to set up **Continuous Integration and Continuous Deployment (CI/CD)** for a **dbt project** using **GitHub Actions**. This ensures **automated testing, deployment, and documentation generation** whenever you push changes to GitHub.  

---

## **🎯 Why Set Up CI/CD for dbt?**
✅ **Automate Testing** - Run `dbt test` on every pull request.  
✅ **Validate SQL Queries** - Ensure that `dbt run` executes successfully.  
✅ **Deploy to Production** - Automate deployment of dbt models to production.  
✅ **Generate Documentation** - Automatically update `dbt docs` after deployment.  

---

## **📌 Prerequisites**
1️⃣ A **dbt project** set up locally.  
2️⃣ A **GitHub repository** for your dbt project.  
3️⃣ A **Snowflake, BigQuery, or PostgreSQL database** with correct credentials.  
4️⃣ GitHub Actions enabled for your repo.  

---

## **📌 Step 1: Store Database Credentials as GitHub Secrets**
In your GitHub repository:  
1. **Go to**: `Settings` → `Secrets and variables` → `Actions`  
2. Click **"New repository secret"**, and add the following:  

| Secret Name               | Description (Example for Snowflake) |
|---------------------------|------------------------------------|
| `DBT_PROFILES_YML`        | Contents of `profiles.yml` (Base64 encoded) |
| `DBT_SNOWFLAKE_USER`      | Snowflake Username |
| `DBT_SNOWFLAKE_PASSWORD`  | Snowflake Password |
| `DBT_SNOWFLAKE_ACCOUNT`   | Snowflake Account URL |
| `DBT_SNOWFLAKE_DATABASE`  | Snowflake Database |
| `DBT_SNOWFLAKE_SCHEMA`    | Snowflake Schema |

---

## **📌 Step 2: Create a `profiles.yml` Template for GitHub Actions**
📍 Inside your dbt project, create a `ci/profiles_template.yml` file:
```yaml
default:
  target: dev
  outputs:
    dev:
      type: snowflake
      account: "{{ env_var('DBT_SNOWFLAKE_ACCOUNT') }}"
      user: "{{ env_var('DBT_SNOWFLAKE_USER') }}"
      password: "{{ env_var('DBT_SNOWFLAKE_PASSWORD') }}"
      database: "{{ env_var('DBT_SNOWFLAKE_DATABASE') }}"
      schema: "{{ env_var('DBT_SNOWFLAKE_SCHEMA') }}"
      warehouse: my_warehouse
      role: my_role
      threads: 4
```
✅ This dynamically reads credentials from **GitHub Secrets**.

---

## **📌 Step 3: Create a GitHub Actions Workflow**
📍 Inside your GitHub repository, create a **new workflow file**:  
📂 `.github/workflows/dbt_ci_cd.yml`
```yaml
name: dbt CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  dbt-ci:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set Up Python
        uses: actions/setup-python@v3
        with:
          python-version: '3.9'

      - name: Install dbt
        run: |
          pip install dbt-core dbt-snowflake

      - name: Configure dbt Profile
        run: |
          echo "${{ secrets.DBT_PROFILES_YML }}" | base64 --decode > ~/.dbt/profiles.yml

      - name: Run dbt Debug
        run: dbt debug

      - name: Run dbt Seed
        run: dbt seed

      - name: Run dbt Run
        run: dbt run

      - name: Run dbt Test
        run: dbt test

  dbt-deploy:
    needs: dbt-ci
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v3

      - name: Set Up Python
        uses: actions/setup-python@v3
        with:
          python-version: '3.9'

      - name: Install dbt
        run: |
          pip install dbt-core dbt-snowflake

      - name: Configure dbt Profile
        run: |
          echo "${{ secrets.DBT_PROFILES_YML }}" | base64 --decode > ~/.dbt/profiles.yml

      - name: Deploy dbt Models (Full Refresh)
        run: dbt run --full-refresh

      - name: Generate dbt Documentation
        run: dbt docs generate

      - name: Upload dbt Documentation
        uses: actions/upload-artifact@v3
        with:
          name: dbt-docs
          path: target/

      - name: Deploy Docs to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./target
```

---

## **📌 Step 4: Explanation of GitHub Actions Workflow**
### **🔹 `dbt-ci` Job (Runs on Every PR & Commit to `main`)**
1️⃣ **Checks out code** from the GitHub repository.  
2️⃣ **Sets up Python** to install dbt.  
3️⃣ **Installs dbt & Dependencies** (for Snowflake, use `dbt-snowflake`).  
4️⃣ **Configures dbt Profile** using GitHub Secrets.  
5️⃣ **Runs dbt Debug, Seed, Run, and Test** to validate changes.  

### **🔹 `dbt-deploy` Job (Only on `main` Branch)**
1️⃣ Runs after `dbt-ci` succeeds.  
2️⃣ Deploys dbt models using `dbt run --full-refresh`.  
3️⃣ Generates & uploads dbt documentation (`dbt docs generate`).  
4️⃣ Deploys dbt docs to **GitHub Pages** for easy access.  

---

## **📌 Step 5: Enable GitHub Pages for dbt Docs**
1. Go to your **GitHub repo → Settings → Pages**.  
2. Under **"Source"**, select `"GitHub Actions"`.  
3. Your dbt documentation will be available at:  
   ```
   https://your-github-username.github.io/your-repo-name/
   ```

---

## **📌 Step 6: Trigger the CI/CD Pipeline**
- **Push changes to GitHub**:
  ```sh
  git add .
  git commit -m "Added dbt CI/CD pipeline"
  git push origin main
  ```
- **Go to GitHub → Actions Tab** to monitor the workflow.  
- If everything runs successfully, your **dbt models are tested, deployed, and documented automatically!** 🎉  

---

## **📌 Step 7: Bonus - Using `dbt slim CI` for Faster Builds**
Instead of `dbt run`, use **state-based selection**:
```yaml
- name: Run dbt (Only Modified Models)
  run: dbt run --select state:modified+
```
✅ **Benefit:** Runs only models affected by recent code changes. 🚀  

---

## **🎯 Summary**
✅ **Automated Testing**: `dbt test` runs on every PR.  
✅ **Production Deployment**: Only runs `dbt run --full-refresh` on `main`.  
✅ **Automatic Documentation**: `dbt docs generate` updates GitHub Pages.  
✅ **Secure Credentials**: Uses GitHub Secrets for database access.  

---

