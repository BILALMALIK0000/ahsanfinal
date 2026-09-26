# Error Resolution: better-sqlite3 Native Binding

## Error

The server failed during startup with:


Error: Could not locate the bindings file


The failure occurred when `server.js` created the SQLite database with
`new Database(...)`.

## Cause

The installed `better-sqlite3@11.10.0` package did not contain a native
binding for the current environment:

- Windows x64
- Node.js `24.20.0`
- Missing `better_sqlite3.node` binary

Rebuilding from source was also unavailable because `node-gyp` could not find
a usable Python installation on the machine.

## Resolution Applied

1. Checked the database initialization path in `server.js`.
2. Tried rebuilding `better-sqlite3` from source.
3. Confirmed that source compilation was blocked by the missing Python build
   prerequisite.
4. Checked the current `better-sqlite3` release compatibility.
5. Updated `better-sqlite3` to `13.0.3`, which supports Node.js `22` and newer.
6. Let npm install the matching native binding and update `package-lock.json`.
7. Started the application and checked the health endpoint.

## Page Routing Fix

The server also returned errors such as:

```text
Error: ENOENT: no such file or directory, stat 'C:\Users\Aptech_BLD\Desktop\PRESSZILA\admin.html'
```

The HTML pages are stored in the `public/` directory, but `server.js` was
searching for them in the project root. The server was updated to:

- Serve static files from `public/`.
- Load all configured page routes from `public/`.

This fixed the `/`, `/home.html`, `/admin.html`, and `/assets/css/style.css`
routes. Each returned HTTP `200` during verification.

## Verification

The following command now starts the server successfully:


npm.cmd start


The health check returned:


{"ok":true,"website":"PRESSZILA","status":"running"}


Check it at `http://localhost:3000/api/health`.

## Future Installation

Use:


npm.cmd install
npm.cmd start


Do not copy `node_modules` between machines or Node.js versions. If the
binding is missing after an environment change, reinstall dependencies with
`npm.cmd install`.

## Alternative Recovery

If npm must compile `better-sqlite3` from source, install Python 3 and the
Visual Studio C++ Build Tools, then run:


npm.cmd rebuild better-sqlite3 --build-from-source


The upgraded dependency is preferred because it avoids requiring that local
native build toolchain when a compatible prebuilt binding is available.