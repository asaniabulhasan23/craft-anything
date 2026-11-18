# Deploying this fork on Vercel

Vercel's build image has no Rust or Trunk toolchain, so **the Rust app cannot be
built on Vercel**. Importing the repository as a normal project used to deploy
`apps/craft-anything-web/index.html` — the unprocessed Trunk template with
`<link data-trunk rel="rust">` — so the page sat on “Loading Craft Anything…”
forever and the app never started.

This fork therefore ships the browser viewer as prebuilt static files:

| Path | What it is |
| --- | --- |
| `apps/craft-anything-web/public/` | The static web app from `craft-anything-web-<version>.zip` (index.html, wasm-bindgen JS glue, the `.wasm` binary, licences) |
| `vercel.json` (repository root) | For a project whose Root Directory is `./`: serves `apps/craft-anything-web/public` and sets the headers from `packaging/web/README.md` |
| `apps/craft-anything-web/vercel.json` | For a project whose Root Directory is `apps/craft-anything-web`: serves its `public/` folder with the same headers |

Either Root Directory setting works, so an existing project does not have to be
recreated — it only needs a new deployment (push, or **Redeploy** in the Vercel
dashboard).

## Deploy

1. Open <https://vercel.com/new> and import the **craft-anything** repository.
2. Leave Framework Preset as **Other**, Root Directory as `./`, Build Command
   empty. `vercel.json` supplies the output directory.
3. Press **Deploy**. The project is live at `https://<project>.vercel.app`.

No environment variables, no server-side code, no database. A dropped or picked
file is read in the browser only; nothing is uploaded.

## Checking that the deploy really worked

Open the URL and look at the page source (or run `curl -s <url> | head`). A
working deployment has **no `data-trunk`** attribute and references
`craft-anything-web-<hash>.js` / `craft-anything-web-<hash>_bg.wasm`, both of
which must return `200`. A broken one still contains:

```html
<link data-trunk rel="rust" data-bin="craft-anything-web" ...>
```

and only ~1 KB of HTML: that is the template, not the app.

## If the repository is not in Vercel's list

That is a GitHub permission problem, not a build problem:

1. Open <https://github.com/settings/installations>.
2. Find **Vercel** and press **Configure**.
3. Under *Repository access* choose **All repositories**, or add
   `craft-anything` to the selected ones, then **Save**.

## Keeping the site current

`apps/craft-anything-web/public/` is a snapshot of the `vX.Y.Z` web release.
To refresh it after upgrading:

```sh
# 1. Grab the web bundle of the release you want
gh release download v0.6.1 -R storytold/craft-anything \
  -p 'craft-anything-web-*.zip' -O /tmp/web.zip

# 2. Replace the files Vercel serves
rm -rf apps/craft-anything-web/public && mkdir -p apps/craft-anything-web/public
unzip -j /tmp/web.zip 'craft-anything-web-*/index.html' \
  'craft-anything-web-*/*.js' 'craft-anything-web-*/*.wasm' \
  'craft-anything-web-*/LICENSE-*' 'craft-anything-web-*/NOTICE' \
  -d apps/craft-anything-web/public

git add apps/craft-anything-web/public
git commit -m "web: refresh the Vercel static build" && git push
```

Vercel redeploys on every push to the production branch (`main`).
