---
layout: post
title: "Using Github actions to Update a Github repository"
date: 2022-06-02 22:00:00 -0600
categories: DevOps
tags: github devops simple tutorial
image:
  path: /assets/img/header/header--github-actions.png
---
# Using Github actions to Update a Github repository


* * *
## DEMO: Update readme.md with the date command

[GitHub Repository Scheduled Update Using GitHub Actions](https://leimao.github.io/blog/GitHub-Repo-Scheduled-Update-GitHub-Actions/)

Practice updating your github repository with the use of github actions.


* * *
### First, create a new repository

Use the "+" icon to create a new directory, name it anything you like.

With the new directory created, you must head to settings. This will let us change Workflow Permissions.


* * *
### Next, update your repository's settings

Under settings, find 'Actions' on the left. 

Under the 'General' section, head down to 'Workflow permissions'. 

To be able to allow our Github actions runner to modify our repository, we must allow 'Read and write permissions'.


* * *
## Adding sample.yaml to our workflow

To create the file `.github/workflows/update-todays-date.yml`

> You cannot create an empty folder *and then* add files to that folder, but rather creation of a folder must happen *together with* adding of at least a single file. This is because git doesn't track empty folders.
{: .prompt-tip }

- Use `/` in the file name field to create folder(s), e.g. typing `folder1/file1` in the file name field will create a folder `folder1` and a file `file1`.


### Create the workflow example file

Create [the following](https://github.com/leimao/What-Is-The-Date-Today/blob/main/.github/workflows/update-date.yml) yaml file and placed it in the .github/workflows directory:
```
name: Update README.md with date

on:
  # https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#schedule
  schedule:
    # The shortest interval you can run scheduled workflows is once every 5 minutes.
    # Note: The schedule event can be delayed during periods of high loads of GitHub Actions workflow runs. 
    # High load times include the start of every hour. 
    # To decrease the chance of delay, schedule your workflow to run at a different time of the hour.
    # Prime numbers are a good example for time of the hour: 3, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53
    # Example: every 5 minutes.
    # - cron: '*/5 * * * *'
    # Currently: every three days at 10:23am UTC.
    - cron: "23 10 */3 * *"

#on: [push]


jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository code
        # https://github.com/actions/checkout/
        uses: actions/checkout@v3
      - name: Clear README with intro text
        run: |
          cat README.md
          echo "# What is Today's Date?" > README.md
          echo "This repository keeps track of today's date." >> README.md
          echo "* * *" >> README.md
          echo " " >> README.md
          echo "## The current date:  " >> README.md
      - name: Modify date and time
        # NOTE: UTC is adjusted to local MNT time. Please correct based on local standards. 
        run: |
          cat README.md
          echo " $(date -d '-6 hour' +"%m/%d/%Y") " >> README.md
          echo "  " >> README.md
          echo "  " >> README.md
          echo " The time is also: " >> README.md
          echo "  " >> README.md
          echo " $(date -d '-6 hour' +"%I:%M.%S") " >> README.md
          echo "  " >> README.md
          echo "  " >> README.md
          cat README.md
      - name: Push to repository
        run: |
          git config --global user.name "YourUserAccountName"
          git config --global user.email "649454182+UserName@users.noreply.github.com"
          now=$(date)
          git add -A
          git commit -m "Auto Push on $now"
          git push
```

You will need to update your `user.name` and `user.email`. 

Details on how to find your email address can be [found here](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-email-preferences/setting-your-commit-email-address).

* * *

> *NOTE:* To test this, uncomment the : `# on: [push]`
{: .prompt-info }


Once the text above has been entered, at the bottom of the page, click `Commit new file`

This will trigger the script above. 


* * *
## View your workflow results

Workflow for this repo can be found under the main page of that repository, and then clicking on `Actions` at the top.

From here, you should be able to see the file above that you entered, and see "Update README.md with date"

Click on that text above, and you'll be presented with a new page. The left hand sidebar will now read "Jobs".

Click on the text for the job, and you will see how each of the steps was processed. Expand any of the steps to view its details.


* * *
## Errors, Issues, Caveats 

If you forget to change your workflow settings, it will fail with the following error:

`remote: Permission to YourUserAccountName/What-Is-Todays-Date.git denied to github-actions[bot].`

* * *
