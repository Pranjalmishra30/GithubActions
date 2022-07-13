# GithubActions
Write intro about GA

This repository documents my findings from testing out Github-Actions

## TO DO
1. Write intro  
2. Write execute shell script md
3. Write combined waala
4. Write index
 
## Some commonly used Github Actions  

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
          sha: '${{ github.event.pull_request.head.sha }}' 
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

cron statements are used to give schedule for tasks to execute. The time format used is UTC, so convert your local time to UTC.  
Use [Crontab](https://crontab.guru/) to build your custom schedule statements. Checkout the [documentation](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule) 


## REFERENCES  
1.  [Github Actions quickstart](https://docs.github.com/en/actions/quickstart)
2. [Issue with inbuilt cron scheduler](https://upptime.js.org/blog/2021/01/22/github-actions-schedule-not-working/)  
3. [Schedule jobs not running on time (github community)](https://github.community/t/scheduled-jobs-are-not-running-on-time/121271)  
4. 
