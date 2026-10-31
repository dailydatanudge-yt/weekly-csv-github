# Code Along: Python vs Claude Skill

### Same task, two ways: read a CSV from GitHub and count missing values

**Channel:** Daily Data Nudge **Time:** about 20 minutes **Level:** Beginner

---

## What we will build

One task, solved twice:

| Part | Tool | Who does the work |
| --- | --- | --- |
| 1 | Python script | Code reads the file and counts |
| 2 | Claude Skill (no script) | Claude reads the file and counts |

Then we compare the results and learn **when to use which**.

**The task:** a sales CSV sits in GitHub. Read it, show the first rows, and count the missing values per column.

---

## Before you start

You need:

- Python 3.9 or newer
- VS Code
- A terminal (the VS Code terminal is fine)
- For Part 2 only: Claude Code (a paid Claude subscription or Anthropic Console account)

**Our data** lives in a public GitHub repo: `https://github.com/dailydatanudge-yt/weekly-csv-github`

```
weekly-csv-github/
└── Data/
    ├── sales_week1.csv
    ├── sales_week2.csv
    ├── sales_week3.csv
    └── sales_week4.csv
```

Each file has 11 columns: `order_id`, `order_date`, `customer_id`, `customer_name`, `product_category`, `product_name`, `quantity`, `unit_price`, `payment_method`, `city`, `order_status`.

The files contain a few **empty cells on purpose**. Finding them is our job.

---

# PART 1: The Python way

## Step 1: Create the project folder

```powershell
mkdir weekly-csv-github
cd weekly-csv-github
mkdir Scripts
```

## Step 2: Create a virtual environment and install pandas

A virtual environment keeps this project's packages separate from everything else on your machine.

```powershell
python -m venv .venv
.venv\Scripts\activate
pip install pandas
```

When it works, your terminal prompt starts with `(.venv)`.

## Step 3: Get the raw file link

1. Open `sales_week1.csv` on GitHub.
2. Click the **Raw** button.
3. Copy the address from the browser bar.

It should look like this:

```
https://raw.githubusercontent.com/dailydatanudge-yt/weekly-csv-github/main/Data/sales_week1.csv
```

> Why raw? The normal GitHub page shows a web page around the file. The raw link gives pandas only the data.

## Step 4: Write the script

Create `Scripts/read_csv_github.py`:

```python
import time
import pandas as pd

# Raw GitHub link to ONE file (public repo, so no token is needed)
URL = "https://raw.githubusercontent.com/dailydatanudge-yt/weekly-csv-github/main/Data/sales_week1.csv"

start = time.time()

# Step 1: pandas reads a web link directly, just like a local file
df = pd.read_csv(URL)

# Step 2: first look at the data
print("Rows, Columns:", df.shape)
print(df.head())

# Step 3: count missing values per column
missing = df.isna().sum()
print("\nMissing values per column:")
print(missing[missing > 0])   # show only columns that have a problem

print(f"\nDone in {time.time() - start:.2f} seconds")
```

**Save the file (Ctrl+S) before running.** Python runs the saved version, not what you see on screen.

## Step 5: Run it

```powershell
python Scripts\read_csv_github.py
```

## What each part does

| Line | Meaning |
| --- | --- |
| `pd.read_csv(URL)` | Downloads the file and loads it into a table called a DataFrame (`df`) |
| `df.shape` | Number of rows and columns |
| `df.head()` | The first 5 rows. A quick peek at the top of the data |
| `df.isna()` | A True/False table: True wherever a cell is empty |
| `.sum()` | Adds up the True values for each column, which gives the count of missing cells |
| `missing[missing > 0]` | Keeps only the columns that have a problem |

## Expected output (week 1)

```
Rows, Columns: (25, 11)

Missing values per column:
customer_name    1
unit_price       1
city             1
```

**Total: 3 missing values.** In `df.head()` you will see one `NaN`. That is how pandas shows an empty cell.

## Run it 3 times

Run the command again, twice. You get **the same numbers every time**. Remember this for the comparison.

---

# PART 2: The Claude Skill way

## What is a skill?

A skill is a **plain text instruction file** (`SKILL.md`) that tells Claude how to do a task. There is no code in it. Claude reads the instructions and does the work using its own tools.

## Step 6: Install Claude Code

Use the official quickstart for your system: [https://code.claude.com/docs/en/quickstart](https://code.claude.com/docs/en/quickstart)

In VS Code you can install the Claude Code extension, then press `Ctrl+Shift+P`, type "Claude Code", and choose **Open in New Tab**.

## Step 7: Create the skill folder

From the project root:

```powershell
mkdir .claude\skills\csv-missing-check
```

## Step 8: Create the skill file

Create `.claude/skills/csv-missing-check/SKILL.md` with this content:

```markdown
---
name: csv-missing-check
description: Read a CSV file from the Daily Data Nudge GitHub repo and report the row count, column count, and missing values per column. Use this whenever the user asks to check, read, profile, or count missing/empty/null values in a sales CSV from GitHub (for example sales_week1.csv), even if they do not say "skill" or "missing values" explicitly.
---

# CSV Missing Value Check (no scripts)

Read one CSV file straight from GitHub and report what is missing.
Do this directly. Do not create or save any .py file.

## Source

Base URL (public repo, raw files):
https://raw.githubusercontent.com/dailydatanudge-yt/weekly-csv-github/main/Data/

File URL = base URL + file name.
If the user names no file, use sales_week1.csv.

## Steps

1. Fetch the file at the URL above using the web fetch tool.
   If the fetch fails, say so plainly and stop. Do not guess the data.
2. Count the data rows and the columns. The header line is not a data row.
3. For each column, count the cells that are completely empty.
   - A cell is missing only if it is empty (nothing between the commas).
   - Do not count cells that merely contain extra spaces or odd capitalisation.
   - Check every row. Do not sample.
4. Report using the exact format below.

## Output format

**File:** <file name>
**Rows, Columns:** (<rows>, <columns>)

| Column | Missing |
|---|---|
| <only columns with at least 1 missing> | <count> |

**Total missing cells:** <number>

**Method used:** state in one line how the counting was done
(for example "read the text and counted by hand" or "ran code to count").

## Rules

- Never invent, fill in, or guess a missing value.
- If a count is uncertain, say so instead of giving a confident number.
- Do not clean or change the data. This skill only reports.
```

### How the skill file is built

| Part | Purpose |
| --- | --- |
| `name` | What the skill is called. You can also run it by typing `/csv-missing-check` |
| `description` | **How Claude decides to use the skill.** Write it clearly. |
| Steps | The instructions, in plain English |
| Output format | Makes the answer the same shape every time |
| Rules | Guard rails, such as "never guess a missing value" |

## Step 9: Run it

Start Claude Code in the project folder:

```powershell
claude
```

Then type:

```
Check missing values in sales_week1.csv from GitHub
```

If Claude asks permission to fetch the URL, approve it.

## Expected result

The same answer as Part 1:

| Column | Missing |
| --- | --- |
| customer_name | 1 |
| city | 1 |
| unit_price | 1 |

**Total: 3**, with `(25, 11)` for rows and columns. Look at the **Method used** line to see how Claude got there.

## Run it a few times

Run the same prompt 3 to 5 times. Compare each answer with the Python result. Note every difference you see.

---

# Try the other weeks

Change the file name (in Python: the `URL`; in the skill: the prompt) and check your answers against this table.

| File | Rows | Missing cells | Missing columns |
| --- | --- | --- | --- |
| `sales_week1.csv` | 25 | 3 | customer_name, city, unit_price |
| `sales_week2.csv` | 26 | 3 | payment_method, quantity, customer_name |
| `sales_week3.csv` | 26 | 4 | city, unit_price, payment_method, customer_name |
| `sales_week4.csv` | 25 | 5 | order_date, quantity, city, customer_name, order_status |

> Week 2 also has one duplicate row, and week 3 has an order that repeats one from week 2. Cleaning duplicates is a later episode.

---

# The comparison

## Side by side

|  | Python script | Claude Skill |
| --- | --- | --- |
| **Setup** | Write code, install pandas | Write plain English |
| **Same answer every time** | Always | Usually, not guaranteed |
| **Speed** | Under a second | Slower |
| **Cost** | Free | Uses AI usage each run |
| **Checks every row** | Always | Should, but can slip on big files |
| **Can you audit it?** | Yes, point to the one line | The method can change between runs |
| **Handles surprises** (new column, odd layout) | Breaks unless you coded for it | Adapts and explains in plain language |
| **Big files (millions of rows)** | Works | Becomes unreliable and expensive |
| **Who can build it** | Someone who can code | Anyone who can write instructions |

## The rule to remember

> **If the answer must be exact and repeatable, use code.** **If the task needs judgment, explanation, or flexibility, use AI.**

## When to use which

| Situation | Use |
| --- | --- |
| Counting, filtering, summing, joining | **Python** |
| A job that runs every week on a schedule | **Python** |
| Large files or large volumes | **Python** |
| You need to prove how a number was calculated | **Python** |
| Explaining results in plain language | **Skill / Claude** |
| Writing a message to business users | **Skill / Claude** |
| Deciding what to do with unexpected data | **Skill / Claude** |
| A quick one-off check on a small file | Either |

## The best answer is both

A skill does not have to count by itself. In the full pipeline:

- **Python** does the exact work: detect new files, clean, append, count missing values.
- **Claude** directs the work and communicates: it runs the scripts in order and writes a clear message asking business users to fill in the gaps.

Exact work goes to code. Judgment and communication go to AI.

---

# Recap

1. We read a CSV from GitHub with 3 lines of Python.
2. We did the same task with a skill, written in plain English.
3. Code gives a guaranteed, repeatable answer. A skill gives a convenient, flexible one.
4. Pick the tool that matches the task. Use both together for real pipelines.

**Next episode:** clean the data (trim spaces, fix capital letters, remove duplicates).

---

*Daily Data Nudge: short, practical data videos.*