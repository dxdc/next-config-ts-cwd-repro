# next.config.ts relative imports resolve against process.cwd()

`app/next.config.ts` imports `./src/headers`. Building the app works from inside
`app/`, but fails when the same app is built from its parent directory.

## Setup

```sh
cd app && npm install && cd ..
```

## 1. Works: cwd is the project directory

```sh
(cd app && npx next build)
```

## 2. Fails: same project, built from the parent directory

```sh
node app/node_modules/next/dist/bin/next build app
```

```
⨯ Failed to load next.config.ts
Error: Cannot find module './src/headers'
Require stack:
- .../app/next.config.compiled.js
```

`next dev app` from the parent directory fails the same way.

## 3. Worse: a file at the same path under cwd is loaded silently

```sh
mkdir -p src
printf "export const headers = [{ source: '/:path*', headers: [{ key: 'X-Origin', value: 'WRONG-cwd' }] }]\n" > src/headers.ts
node app/node_modules/next/dist/bin/next build app
node -p "JSON.stringify(require('./app/.next/routes-manifest.json').headers[0].headers)"
# => [{"key":"X-Origin","value":"WRONG-cwd"}]   (expected "app")
rm -rf src
```
