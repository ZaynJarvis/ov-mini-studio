# OpenViking Mini Studio

A lightweight, read-only browser for an [OpenViking](https://github.com/volcengine/OpenViking)
context store, built from OpenViking Studio's components and design tokens.

- Browse `viking://` as a tree and preview entries with their L0 (abstract), L1 (overview), and
  L2 (content) levels. Markdown, JSON, and JSONL get rich renderers.
- Search with semantic **find**, regex **grep**, or **glob**, scoped to a top-level folder (⌘K).
- Copy a URI or a file's content to the clipboard (inside MiniMax Code these actions add to the chat
  input instead).
- Light and dark themes follow the system setting, with a manual override. Below 720 px the tree
  sits above the preview with a draggable divider.
- Configure the connection in the browser: a first-run dialog and Settings take the server URL,
  API key, and an optional account and user, and can test the connection before saving.

The UI uses [base-ui](https://base-ui.com) + Tailwind CSS v4 components copied from
[OpenViking Studio](https://github.com/volcengine/OpenViking/tree/main/web-studio). The data layer
and content renderers come from [Atlas](https://github.com/ZaynJarvis/atlas-fs). Everything is
bundled; nothing loads from a CDN.

## Run

Requires Node.js 20.12 or later.

```sh
npm install
npm run build
HOST=0.0.0.0 PORT=8790 node server.mjs
```

Open `http://<host>:8790/`. `HOST` defaults to `127.0.0.1` and `PORT` to `4380`.

For UI development, run `node server.mjs` (port 4380) and `npm run dev` in a second terminal. Vite
proxies `/ov` and `/ov-api` to the server.

## Connection config

| Variable | Default | Meaning |
| --- | --- | --- |
| `DATA_DIR` | `~/.config/openviking-studio-web` | Where `connection.json` is saved |
| `OVCLI_CONF` | `~/.openviking/ovcli.conf` | Optional default (`url`, `api_key`) used only while nothing is saved |

The connection is entered in the browser and saved by the server to `DATA_DIR/connection.json`
(written atomically, mode `0600`). If no connection is saved and `OVCLI_CONF` exists, its `url` and
`api_key` are used.

Server endpoints used by the page:

- `GET /ov/connection`: the URL, account, and user, plus `hasApiKey`. Never the key itself.
- `PUT /ov/connection`: save. A blank `apiKey` keeps the stored key; `clearApiKey: true` removes it.
- `POST /ov/test`: checks `GET /health`, then lists `viking://` with the submitted values (or the
  stored key). Nothing is saved.
- `/ov-api/*`: the read-only proxy.

## Security notes

- **The API key stays on the server.** The page never receives it. It is sent upstream only as
  `Authorization: Bearer` / `X-API-Key`, and never logged or echoed in errors.
- **The proxy is read-only and uses an allowlist.** The allowed routes are `GET /health`,
  `GET /api/v1/fs/{ls,tree,stat}`, `GET /api/v1/content/{read,abstract,overview}`, and
  `POST /api/v1/search/{find,grep,glob}`.
  - Any other method, path, or query key is rejected before any upstream call.
  - `uri` / `target_uri` must be `viking://` URIs.
  - Upstream redirects are refused, so the key cannot follow a redirect to another host.
- **The server has no authentication of its own.** Anyone who can reach the port can browse with
  your key and change the saved connection. Bind to `127.0.0.1` or put it behind your own auth when
  the network is not trusted.
- The HTML is served with a restrictive Content-Security-Policy (`connect-src 'self'`, no
  third-party origins).

## Layout

```text
server.mjs               Static dist/ + connection endpoints + /ov-api proxy
server/openviking.mjs    Connection storage and the read-only route allowlist
src/App.jsx              App shell: tree, preview, search, settings, first-run dialog
src/components/ui/       Studio UI components (base-ui + Tailwind)
src/legacy/              Atlas data layer (fs-api, OpenViking adapter) and renderers
src/styles.css           Studio design tokens and base layer
```

## License

[AGPL-3.0](./LICENSE). Third-party code: see [THIRD_PARTY_NOTICES.md](./THIRD_PARTY_NOTICES.md).
