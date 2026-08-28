# Blender Remote Asset Library Template

A minimal template for publishing a folder of `.blend` files as a **remote asset
library** for Blender 5.2+. GitHub Actions generates the listing, GitHub Pages
serves it, and users add it under
`Preferences ▸ Asset Libraries ▸ Add Remote Asset Library`.

Nothing needs editing to get a working library. The library's name and contact
details are derived from the repository itself, and so are the URLs on the
landing page — so a copy of this template publishes correctly on its first push,
with the example asset in place, and you replace things at your own pace.

## Setup

1. **Use this template** to create your own repository.
2. Enable Pages, either in `Settings ▸ Pages` by setting **Source** to
   **GitHub Actions**, or from a clone of your repo with one command:

   ```sh
   gh api --method POST repos/{owner}/{repo}/pages -f build_type=workflow
   ```
3. Push.

Step 2 is the only manual step, and no workflow can do it for you.
`actions/configure-pages` does have an `enablement` option, but its own
documentation says it *"requires a token other than `GITHUB_TOKEN` to be
provided"* — a personal access token with `repo` scope, or a GitHub App with
`administration:write`. Creating and storing such a token is more setup than
the click it saves, so this template does not ask for one.

If you skip step 2 the build still succeeds and only the deploy fails, with a
job summary repeating the command above. Enable Pages and re-run the workflow.

Your library is then live at `https://<owner>.github.io/<repo>/`. That page
shows the URL to paste into Blender, plus a grid of everything the library
currently contains.

## Adding assets

Drop `.blend` files anywhere inside `assets/` (subfolders are fine — they are
not catalogs, just storage). Inside each file:

- Right-click a datablock and choose **Mark as Asset**. Only marked datablocks
  are published; an unmarked `.blend` contributes nothing.
- Give each asset a **preview**. Without one it still works, but the Asset
  Browser and the landing page show an empty tile.
- Assign each asset to a **catalog**. Catalogs are defined in
  `assets/blender_assets.cats.txt`, which must stay in the library root. The
  easiest way to keep it correct is to point Blender at `assets/` as a *local*
  asset library while authoring, create catalogs in the Asset Browser, and save
  them with `Catalogs ▸ Save Catalogs` — Blender then writes that file for you.

Delete `assets/example.blend` whenever you like; the build only fails if the
library ends up with no assets at all.

Commit the `.blend` files themselves. Note that this puts binaries in git
history; for a large library, enable Git LFS before the repo grows.

## Naming your library

By default the library is called after the repository — `my-scifi-props` shows
up in Blender as *My Scifi Props* — and the contact is your GitHub account. To
override either, add `assets/_asset-library-meta.json` with just the keys you
want to change:

```json
{
  "name": "My Sci-Fi Props",
  "contact": {
    "name": "Jane Doe",
    "url": "https://example.com",
    "email": "jane@example.com"
  }
}
```

Only `contact.name` is required by Blender; `url` and `email` are optional, and
the defaults leave the email out rather than inventing one.

## Testing locally

The build never writes to `assets/`, and neither should you — the generator
rewrites metadata in place and drops generated folders next to the `.blend`
files, so always run it against a copy:

```sh
rm -rf _site && mkdir _site && cp -R assets/. _site/
blender -b --factory-startup -c asset_listing generate _site
python3 .github/scripts/verify_listing.py _site
```

`verify_listing.py` re-hashes every referenced file and fails on an empty
library, a missing thumbnail or a hash that no longer matches — the same check
CI runs before anything is published. Locally it also prints a note about
placeholder metadata, because the name and contact are only filled in by the
workflow; that note is expected and harmless.

## Notes

- The Blender version is pinned once, as `BLENDER_VERSION` in
  `.github/workflows/build-asset-library.yml`. It selects the Blender that
  builds the listing and is echoed on the landing page.
- Blender caches remote listings. After publishing a new asset, select the
  library in the Asset Browser and use `Library ▸ Refresh Remote Listing` — the
  refresh button next to the library dropdown only re-reads the local cache.
- Remote libraries cannot be *linked*, only appended or packed.
- This template ships without a `LICENSE`. Add one that covers your assets;
  without it, nobody can legally reuse what you publish.
