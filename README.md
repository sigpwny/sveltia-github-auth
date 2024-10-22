# Sveltia Github CMS Authenticator

SIGPwny deployment:

```bash
wrangler deploy --name cms-github-auth
```

Then, we have a CNAME alias from `cms-auth.sigpwny.com` to our worker, `cms-github-auth.sigpwny.workers.dev`,

This simple [Cloudflare Workers](https://workers.cloudflare.com/) script allows [Sveltia CMS](https://github.com/sveltia/sveltia-cms) (or Netlify/Decap CMS) users to authenticate through a [GitHub App](https://docs.github.com/en/apps/creating-github-apps/about-creating-github-apps/about-creating-github-apps). This is in constrast to authenticating through a oauth app, and allows much more fine-grained repository access.

You don’t have to use it if you previously had Netlify/Decap CMS and your site is still being deployed to Netlify or if you have already used [another 3rd party OAuth client](https://decapcms.org/docs/external-oauth-clients/).

You can use it if your site is hosted (or has been moved to) somewhere else, such as [Cloudflare Pages](https://pages.cloudflare.com/) or [GitHub Pages](https://pages.github.com/), and you don’t have any other 3rd party client yet.

## How to use it

### Step 1. Deploy this project to Cloudflare Workers

Sign up with Cloudflare, and click the button below to start deploying.

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/reteps/sveltia-github-auth)

Alternatively, you can clone the project and run [`pnpm run deploy`](https://developers.cloudflare.com/workers/wrangler/commands/#deploy) locally.

If you do it this way, update your `wrangler.toml` to include environment variables:

```toml
[vars]
GITHUB_CLIENT_ID = "Iv23liv25McWF48WFQOB"
GITHUB_REPO_ID = "876466748"
ALLOWED_DOMAINS = "astro-starter-pete.pages.dev"
```

and then add your secret with `wrangler secret put GITHUB_CLIENT_SECRET`. Finally, deploy through wrangler.

Once deployed, open your Cloudflare Workers dashboard, select the `sveltia-github-auth` service, then the worker URL (`https://sveltia-github-auth.<SUBDOMAIN>.workers.dev`) will be displayed. Copy it for Step 2. It will also be used in Step 4.

### Step 2. Register the Worker as an OAuth app

#### GitHub

[Register a new GitHub application](https://github.com/settings/apps/new) on GitHub ([details](https://docs.github.com/en/apps/creating-github-apps/registering-a-github-app/registering-a-github-app)) with the following properties, including your Worker URL from Step 1:

- Application name: `Sveltia Github CMS Authenticator` (or whatever)
- Homepage URL: `https://github.com/reteps/sveltia-github-auth` (or whatever)
- Application description: (can be left empty)
- Authorization callback URL: `<YOUR_WORKER_URL>/callback`
- Ensure 'Setup URL' is blank, and 'Request user authorization (OAuth) during installation' is unchecked. Ensure 'expire access token' is unchecked.

Once registered, click on the **Generate a new client secret** button. The app’s **Client ID** and **Client Secret** will be displayed. We’ll use them in Step 3 below.

#### Install GitHub application

Before you can use the authenticator to authorize users, you must install the app on the target repo.

You can navigate to `https://github.com/apps/<app slug>/installations/new` to install it on the repo.

#### Scope the GitHub access tokens

Optionally, you can scope the user access tokens further. See [This page](https://docs.github.com/en/apps/creating-github-apps/authenticating-with-a-github-app/generating-a-user-access-token-for-a-github-app#using-the-web-application-flow-to-generate-a-user-access-token)

Get the repo id with:

```bash
curl -s 'https://api.github.com/repos/<owner>/<repo>' | jq .id
# 876466748
```

You can then use this with the `GITHUB_REPO_ID` environment variable.

### Step 3. Configure the Worker

Go back to the `sveltia-github-auth` service page on the Cloudflare dashboard, select **Settings** > **Variables**, and add the following Environment Variables to your worker ([details](https://developers.cloudflare.com/workers/platform/environment-variables/#environment-variables-via-the-dashboard)):

#### Environment Variables

- `GITHUB_CLIENT_ID`: **Client ID** from Step 2
- `GITHUB_CLIENT_SECRET`: **Client Secret** from Step 2; click the **Encrypt** button to hide it
- `GITHUB_HOSTNAME`: Required only if you’re using GitHub Enterprise Server. Default: `github.com`
- `GITHUB_REPO_ID` (Optional) The ID of the GitHub repo
- `ALLOWED_DOMAINS`: (Optional) Your site’s hostname, e.g. `www.example.com`
  - Multiple hostnames can be defined as a comma-separated list, e.g. `www.example.com, www.example.org`
  - A wildcard (`*`) can be used to match any subdomain, e.g. `*.example.com` that will match `www.example.com`, `blog.example.com`, `docs.api.example.com`, etc. (but not `example.com`)
  - To match a `www`-less naked domain and all the subdomains, use `example.com, *.example.com`

Save and deploy.

### Step 4. Update your CMS configuration

Open `admin/config.yml` locally or remotely, and add your Worker URL from Step 1 as the new `base_url` property under `backend`:

```diff
 backend:
   name: github
   repo: username/repo
   branch: main
+  base_url: <YOUR_WORKER_URL>
```

Commit the change. Once deployed, you can sign into Sveltia CMS remotely with GitHub!

## FAQ

### Why do I have to set this thing up in the first place?

Technically, we could host Sveltia CMS Authenticator on our own server and let anyone use it, just like Netlify does. The cost probably wouldn’t matter because it’s just a small, short-lived script. However, running such a **service** certainly comes with legal, privacy and security liabilities that we cannot afford. Remember that Sveltia CMS is nothing more than [@reteps](https://github.com/reteps)'s fork of [@kyoshino](https://github.com/kyoshino)’s personal project. That’s why the authenticator is not offered as SaaS and you have to install it yourself.

## Acknowledgements

This project was inspired by [`sveltia-cms-auth`](https://github.com/sveltia/sveltia-cms-auth)
