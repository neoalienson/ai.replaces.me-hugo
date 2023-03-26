---
title: "Creating a Blog With Hugo"
date: 2023-03-21T02:01:38+08:00
draft: false
---

{{< toc >}}

## Setup Hugo and GitHub Pages
The first step to have AI replaces me is uses it to generate this blog site. Let's try with ChatGPT 

#### Prompt
Create a blog “ai.replaces.me” with hugo, a static page generator. It uses hugo theme PaperMod.  The blog will host on GitHub Page with domain name ai.replaces.me. Create ci/cd pipelines with github actions. The site will deploy to github-pages environment by the github actions workflow. The github actions uses self-hosted runner.

#### ChatGPT

To create a blog "ai.replaces.me" with Hugo and deploy it to GitHub Pages using GitHub Actions, you can follow these steps:

1. Install Hugo on your local machine.

2. Create a new Hugo site by running the command hugo new site ai.replaces.me in your terminal.

3. Navigate to the new site directory by running cd ai.replaces.me.

4. Add the PaperMod theme as a submodule by running git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod.

5. Copy the example config.toml file from the theme by running cp themes/PaperMod/exampleSite/config.toml ..

6. Edit the config.toml file to your liking, including setting the baseURL to https://ai.replaces.me.

7. Create a new content file for your first blog post by running hugo new posts/my-first-post.md.

8. Write your blog post in Markdown format in the newly created file.

9. Add, commit, and push your changes to a new GitHub repository.

10. In the repository settings on GitHub, enable GitHub Pages and set the source to the gh-pages branch.

11. Create a new personal access token with the repo scope in your GitHub account settings.

12. In the repository settings on GitHub, go to the Secrets tab and add a new secret called ACCESS_TOKEN with the value of your personal access token.

13. Create a new workflow file called .github/workflows/deploy.yml with the following contents:
~~~~
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: self-hosted
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
      - name: Build
        run: hugo --minify
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          deploy_key: ${{ secrets.ACCESS_TOKEN }}
          publish_dir: ./public
~~~~
14. Commit and push the workflow file to your repository.

15. Wait for the workflow to complete, then visit https://ai.replaces.me to see your deployed blog.

That's it! You now have a static blog powered by Hugo and deployed to GitHub Pages using GitHub Actions. You can add more blog posts and customize the theme to your liking.
