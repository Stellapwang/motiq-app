# motiq-app

The MotiQ pilot web app: a single self-contained HTML file that runs the drawing and
tapping battery on iPad. It is the prototype instrument used for feasibility work, not
the production instrument.

| Doc | What it covers |
|---|---|
| [SCOPE.md](SCOPE.md) | What this project is, how it deploys, branch and tag discipline |
| [CLAUDE.md](CLAUDE.md) | Universal working rules |
| [BACKLOG.md](BACKLOG.md) | Open work |

## Deploy

Production deploys from `main` to https://motiq-pilot-v01.netlify.app. There is no build
step: the publish directory is the repo root and the deployed artifact is `index.html`
and nothing else. The app has no external assets and loads nothing from a CDN.

Branch deploys are enabled — each pushed branch gets its own Netlify URL for testing.
Never deploy by hand; a manual upload produces a live file that matches no commit.

## Versioning

`index.html` carries its own version in `const BUILD`, and that string is written into
every record the app exports. Every release is tagged with the version field of `BUILD`
exactly, so `git checkout <build>` returns the code that produced a given record.

Filenames are not version identifiers and the numbering has real gaps. `BUILD` governs.
