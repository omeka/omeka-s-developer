# Web Archive

## Server requirements

The player loads archive files using HTTP Range requests. If your server applies
`Content-Encoding` (for example, Apache `mod_deflate` gzipping the response) to a
web archive file, playback will fail: the encoded response omits `Content-Length`
and returns status `200` instead of `206` for Range requests, both of which the
player requires.

This most often affects uncompressed `.warc` files, since `.wacz` and `.warc.gz`
are already compressed and are usually left alone. When the module detects this
condition it shows a message instead of a broken player.

To resolve it, exclude web archive files from compression. For Apache, add to your
configuration:

```apache
SetEnvIfNoCase Request_URI \.warc$ no-gzip
```

## Asset build

The ReplayWeb.page JS assets (`asset/vendor/replaywebpage/ui.js`, `asset/vendor/replaywebpage/sw.js`) are copied from the `replaywebpage` npm package. To update the player version:

```sh
npm install
npm run build
```
