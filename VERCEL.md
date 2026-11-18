# Deploying this fork on Vercel

Vercel's build image has no Rust or Trunk toolchain, so a plain import of this
repository fails: Vercel looks for a Node.js/web project, finds no build output,
and stops with **"No Output Directory named 'public' found"**.

This fork therefore ships the browser viewer as prebuilt static files, which is
the only part of Craft Anything that can run on Vercel:

| Path | What it is |
| --- | --- |
| `public/` | The static web app from `craft-anything-web-<version>.zip` (index.html, wasm-bindgen JS glue, the `.wasm` binary, licences) |
| `vercel.json` | Tells Vercel to skip the build and serve `public/` as-is, with the headers from `packaging/web/README.md` (`.wasm` as `application/wasm`, immutable caching for hashed assets, no-cache for `index.html`) |

## Deploy

1. Open <https://vercel.com/new>.
2. Pick the **craft-anything** repository from the list and press **Import**.
3. Leave every setting at its default (Framework Preset: **Other**, Root
   Directory: `./`, no Build Command, Output Directory: `public` — `vercel.json`
   supplies it) and press **Deploy**.
4. The project is live at `https://<project>.vercel.app`.

No environment variables, no server-side code, no database. The viewer reads a
dropped file in the browser only; nothing is uploaded.

## If the repository is not in Vercel's list

That is a GitHub permission problem, not a build problem. Grant the Vercel
GitHub App access to the repo:

1. Open <https://github.com/settings/installations>.
2. Find **Vercel** next to this account and press **Configure**.
3. Under *Repository access* choose **All repositories**, or **Only select
   repositories** and add `craft-anything`, then **Save**.
4. Reload <https://vercel.com/new>; the repo now appears.

Before this change the fork was a Rust/Cargo workspace only (apps, crates,
vendor), which no Vercel preset can build, so every import ended in a build
failure even after the permissions were correct.

## Keeping the site current

`public/` is a snapshot of the `vX.Y.Z` web release. To refresh it after
upgrading:

```sh
# 1. Grab the web bundle of the release you want
gh release download v0.6.1 -R storytold/craft-anything \
  -p 'craft-anything-web-*.zip' -O /tmp/web.zip

# 2. Replace the files Vercel serves (keep index.html, *.js, *.wasm, licences)
rm -rf public && mkdir public
unzip -j /tmp/web.zip 'craft-anything-web-*/index.html' \
  'craft-anything-web-*/*.js' 'craft-anything-web-*/*.wasm' \
  'craft-anything-web-*/LICENSE-*' 'craft-anything-web-*/NOTICE' -d public

git add public && git commit -m "web: refresh the Vercel static build" && git push
```

Vercel redeploys on every push to the production branch (`main`).
