# powerbi-testing

This repository is a test of a Git workflow for Power BI reports. The report uses the Power BI Project format (PBIP). The data is the Financial Sample that comes with Power BI Desktop.

## Why PBIP instead of .pbix

A `.pbix` file is one binary file. Git cannot show the changes inside a binary file. If two people edit copies of the same `.pbix` file, the last copy that somebody saves replaces the other changes.

A PBIP project keeps the same report as a folder of text files. Git records each change to each file. As a result, you can:

- See which lines changed, and who changed them.
- Go back to an earlier version of the report.
- Review a change in a pull request before the merge into `main`.
- Work on different parts of the report at the same time.

## Folder structure

```
pbip_test/
├── pbip_test.pbip              Open this file in Power BI Desktop
├── .gitignore                  Keeps the local data and settings out of Git
├── pbip_test.Report/           The report, in PBIR format
│   └── definition/
│       └── pages/<page>/visuals/<visual>/visual.json
└── pbip_test.SemanticModel/    The data model, in TMDL format
    └── definition/
        └── tables/financials.tmdl
```

- Each page and each visual has its own JSON file. The pie chart is in one `visual.json` file.
- Each table has its own TMDL file. The file contains the columns, the measures, and the Power Query steps that load the data.

## The data is not in Git

The `.gitignore` file excludes `cache.abf` and `localSettings.json`. The `cache.abf` file is the local copy of the data. Git keeps the queries that load the data, but not the data.

When you open a new clone, Power BI Desktop opens the model without data. You must refresh the data before the visuals show values.

Our real reports load data from shared sources, for example SharePoint and SQL Server. The queries contain the address of each source. As a result, all people refresh from the same source.

Each person signs in to the source with their own credentials. Git does not keep the credentials.

### The sample data in this demo

This demo uses a local file instead of a shared source:

```
C:\Program Files\Microsoft Power BI Desktop\bin\SampleData\Financial Sample.xlsx
```

The standalone installer of Power BI Desktop puts the file at this path. Other installation methods can put the file at a different path. Then the refresh fails.

If the refresh fails, open **Transform data**. Then change the path in the **Source** step. Do not commit the changed path. The changed path is correct only for your computer. Step 7 below shows how to keep the path out of a commit.

## How to make a change

Before you start, install Power BI Desktop and Git.

CAUTION: Save and close Power BI Desktop before you pull, switch branches, or merge. An open project can write its old version over the new files.

1. Clone the repository. Then go into the folder:
   ```
   git clone https://github.com/brickfrog/powerbi-testing.git
   cd powerbi-testing
   ```
2. Make a branch for your change:
   ```
   git switch -c my-change
   ```
3. Open `pbip_test/pbip_test.pbip` in Power BI Desktop.
4. Refresh the data (**Home** > **Refresh**).
5. Make your changes. Then save the project (**Ctrl+S**).
6. Look at the changes:
   ```
   git status
   git diff
   ```
7. Commit the changes:
   ```
   git add -A
   git commit -m "Describe the change"
   ```
   If you changed the **Source** path for your computer, unstage that change before the commit:
   ```
   git add -A
   git reset -p pbip_test/pbip_test.SemanticModel/definition/tables/financials.tmdl
   git commit -m "Describe the change"
   ```
   Git shows each change in the file. Type `y` for the path change. Type `n` for all other changes. If Git shows the path change together with other changes, type `s` to split them first.
8. Push the branch:
   ```
   git push -u origin my-change
   ```
9. Open a pull request on GitHub. Another person reviews the diff before the merge into `main`.

To move an existing `.pbix` file to this format, open the file in Power BI Desktop. Select **File** > **Save as**. Then set **Save as type** to **Power BI project files (\*.pbip)**.

## What a change looks like

Commit [`831fcb0`](https://github.com/brickfrog/powerbi-testing/commit/831fcb06feecda7ffe3e0b60088b4a4376684450) is an example. The commit changes two files:

- `financials.tmdl`: A new Power Query step keeps only the rows for the year 2014.
- `visual.json`: The pie chart has a new title, a new name for the measure, and new legend and label settings.

Formatting changes make long diffs. Power BI writes each formatting setting as many lines of JSON. In this commit, the formatting changes add approximately 90 lines.

## When two people change the report

Each visual, page, and table is in a separate file. If two people change different files, Git merges the changes without a conflict. If two people change the same file, Git can show a merge conflict.

Some files change for many types of edits. For example, `pages.json` changes when somebody adds a page.

If a merge conflict occurs, resolve the conflict in the text file. Then open the project in Power BI Desktop. Make sure that the report loads.

## Limits

- Each person refreshes the data on their own computer. Git does not keep the data or the credentials.
- All people must use a current version of Power BI Desktop. Older versions can fail to open the PBIR format.
- This demo reads a local file. A local file path is correct only on computers that have the file at the same path. Real reports use shared sources.
