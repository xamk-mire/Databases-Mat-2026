# GitHub Repository — Submission Instructions

This course uses **your own GitHub repository** as the place to store and submit exercise work. You will copy the weekly lecture folders into that repository, complete the exercises there, and push your changes so your instructor can review them.

You do **not** need prior Git experience. Follow the steps below in order. If you get stuck, ask in class or contact your instructor.

---

## What You Are Setting Up

| Item | Purpose |
|------|---------|
| **GitHub account** | Free cloud storage for your project files with version history |
| **Your repository** | Your personal copy of the course exercise structure |
| **Weekly folders** | One folder per teaching week (copied from course materials) |
| **`Exercises.md`** | The file where you write most of your answers |
| **SQL and diagram files** | Optional separate files for scripts, ER diagrams, and the Week 48 final package |

**Required submissions:** Weeks **36–48** (TrailShop project). Weeks **49–51** are optional exam review — nothing to submit.

---

## Step 1 — Create a GitHub Account

1. Go to [https://github.com/signup](https://github.com/signup).
2. Follow the registration steps.
3. Verify your email address if GitHub asks you to.

If you already have an account, you can use it.

---

## Step 2 — Create Your Course Repository

1. Sign in to GitHub.
2. Click the **+** icon (top right) → **New repository**.
3. Fill in the form:
   - **Repository name:** e.g. `databases-trailshop` or `TTZ1234-databases` (use the naming pattern your instructor specifies, if any).
   - **Description:** optional, e.g. `Databases course — TrailShop project`.
   - **Visibility:** choose **Private** unless your instructor tells you to use Public.
   - **Do not** add a README, `.gitignore`, or licence yet — start with an empty repository.
4. Click **Create repository**.

Keep this page open. You will need the repository URL (e.g. `https://github.com/your-username/databases-trailshop`).

---

## Step 3 — Copy Weekly Folders into Your Repository

The course materials are organised under `Lectures/` with one folder per week:

```
Lectures/
  Week36-IntroToDatabases/
  Week37-RelationalConcepts/
  Week38-ConceptualDataModelling/
  ...
  Week48-AdminSecurityNoSQL/
  Week49-FinalReviewDesign/    ← optional review only
  Week50-FinalReviewSQL/       ← optional review only
  Week51-FinalExam/            ← optional review only
```

### Recommended repository layout

Copy the **week folders** into the **root** of your repository so it looks like this:

```
databases-trailshop/                    ← your GitHub repo root
  README.md                           ← you create this (see Step 4)
  Week36-IntroToDatabases/
    Exercises.md                      ← copy from course; add your answers here
  Week37-RelationalConcepts/
    Exercises.md
  Week38-ConceptualDataModelling/
    Exercises.md
  ...
  Week48-AdminSecurityNoSQL/
    Exercises.md
    trailshop_final.sql               ← add in Week 48
    trailshop_queries.sql
    reflection.md
    diagrams/
      er-diagram.png                  ← or .pdf
```

### What to copy from each week folder

| File | Copy? | Notes |
|------|-------|-------|
| **`Exercises.md`** | **Yes** | Required — this is where you submit answers |
| **`Theory.md`** | Optional | Reference only; not required for submission |
| **Your `.sql` files** | As needed | Create in the week folder or a `sql/` subfolder |
| **ER diagrams** | As needed | e.g. `diagrams/` subfolder inside the relevant week |

**Minimum for submission weeks (36–48):** copy each week's folder and at least **`Exercises.md`**. You may omit `Theory.md` to keep the repository smaller.

### Which weeks require submission?

| Weeks | Submit? |
|-------|---------|
| **36–48** | Yes — complete and push `Exercises.md` (and Week 48 final files) |
| **49–51** | No — optional exam preparation only |

---

## Step 4 — Add a README

Create a short `README.md` in the repository root:

```markdown
# Databases — TrailShop Coursework

**Student:** Your Full Name  
**Student ID:** (if required by your instructor)  
**Course:** Databases  
**Period:** Autumn 2026

This repository contains my weekly exercise submissions for the TrailShop project (Weeks 36–48).
```

Replace the placeholders with your own details.

---

## Step 5 — Put the Repository on Your Computer

Choose **one** method below.

### Option A — GitHub Desktop (recommended for beginners)

1. Download [GitHub Desktop](https://desktop.github.com/) and install it.
2. Sign in with your GitHub account.
3. **File → Clone repository** → select your new repository → choose a folder on your computer → **Clone**.
4. Copy the weekly folders from the course `Lectures/` directory into the cloned folder (see Step 3).
5. Add your `README.md`.
6. In GitHub Desktop you will see changed files. Write a short summary (e.g. `Add Week 36 exercises`) → **Commit to main** → **Push origin**.

### Option B — Git in the terminal (Windows PowerShell)

1. Install [Git for Windows](https://git-scm.com/download/win) if you do not have it.
2. Open PowerShell and go to the folder where you want the project:

```powershell
cd $HOME\Documents
git clone https://github.com/YOUR-USERNAME/YOUR-REPO-NAME.git
cd YOUR-REPO-NAME
```

3. Copy the weekly folders from the course materials into this directory.
4. Create `README.md`, then:

```powershell
git add .
git commit -m "Add Week 36–48 exercise folders"
git push
```

Git may ask you to sign in to GitHub the first time you push.

### Option C — Upload via the GitHub website

1. Open your repository on github.com.
2. Click **Add file → Upload files**.
3. Drag in your week folders and `README.md`.
4. Click **Commit changes**.

This works for small updates; for regular weekly work, Option A or B is easier.

---

## Step 6 — Complete and Submit Work Each Week

### Working on exercises

1. Open the week's **`Exercises.md`** in your editor (VS Code, Cursor, etc.).
2. Find the **Your Answer** / **Your SQL** sections.
3. Replace the placeholder text with your own work.
4. Save the file.

### Saving SQL scripts separately (optional but good practice)

For longer SQL, you may create files such as:

```
Week40-SQLFundamentals/
  Exercises.md
  sql/
    create_tables.sql
    insert_sample_data.sql
```

Mention the filename in `Exercises.md` if the answer is in a separate file, e.g. *“See `sql/create_tables.sql`”*.

### Weekly commit and push

After you finish a week's exercises:

1. **Commit** with a clear message, e.g. `Complete Week 40 exercises`.
2. **Push** to GitHub so your instructor can see the latest version.

Do this **every week** for Weeks 36–48. Do not wait until Week 48 to push everything at once — weekly commits show your progress and protect you if you lose local files.

---

## Step 7 — Week 48 Final TrailShop Submission

The final project package is submitted **in Week 48** as part of your repository. See `Lectures/Week48-AdminSecurityNoSQL/Exercises.md` for full requirements.

Place these files in **`Week48-AdminSecurityNoSQL/`** (or paths your instructor specifies):

| File | Description |
|------|-------------|
| `trailshop_final.sql` | Full schema, data, views, indexes, roles, transaction demo |
| `trailshop_queries.sql` | At least 10 demonstration queries |
| `reflection.md` | 300–500 word reflection |
| ER diagram | Image or PDF (e.g. `diagrams/er-diagram.png`) |

Also ensure **`Week48-AdminSecurityNoSQL/Exercises.md`** is complete.

Commit and push with a message such as `Final TrailShop project submission`.

---

## Step 8 — Share the Repository with Your Instructor

Your instructor will tell you how to grant access. Common options:

- **Private repo:** add your instructor as a **collaborator** (Settings → Collaborators → Add people), or
- **Submit the repository URL** via the learning platform (Moodle, etc.).

Make sure the default branch (**main**) contains your latest pushed work before any deadline.

---

## Good Practices

### Do

- Commit and push **regularly** (at least once per submitted week).
- Use **clear commit messages** (`Complete Week 41 exercises`, not `update`).
- Keep **`Exercises.md`** as the main place for written answers.
- Test SQL in PostgreSQL locally before pasting into your files.

### Do not

- Commit **passwords**, connection strings with real passwords, or `.env` files.
- Copy another student's work — GitHub history and similarity checks may be used.
- Rename week folders arbitrarily — keep names like `Week36-IntroToDatabases` so grading is straightforward.

### If you use a `.gitignore`

Create a `.gitignore` in the repository root to avoid accidental commits:

```
.env
*.backup
pgdata/
```

---

## Quick Reference — Weekly Workflow

```
1. Pull latest changes (if working on multiple computers)
2. Open WeekXX-.../Exercises.md
3. Complete your answers
4. Save any .sql or diagram files in that week's folder
5. git add → git commit → git push   (or use GitHub Desktop)
6. Confirm files appear on github.com before the deadline
```

---

## Troubleshooting

| Problem | What to try |
|---------|-------------|
| **Push rejected** | Run `git pull` first, resolve conflicts, then push again |
| **Forgot to push before deadline** | Push immediately; only the version on GitHub counts if that is the rule |
| **Wrong file edited** | You copied from course materials — always edit files **inside your repo clone**, not the original course folder |
| **Large ER diagram won't upload** | Export as PNG or PDF; keep under GitHub's file size limits (100 MB per file) |
| **No Git installed** | Use GitHub Desktop or the website upload option |

---

## Summary

1. Create a **private GitHub repository** for the course.
2. **Copy weekly folders** from `Lectures/` (at least `Exercises.md` for Weeks 36–48).
3. **Complete exercises** in place each week.
4. **Commit and push** so your instructor can review your work.
5. Add **Week 48 final files** (`trailshop_final.sql`, queries, reflection, ER diagram) to the same repository.

For TrailShop project details, see `Initial-Course-project-summary.md`. For the weekly schedule, see `CourseSyllabus.md`.
