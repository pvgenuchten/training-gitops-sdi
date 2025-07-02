---
title: Git and CI/CD
author: 
- name: Paul van Genuchten 
- name: Tom Kralidis
date: 2025-06-24
---

## Introduction

This page introduces a number of generic IT functionalities. Which can support communities in efficient co-creation of content. Considering the previous material, a relevant scenario is: whenever a user adds or updates a dataset (metadata) in the file repository, an automated pipeline is triggered which validates the changes, converts and uploads the updated records to a catalogue.

## Git content versioning

In its core [Git](https://git-scm.com/) is a code version management system, traditionally used for maintaining software codes. In case you never worked with Git before, have a look at this [Git & GitHub explanation](https://www.w3schools.com/git/git_intro.asp). Some users interact with Git via the command line (shell). However excellent Graphical User Interfaces exist to work with Git repositories, such as [GitHub Desktop](https://desktop.github.com/), a [Git client within Visual Studio](https://learn.microsoft.com/en-us/visualstudio/version-control/git-with-visual-studio?view=vs-2022), [TortoiseGit](https://tortoisegit.org/), [Smartgit](https://www.syntevo.com/smartgit/), and [many others](https://git-scm.com/downloads/guis).

These days Git based coding communities like GitHub, GitLab, Bitbucket offer various services on top of Git to facilitate in co-creation of digital assets. Those services include authentication, issue management, release management, forks, pull requests and CI/CD. The types of digital assets maintained via Git vary from software, deployment scripts, configuration files, documents, website content, metadata records, up to actual datasets. Git is most effective with text based formats, which explains the popularity of formats like CSV, YAML, and Markdown.

## CI/CD

[Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) & [Continuous delivery](https://en.wikipedia.org/wiki/Continuous_deployment) describe a process in which changes in software or configuration are automatically tested and delivered to a relevant environment. These processes are commonly facilitated by Git environments. With every commit to the Git repository an action is triggered which runs some tasks. 

## GitHub Pages exercise

This exercise introduces the CI/CD topic by setting up a basic markdown website in [GitHub Pages](https://pages.github.com/), maintained through Git, similar as to how this workshop website is maintained. [Markdown](https://en.wikipedia.org/wiki/Markdown) is a popular format to store text with annotations on Git. The site will be rendered by the [Quarto](https://quarto.org) library. Quarto is one of many platforms to generate a website from a set of markdown files. Quarto facilitates integrations with R and Python scripts for advanced content creation.

- Create a [new repository](https://docs.github.com/en/get-started/quickstart/create-a-repo) in your GitHub account, for example 'My-first-CMS'. 
- Before we add any content [create a branch](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-and-deleting-branches-within-your-repository) 'gh-pages' on the repository, this branch will later contain the generated html sources of the website, which will be shared via <https://{you}.github.io/{my-first-cms}>. 

- Create a file docs/index.md and docs/about.md. Start each file with a header:

```yaml
---
title: Hello World
author: Peter pan
date: 2023-11-11
---
```

Add some markdown content to each page (under the header), for example:

```markdown
# Welcome

Welcome to *my website*.

- I hope you enjoy it.
- Visit also my [about](./about.md) page.
```

- Now click on `Actions` in the GitHub menu. Notice that GitHub has already set up a workflow to publish our content using [jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll), it should already be available at https://user.github.io/repo. 


## Using Quarto

We've selected an alternative to jekyll, called [quarto](https://quarto.org).
In order to activate Quarto you need to set a number of items yourself.

- Create a file `_quarto.yml` into the new Git repository, with this content:

```yaml
project:
  type: website
  render:
    - "*.md"
website:
  title: "My first CMS"
  navbar:
    left:
      - href: index.md
        text: Home
      - about.md
format:
  html:
    theme: cosmo
    toc: true
```

- First you need to allow the workflow-runner to make changes on the repository. For this, open `Settings`, `Actions`, `General`. Scroll down to `Workflow permissions`. Tick the `Read and write permissions` and click `Save`. If the option is grayed out, you first need to allow this feature in your organization.
- Then, from `Actions`, select `New workflow`, then `set up a workflow yourself`.
- On the next page we will create a new workflow script, which is stored in the repository at /.github/workflows/main.yml.

```yaml
name: Docs Deploy

on:
  push:
    branches: 
      - main

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v3
      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2
        with: 
          tinytex: true 
          path: docs
      - name: Publish to GitHub Pages (and render)
        uses: quarto-dev/quarto-actions/publish@v2
        with:
          target: gh-pages
          path: docs
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
```

- Save the file, via `actions` you can follow the progress of the workflow at every push to the repository.
- On the logs notice how a container is initialised, the source code is checked out, the quarto dependency is installed, the build is made and pushed to the gh-pages branch.

Notice that the syntax to define workflows is different for every CI/CD platform, however they generally follow a similar pattern. For GitHub identify in the file above: 

- It defines at what events the workflow should trigger (in this case at `push` events). 
- a build job is triggered, which indicates a container image (runs-on) to run the job in, then triggers some steps. 
- The final step triggers a facility of quarto to [publish](https://github.com/quarto-dev/quarto-actions/tree/main/publish) its output to a GitHub repository

The above setup is optimal for co-creating a documentation repository for your community. Users can visit the source code via the `edit on GitHub` link and suggest improvements via issues of pull requests. Notice that this tutorial is also maintained as markdown in Git.

---

## Update catalogue from Git CI/CD

For this scenario we need a database in the cloud to host our records (which is reachable by GitHub actions). For the training consider to set up a FREE account at [supabase.com](https://supabase.com/dashboard). 

- At supabase, create a new account. 
- Then create a new Instance of type `Tiny (free)`.
- Click on the instance and notice the relevant connection string (URL) and password 
- Connect your instance of pycsw to this database instance, by updating `pycsw.cfg` and following the instructions at [Catalogue publication](./3-catalogue-publication.md)
- Verify in supabase dashboard if the records are correctly loaded.

We will now publish our records from GitHub to our database.

- Create a new repository on GitHub for the records
- Make sure [git-scm](https://git-scm.com/) (or a GUI tool like [Git kraken](https://www.gitkraken.com/) or [Smartgit](https://www.syntevo.com/smartgit/)) is intalled on your system.
- Clone (download) the repository to a local folder.

```bash
git clone https://github.com/username/records-repo.git
```
- Copy the MCF files, which have been generated in [Catalogue publication](./3-catalogue-publication.md), to a `data` folder in the cloned docker repository.
- Commit & push the files

```bash
git add -A && git commit -m "Your Message"
```

Before you can push your changes to GitHub, you need to set up authentication, generally 2 options are possible:
- Using a [personal access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- Or using [SSH public key](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/about-authentication-to-github#ssh)

```bash
git push origin main
```

We'll now set up CI/CD to publish the records

- Place the pycsw.cfg file in the root of the repository (including the postgres database connection)
- Create a new custom workflow file with this content:

```yaml
name: Records Deploy

on: 
  push:
    paths:
      - '**'

defaults:
  run:
    working-directory: .

jobs:
  build:
    name: Build and Deploy Records
    runs-on: ubuntu-latest
    steps:
        - uses: actions/checkout@v3
        - uses: actions/setup-python@v4
          with:
              python-version: 3.9
        - name: Install dependencies
          run: |
            sudo add-apt-repository ppa:ubuntugis/ppa
            sudo apt-get update
            sudo apt-get install gdal-bin
            sudo apt-get install libgdal-dev
            ogrinfo --version
            pip3 install GDAL==3.4.3
            pip3 install geodatacrawler pycsw sqlalchemy
        - name: Crawl metadata
          run: |
            export pgdc_webdav_url=http://localhost/collections/metadata:main/items
            export pgdc_canonical_url=https://github.com/pvgenuchten/data-training/tree/main/datasets/
            crawl-metadata --dir=./datasets --mode=export --dir-out=/tmp
        - name: Publish records
          run: |   
            pycsw-admin.py delete-records --config=./pycsw.yml -y
            pycsw-admin.py load-records --config=./pycsw.yml  --path=/tmp
```

- Verify that the records are loaded on pycsw (through postgres)
- Change or add some records to Git, and verify if the changes are published (may take some time)

Normally, we would **not** add a connection string to a database in a config file posted on GitHub. Instead GitHub offers [secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) to capture this type of information. 

---

## Cross linking catalogue and Git

While users are browsing the catalogue (or this page), they may find irregularities in the content. They can flag this as an issue in the relevant Git repository. A nice feature is to add a link in the catalogue page which brings them back to the relevant MCF in the Git repository. With proper authorizations they can instantly improve the record, or suggest an improvement via an issue or [pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests).

---

## Summary

In this section you learned about using actions in GitHub (CI/CD). In the [next section](./6-data-publication.md) we are diving into data publication. Notice that you can also use Git CI/CD mechanisms to deploy or evaluate metadata and data services.

