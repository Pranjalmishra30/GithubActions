# GithubActions
Github actions allow us to automate, build and deploy customized workflows.  
This repository documents my findings from testing and learning about Github-Actions

## Index  
1. [Commonly used Github Actions](#commonly-used-github-actions)  
2. [Customized workflow](#customized-workflow)  
3. [References](#references)  

## Commonly used Github Actions  

### [Create-branch](https://github.com/marketplace/actions/create-branch)  
```yml
name: Create branch  
on: [push]  
jobs:  
  create-branch:
    runs-on: ubuntu-latest  
    steps:  
      - uses: peterjgrainger/action-create-branch@v2.2.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          branch: Created-Actions
          sha: ${{ github.event.pull_request.head.sha }} 
```

### [Delete-branch](https://github.com/marketplace/actions/delete-multiple-branches)  
```yml
name: Delete branch  
on: [push]  
jobs:  
  delete-branch:
    runs-on: ubuntu-latest  
    steps:  
      - uses: dawidd6/action-delete-branch@v3
        with:
          github_token: ${{github.token}}
          branches: Sept-2022
```  

### [Merge-branch](https://github.com/marketplace/actions/branch-merge)  
```yml
name: Merge branch  
on: [push]  
jobs:  
  merge-branch:
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v3
      - uses: everlytic/branch-merge@1.1.2
        with:
          github_token: ${{ github.token }}
          source_ref: ${{ github.ref }}
          target_branch: 'main'
          commit_message_template: '[Automated] Merged {source_ref} into target {target_branch}'
```  

### Execute a shell script  
*Replace with your script name*
```yml
name: Execute script  
on: [push]  
jobs:  
  exec-script:
    runs-on: ubuntu-latest  
    steps:  
      - uses: actions/checkout@v2
      - run: |
        chmod +x script.sh
        ./script.sh
      shell: bash 
```

### Schedule-task  
```yml  
name: Schedule Event every 5 mins
on:
  schedule:
    - cron: '*/5 * * * *'
jobs:
  test-schedule:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Job has run at scheduled time"
```  

cron statements are used to give schedule for tasks to execute. The time format used is UTC, so convert your local time to UTC. Use [Crontab](https://crontab.guru/) to build your custom schedule statements. Checkout the [documentation](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule) 





## Customized Workflow  
Using individual scripts to build an action for a customized workflow.  

**The Problem**:  
Imagine a repository which has 2 branches, main and a month branch. The main branch has all tested codes whereas the month branch is where all trials and experiments happen.  

At the end of each month, all changes in the month branch get merged into main. The old branch is deleted and a new branch for next month is created. The readme is also updated with date of merging. Branch name convention: "Month-Year" (July-2022). An automated email should be sent to all contributors, notifying them of the change.  

This workflow needs to be automated with the help of github actions.  

**Solution**:  

This workflow is scheduled to run at 1:00 AM IST on the 1st day of every month.  

Workflow file:
```yml
name: Branch Create and Delete workflow
on:
  schedule:
    - cron: '30 7 1 * *'
jobs:
  mrg-dlt-crt:
    runs-on: ubuntu-latest
    steps:

      # Prev and current month names
      - run: echo "Curr=$(date "+%B-%Y")" >> $GITHUB_ENV
      - run: echo "Prev=$(date -d "$(date +%Y-%m-1) -1 month" +%B-%Y)" >> $GITHUB_ENV
      
      # name: Merge prev month branch
      - uses: everlytic/branch-merge@1.1.2
        with:
          github_token: ${{ github.token }}
          source_ref: ${{ env.Prev }}
          target_branch: 'main'
          commit_message_template: '[Automated] Merged {source_ref} into main branch'
      
      # name: Delete old branch
      - uses: dawidd6/action-delete-branch@v3
        with:
          github_token: ${{github.token}}
          branches: ${{ env.Prev }}
      
      # name: Update README
      - uses: actions/checkout@v2
      - run: |
          chmod +x update.sh
          ./update.sh
        shell: bash 

      # name: commit and push
      - run: |
          git config --global user.email "pranjalmishra2022@gmail.com"
          git config --global user.name "Pranjal"
          git pull
          git add -A
          git commit -m "edited readme"
          git push 
      
      - uses: actions/checkout@v2

      # Create branch and Email TO BE FIXED
      # - uses: peterjgrainger/action-create-branch@v2.2.0
      #   env:
      #     GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      #   with:
      #     branch: ${{ env.Curr }}
      #     sha: '${{ github.event.pull_request.head.sha }}'

```

Update README script  
```sh
#!/bin/bash
txt=$(date "+%d %B %Y")
sed -i "s/main:.*/main: $txt/" README.md 
```

## REFERENCES  
1.  [Github Actions quickstart](https://docs.github.com/en/actions/quickstart)
2. [Issue with inbuilt cron scheduler](https://upptime.js.org/blog/2021/01/22/github-actions-schedule-not-working/)  
3. [Schedule jobs not running on time (github community)](https://github.community/t/scheduled-jobs-are-not-running-on-time/121271)  
