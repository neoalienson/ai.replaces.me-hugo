---
title: "Creating a Blog With Hugo"
date: 2023-03-21T02:01:38+08:00
draft: false
---

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

## Other prompts
### Delete old releases and tags
how to delete old all github release and tag using github script. keep the latest release and tag

To delete all old GitHub releases and tags using a GitHub Actions workflow and keep the latest release and tag, you can use the actions/github-script action to interact with the GitHub API and delete the releases and tags. Here's an example:

name: Delete Old Releases and Tags

on:
  schedule:
    - cron: '0 0 * * *' # Run every day at midnight

jobs:
  delete-old-releases-and-tags:
    runs-on: ubuntu-latest
    steps:
      - name: Get Releases
        uses: actions/github-script@v4
        id: get-releases
        with:
          script: |
            const { data } = await github.repos.listReleases({
              owner: context.repo.owner,
              repo: context.repo.repo,
              per_page: 100
            });
            core.setOutput('releases', JSON.stringify(data));

      - name: Delete Releases
        uses: actions/github-script@v4
        with:
          script: |
            const releases = JSON.parse('${{ steps.get-releases.outputs.releases }}');
            let latestRelease = null;
            for (const release of releases) {
              if (latestRelease === null || new Date(release.created_at) > new Date(latestRelease.created_at)) {
                latestRelease = release;
              }
            }
            for (const release of releases) {
              if (release.id !== latestRelease.id) {
                console.log(`Deleting release ${release.tag_name}...`);
                await github.repos.deleteRelease({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  release_id: release.id
                });
                console.log(`Deleted release ${release.tag_name}.`);
              }
            }

      - name: Get Tags
        uses: actions/github-script@v4
        id: get-tags
        with:
          script: |
            const { data } = await github.git.listTags({
              owner: context.repo.owner,
              repo: context.repo.repo,
              per_page: 100
            });
            core.setOutput('tags', JSON.stringify(data));

      - name: Delete Tags
        uses: actions/github-script@v4
        with:
          script: |
            const tags = JSON.parse('${{ steps.get-tags.outputs.tags }}');
            let latestTag = null;
            for (const tag of tags) {
              if (latestTag === null || new Date(tag.commit.committer.date) > new Date(latestTag.commit.committer.date)) {
                latestTag = tag;
              }
            }
            for (const tag of tags) {
              if (tag.sha !== latestTag.sha) {
                console.log(`Deleting tag ${tag.name}...`);
                await github.git.deleteRef({
                  owner: context.repo.owner,
                  repo: context.repo.repo,
                  ref: `tags/${tag.name}`
                });
                console.log(`Deleted tag ${tag.name}.`);
              }
            }
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
In this example, the workflow is triggered daily at midnight using a schedule event. The delete-old-releases-and-tags job has four steps:

Get Releases gets a list of releases using the actions/github-script action and the listReleases method provided by Octokit. We are requesting a maximum of 100 releases per page. The releases are stored in the releases output as a JSON string.

Delete Releases loops through the list of releases, finds the latest release, and deletes all old releases using the deleteRelease method provided by Octokit. We use the new Date() function to compare the created_at property of each release and find the latest one. If the current release is not the latest, we delete the release using the repos.deleteRelease method. We log the deleted release to the console.

Get Tags gets a list of tags using the actions/github-script action and the listTags method provided by Octokit. We are requesting a maximum of 100 tags per page. The tags are stored in the tags output as a JSON string.

Delete Tags loops through the list of tags, finds the latest tag, and deletes all old tags using the deleteRef method provided by Octokit. We use the new Date() function to compare the commit.committer.date property of each tag and find the latest one. If the current tag is not the latest, we delete the tag using the git.deleteRef method. We log the deleted tag to the console.

You can modify the JavaScript code block to customize the delete process, such as changing the criteria for deleting a release or tag or keeping more than one latest release or tag. You can also modify the schedule event to change the frequency of the delete operation. Note that deleting releases and tags can be a destructive operation and cannot be undone, so use with caution.
