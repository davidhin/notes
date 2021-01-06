# Basic GitHub Guideline

#### Project Board Setup

1. Create a new project board (in a GitHub repository, go to the "Projects" tab)
2. The project board name can be anything (e.g. main, the name of the tool, etc)
3. Under Project template, choose "*Automated kanban*"
4. The project board displays issues in a three-column layout (to do, in progress, done). GitHub gives you additional info on how to use it. 

#### Main Workflow

> Note: This is just a guideline on GitHub flow, but it can be modified as necessary (see notes). 

1. Create a new issue + description. Usually, it's good to follow a consistent issue description template (see an example below). Assign all new issues to the project board you created earlier - they will automatically be added to the "To do" column, and moved to "Done" when the issue is closed. 
2. Create a branch called JOB-# where # is the issue number (this can be done locally or through GitHub) For example, if Issue #1 is "APIs for getting attack patterns for a given consequence", then the code relating to this issue should be submitted to a branch named JOB-1.
3. Once you're happy with your code, push the JOB-1 branch to GitHub. Create a pull request from JOB-1 to Master. If there's nothing to say in the PR description, just link it to the issue (e.g. closes #1). Request a code reviewer (if doing code review).
4. (If doing code review): If the reviewer has comments/changes to make, commit your changes to the branch with the name: "QA-# blah blah blah", where # is the Pull Request Number and blah blah blah is a short description of what you fixed. 
5. (If doing code review): Once the reviewer accepts the pull request, it can be merged into the Master branch. The JOB-1 branch can now be deleted and Issue #1 can be closed. 

#### Example Issue Description Template

Below is an example of an issue following a defined structure. Note that it is more oriented towards agile software development (user stories, acceptance criteria), so it may not apply to all projects. 

> **User Story** 
> As a user, I want to be able to see if my requests are loading so that I know to wait to not resend requests that are being processed.
>
> **Acceptance Criteria** 
>
> - [x] Add loading page after uploading files. 
> - [x] Add loading page after file explorer page. 
> - [ ] Add loading page after checking files on project page.

#### Additional Notes:

- Do not commit directly to master
- Branch names should follow the JOB-# naming scheme (all caps, dash, number).
- Try to keep issues and the code in their corresponding PR self-contained, coherent, and not too large. 
- You should use an automatic formatter, linter, etc for your code  to keep things consistent. 
- Issue/PR tags are optional (e.g. if you want to add priority tags in Step 1, you can, otherwise you don't have to)
- This workflow omits a "dev" branch, as it is not entirely necessary for small projects. However, use of a dev branch would mean that during Step 5, the PR is merged into a permanent "dev" branch instead of master. During set milestone periods (e.g. 2 weeks), the dev branch can be merged into master, providing all tests pass.  