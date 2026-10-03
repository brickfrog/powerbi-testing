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

## From development to production

This repository shows only the Git part of the workflow. A full setup also controls how changes get to the production reports. The practices below come from the Microsoft documentation for Power BI and Fabric.

### One workspace for each stage

Use three workspaces: Development, Test, and Production. Make changes only in Development. A release moves the changes to Test, and then to Production. Do not publish from Power BI Desktop directly to Production.

### Parameters for data sources

Put the SQL Server name, the database name, and the SharePoint site address in Power Query parameters. Each stage sets its own parameter values during the deployment. As a result, Test reads test data and Production reads production data.

If the addresses are not parameters, the person who publishes last decides which source Production reads.

### Release methods

Microsoft documents these [release methods](https://learn.microsoft.com/en-us/fabric/cicd/manage-deployment):

| Method | How it works | Good fit for |
|---|---|---|
| Deployment pipelines | Git connects only to the Development workspace. A deployment moves the content from Development to Test to Production. Deployment rules change the data source parameters for each stage. | Most Power BI teams. This method needs the least engineering work. |
| One Git branch for each stage | Each workspace syncs from its own branch. A pull request moves a change from one branch to the next. | Teams that want Git to be the only record of each release. |
| One main branch and a build script | A merge into `main` starts GitHub Actions or Azure DevOps. The script uses [`fabric-cicd`](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-deploy-fabric-cicd) to deploy the project to each workspace with the values for that stage. | Teams that already use CI/CD for other code. |

A good first step is deployment pipelines, with Git connected to the Development workspace.

### Automatic checks on each pull request

Run checks in GitHub Actions or Azure DevOps for each pull request. Microsoft gives an [example build pipeline](https://learn.microsoft.com/en-us/power-bi/developer/projects/projects-build-pipelines) for this:

- Tabular Editor Best Practice Analyzer examines the semantic model, for example names, unused columns, and DAX patterns.
- PBI Inspector examines the report against rules for visuals.

Protect the `main` branch. Do not allow direct pushes. Require a review from at least one other person.

### Releases

With deployment pipelines, a release is a deployment from Test to Production. Many teams require an approval before this deployment. With the Git methods, a merge or a tag on `main` starts the deployment. In all methods, Production gets only content that passed the review and the Test stage.

### Requirements

- Deployment pipelines need each workspace on a Premium (P) capacity, a Fabric (F) capacity, or Premium Per User (PPU). Only PPU users can open a PPU workspace.
- Git integration for a workspace needs a Premium (P) or Fabric (F) capacity. PPU is not sufficient.
- With Pro licenses only, these Fabric features are not available. A script can publish a `.pbix` file with the Power BI REST API ([Imports](https://learn.microsoft.com/en-us/rest/api/power-bi/imports/post-import-in-group)). But somebody must save the `.pbix` file from the project, and you must build the steps from Development to Production yourself.
- An on-premises SQL Server needs a gateway connection for each stage. SharePoint Online does not need a gateway.
- Fabric Git integration supports GitHub and Azure DevOps.

### Side note: government clouds

The US government clouds (GCC, GCC High, and DoD) are separate from the commercial Power BI service. New features arrive in these clouds later. Before you select a method, find out which cloud and which capacity your organization uses:

- The sign-in address shows the cloud: `app.powerbigov.us` is GCC, `app.high.powerbigov.us` is GCC High, and `app.mil.powerbigov.us` is DoD.
- A diamond icon next to the workspace name shows that the workspace is on a capacity.

Status in October 2026:

- GCC High: Fabric became [generally available on October 1, 2026](https://www.microsoft.com/en-us/microsoft-cloud/blog/us-government/2026/09/02/microsoft-fabric-in-gcc-high-building-the-data-foundation-for-ai/). An existing Premium capacity can also run Fabric.
- GCC: Microsoft [lists only P and EM capacities](https://learn.microsoft.com/en-us/fabric/enterprise/powerbi/service-government-us-overview). F capacities are not available. Ask your administrator if Fabric is available in your tenant.
- DoD: Microsoft has not announced a date for Fabric.
- `fabric-cicd` uses the Fabric REST APIs. Government clouds use different addresses for these APIs, and the APIs can be unavailable. Make sure that they work in your cloud before you select the build script method.

In a government cloud, deployment pipelines on a P capacity or PPU are the safest first choice. With PPU, keep the repository in GitHub or Azure DevOps and publish to the Development workspace from Power BI Desktop, because Git integration needs a capacity.

## Limits

- Each person refreshes the data on their own computer. Git does not keep the data or the credentials.
- All people must use a current version of Power BI Desktop. Older versions can fail to open the PBIR format.
- This demo reads a local file. A local file path is correct only on computers that have the file at the same path. Real reports use shared sources.
