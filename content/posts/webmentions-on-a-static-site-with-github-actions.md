---
title: "Webmentions on a static site with GitHub Actions"
slug: webmentions-on-a-static-site-with-github-actions
date: 2020-03-27
categories: ["articles"]
tags:
  - webmentions
  - static sites
  - GitHub Actions
---

Last year, I [added webmentions to this blog](/adding-webmentions-to-my-blog/). To recap, webmentions are a web standard to create a network of comments, likes, and reposts between ordinary sites. I set up a [brid.gy](https://brid.gy) account to poll Twitter for webmentions targetting my blog, and I caught them with [webmention.io](https://webmention.io).

Webmentions were fetched with AJAX and rendered at the bottom of each page. There were two things I didn't like about this approach:

- I'd rather just have them prerendered by Hugo, my static site generator
- Webmentions are stored on Webmention.io, but I'd rather have ownership over them

After some tinkering, I came up with an alternative: a cron-based GitHub Action that queries webmention.io for new webmentions. The Action then commits them to my site's repository, so I can access the data with my static site generator, Hugo.

<aside>
While I applied this to a Hugo blog, the Node script and GitHub workflow work for any static site generator.
</aside>

<!--more-->

Interested? Make sure you read [last year's webmentions](https://sebastiandedeyne.com/adding-webmentions-to-my-blog/) post first, as you'll need to set up Bridgy and Webmention.io as explained there before setting up this Action.

## Fetching webmentions

First we need to fetch webmentions from webmention.io. We'll write a short Node script for that. The script will then be called from our GitHub Action, but can be run locally too for testing purposes.

```bash
node ./webmentions.js
```

We need to set a few parameters when querying webmention.io. First, an API token, which we can grab from our [settings on webmention.io](https://webmention.io/settings).

```js
const token = process.env.WEBMENTIONS_TOKEN;
```

The GitHub Action will retrieve it from the repository's "Secrets", more on that later. Locally we can set the `WEBMENTIONS_TOKEN` on the command line.

```bash
WEBMENTIONS_TOKEN=XXX node ./webmentions.js
```

Next, we determine a start date. We don't always need to fetch all webmentions from the beginning of time.

The Action will run once every six hours. Fetching the webmentions of the past 6 hours *should* be enough but I've noticed some mentions come in with a delay. My script is currently set up to retrieve all webmentions from the past 3 days. It's okay if we fetch the same mentions multiple times, we'll filter out the duplicates later on.

```js
const since = new Date();
since.setDate(since.getDate() - 3);
```

With these parameters we can set up a request URL for the webmention.io API.

```js
const url =
  "https://webmention.io/api/mentions.jf2" +
  "?domain=sebastiandedeyne.com" +
  `&token=${token}` +
  `&since=${since.toISOString()}` +
  "&per-page=999";
```

Now to call the URL and grab the webmentions from the response. We'll use Node's built in `https` module for this. It's quite verbose, but I like a zero-dependencies approach where possible.

```js
const https = require('https');

function fetchWebmentions() {
  const token = process.env.WEBMENTIONS_TOKEN;

  const since = new Date();
  since.setDate(since.getDate() - 3);

  const url =
    "https://webmention.io/api/mentions.jf2" +
    "?domain=sebastiandedeyne.com" +
    `&token=${token}` +
    `&since=${since.toISOString()}` +
    "&per-page=999";

  return new Promise((resolve, reject) => {
    https
      .get(url, res => {
        let body = "";

        res.on("data", chunk => {
          body += chunk;
        });

        res.on("end", () => {
          try {
            resolve(JSON.parse(body));
          } catch (error) {
            reject(error);
          }
        });
      })
      .on("error", error => {
        reject(error);
      });
  }).then(response => {
    if (!("children" in response)) {
      throw new Error("Invalid webmention.io response.");
    }

    return response.children;
  });
}
```

Webmention.io returns an object with a `children` key containing the webmentions.

Now we need to store the webmentions from the response. In a Hugo site, we can store them in `data/webmentions`. We'll create one JSON file per webmention target (a webmention target is a fancy word for the URL that the webmention points towards).

```txt
data/
  webmentions/
    adding-webmentions-to-my-blog.json
    unix-things--listing-directories.json
```

Slashes in the webmention target get trimmed and replaced with `--`, so `https://sebastiandedeyne/unix-things/listing-directories/` maps to `unix-things--listing-directories.json`.

We can generate a slug with some regex voodoo, then use them to build a full path to a JSON file.

```js
fetchWebmentions().then(webmentions => {
  webmentions.forEach(webmention => {
    const slug = webmention["wm-target"]
      .replace("https://sebastiandedeyne.com/", "")
      .replace(/\/$/, "")
      .replace("/", "--");

    const filename = `${__dirname}/data/webmentions/${slug}.json`;

    // ...
  });
});
```

Two things can happen now: either it's the first webmention for the target (so there's no JSON file yet), or there already are webmentions for the target (so there's already a JSON file in place).

The first case is the easiest one to handle. If the file doesn't exist yet, we create a new one with the incoming webmention.

```js
fetchWebmentions().then(webmentions => {
  webmentions.forEach(webmention => {
    const slug = webmention["wm-target"]
      .replace("https://sebastiandedeyne.com/", "")
      .replace(/\/$/, "")
      .replace("/", "--");

    const filename = `${__dirname}/data/webmentions/${slug}.json`;

    if (!fs.existsSync(filename)) {
      fs.writeFileSync(filename, JSON.stringify([webmention], null, 2));

      return;
    }
  });
});
```

If there already is a JSON file, we need to read it out, append the new webmention, and filter out existing webmentions with the same ID so we're not adding duplicates.

```js
const entries = JSON.parse(fs.readFileSync(filename))
  .filter(wm => wm["wm-id"] !== webmention["wm-id"])
  .concat([webmention]);

entries.sort((a, b) => a["wm-id"] - b["wm-id"]);

fs.writeFileSync(filename, JSON.stringify(entries, null, 2));
```

Before writing the new set of webmentions back to the file, we sorted them to ensure the same set of webmentions always creates the exact same file.

{{< details summary="Show full script" >}}
```js
const fs = require("fs");
const https = require("https");

fetchWebmentions().then(webmentions => {
  webmentions.forEach(webmention => {
    const slug = webmention["wm-target"]
      .replace("https://sebastiandedeyne.com/", "")
      .replace(/\/$/, "")
      .replace("/", "--");

    const filename = `${__dirname}/data/webmentions/${slug}.json`;

    if (!fs.existsSync(filename)) {
      fs.writeFileSync(filename, JSON.stringify([webmention], null, 2));

      return;
    }

    const entries = JSON.parse(fs.readFileSync(filename))
      .filter(wm => wm["wm-id"] !== webmention["wm-id"])
      .concat([webmention]);

    entries.sort((a, b) => a["wm-id"] - b["wm-id"]);

    fs.writeFileSync(filename, JSON.stringify(entries, null, 2));
  });
});

function fetchWebmentions() {
  const token = process.env.WEBMENTIONS_TOKEN;

  const since = new Date();
  since.setDate(since.getDate() - 3);

  const url =
    "https://webmention.io/api/mentions.jf2" +
    "?domain=sebastiandedeyne.com" +
    `&token=${token}` +
    `&since=${since.toISOString()}` +
    "&per-page=999";

  return new Promise((resolve, reject) => {
    https
      .get(url, res => {
        let body = "";

        res.on("data", chunk => {
          body += chunk;
        });

        res.on("end", () => {
          try {
            resolve(JSON.parse(body));
          } catch (error) {
            reject(error);
          }
        });
      })
      .on("error", error => {
        reject(error);
      });
  }).then(response => {
    if (!("children" in response)) {
      throw new Error("Invalid webmention.io response.");
    }

    return response.children;
  });
}
```
{{</ details >}}

## Running on GitHub Actions

Now that we have our script in place, we want to run it periodically as a GitHub Action. The Action gets stored in a workflow YAML file in the repository.

```txt
.github/
  workflows/
    webmentions.yml
```

First we need to name it, and determine when it will run. In our case, every 6 hours.

```yml
name: Webmentions

on:
  schedule:
    - cron: "0 */6 * * *"
```

The `cron` syntax is pretty crazy if you're new to it. [Crontab Guru](https://crontab.guru) is a useful tool to help understand what's going on.

Next, we need to set up the environment. We'll spin up an Ubuntu image, clone the repository and set up Node.js.

```yml {hl_lines=["7-17"]}
name: Webmentions

on:
  schedule:
    - cron: "0 */6 * * *"

webmentions:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@master

      - name: Set up Node.js
        uses: actions/setup-node@master
        with:
          node-version: 12.x
```

With out image up and running, we can run the webmentions script in the next step.

```yml
- name: Fetch webmentions
  env:
    WEBMENTIONS_TOKEN: ${{ secrets.WEBMENTIONS_TOKEN }}
  run: node ./webmentions.js
```

The `WEBMENTIONS_TOKEN` is stored as a secret in the GitHub repository.

![Screenshot of GitHub's Secrets interface](/media/webmentions-github-secrets.jpg)

All of this is useless without actually persisting the webmentions we fetched. Since we're running in an Ubuntu environment, and we can commit the new webmentions to the repository.

```yml
- name: Commit to repository
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    COMMIT_MSG: |
      Fetch webmentions
      skip-checks: true
  run: |
    git config user.email "sebastiandedeyne@gmail.com"
    git config user.name "Sebastian De Deyne"
    git remote set-url origin https://x-access-token:${GITHUB_TOKEN}@github.com/sebastiandedeyne/sebastiandedeyne.com.git
    git checkout master
    git add .
    git diff --quiet && git diff --staged --quiet || (git commit -m "${COMMIT_MSG}"; git push origin master)
```

That last line looks pretty crazy. Translated to human, it means "only commit these changes if there *are* changes". There's no need to commit if there are no new webmentions.

Whenever this Action runs and new webmentions are added, the commit triggers a deploy on Netlify, and my live site is up to date!

{{< details summary="Show full script" >}}
```yml
name: Webmentions

on:
  schedule:
    - cron: "0 */6 * * *"

jobs:
  webmentions:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@master

      - name: Set up Node.js
        uses: actions/setup-node@master
        with:
          node-version: 12.x

      - name: Fetch webmentions
        env:
          WEBMENTIONS_TOKEN: ${{ secrets.WEBMENTIONS_TOKEN }}
        run: node ./webmentions.js

      - name: Commit to repository
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          COMMIT_MSG: |
            Fetch webmentions
            skip-checks: true
        run: |
          git config user.email "sebastiandedeyne@gmail.com"
          git config user.name "Sebastian De Deyne"
          git remote set-url origin https://x-access-token:${GITHUB_TOKEN}@github.com/sebastiandedeyne/sebastiandedeyne.com.git
          git checkout master
          git add .
          git diff --quiet && git diff --staged --quiet || (git commit -m "${COMMIT_MSG}"; git push origin master)
```
{{</ details >}}

Now that everything's set up, commits our pouring in with freshly fetched webmentions!

![Screenshot of GitHub commits](/media/webmentions-github-commits.jpg)
