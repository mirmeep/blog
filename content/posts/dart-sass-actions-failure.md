+++
title = 'HUGO GitHub Actions Deployment Failure with Dart-Sass'
date = 2026-05-29T14:44:13-07:00
tags = ["github"]
draft = false
+++

### ISSUE: Hugo Site GitHub Actions Deployment Failure: `sudo snap install dart-sass`

![error message](/images/blog/dart-sass-actions-failure.png)

```
Run sudo snap install dart-sass
error: cannot perform the following tasks:
- Download snap "snapd" (26865) from channel "stable" (Get "https://api.snapcraft.io/api/v1/snaps/download/PMrrV4ml8uWuEUDBT8dSGnKUYbevVhc4_26865.snap": read tcp 10.1.0.253:40816->185.125.188.59:443: read: connection reset by peer)
```

Installing Dart Sass via GitHub Actions through snap install results in a connection reset by peer error (shown above).

This error occurred after making a change to my GitHub workflows file (in this case, `hugo.yaml`) in the root of my Hugo project. If you must know, the change was a simple command to dynamically inject an API key from my GitHub repository settings into my Hugo project.

But, what does installing dart-sass have anything to do with injecting an API key? Clearly, the change had nothing to do with dart-sass. So, why was this happening? 

It was merely a coincidence. 
### WHY? GitHub Caches Workflows During Deployment 

Note: this section contains speculation and needs further fact checking.

By updating hugo.yaml, `sudo snap install dart-sass` suddenly gave an error because something about the connection between the snap store and GitHub pages had stopped working. This behavior implies that GitHub does not re-run the commands in the workflows if:
1. A change hasn't been made to the workflow.
2. The output of the command already exists.

[More on GitHub caching here](https://cli.github.com/manual/gh_cache_delete "https://cli.github.com/manual/gh_cache_delete")

Since I hadn't made a change to my workflow file in who knows how long- probably since I began this project a few years ago- the command `sudo snap install dart-sass` hasn't been run in quite some time. Rather, it's output, dart-sass, became cached and GitHub Actions used the cached version of dart-sass during deployment. Either the snap store is having issues or this method is outdated. Regardless, we have to fetch it via a different method.
### SOLUTION: GitHub Release Downloader
We will obtain dart sass through the GitHub Release Downloader. This tool allows us to download assets from specified repositories (in this case, sass/dart-sass) directly into our Hugo project.

```yaml
- name: Install Dart Sass
	uses: robinraju/release-downloader@v1
	with:
	  repository: "sass/dart-sass"
	  latest: true
	  filename: "dart-sass-*-linux-x64.tar.gz "
	  tarBall: false
	  zipBall: false
	  extract: true
- name: Add Dart-Sass to Path
	run: echo "$(pwd)/dart-sass" >> $GITHUB_PATH
	shell: bash
```

This uses the GitHub Release Downloader and requests a download for the repository sass/dart-sass. We must add it to GITHUB_PATH so it can be accessible from anywhere in the Hugo project. 

This is just a quick fix I had found on the fly. Perhaps there is a better way, but this seems to be working for now.

Sources
[GitHub Release Downloader](https://github.com/robinraju/release-downloader)
[Dart-Sass Repository](https://github.com/sass/dart-sass)