<h1 align="center">
  <br>
  <a href="https://webtorrent.io">
    <img src="https://webtorrent.io/img/WebTorrent.png" alt="WebTorrent" width="200">
  </a>
  <br>
  WebTorrent Desktop
  <br>
  <br>
</h1>

<h4 align="center">The streaming torrent app. For Mac, Windows, and Linux.</h4>

> **Fork notice.** This is the [`psifertex/webtorrent-desktop`](https://github.com/psifertex/webtorrent-desktop) fork. It exists to publish a **universal macOS build** that runs natively on both Apple Silicon (M1/M2/M3/…) and Intel Macs — addressing the long-standing upstream gap tracked in [webtorrent/webtorrent-desktop#1907](https://github.com/webtorrent/webtorrent-desktop/issues/1907) (open since Dec 2020). The Windows and Linux build flow is unchanged from upstream; use the upstream releases or build locally for those platforms.

## Install

### macOS (this fork's universal build)

Download the universal `.zip` from the **[fork's releases page](https://github.com/psifertex/webtorrent-desktop/releases)**. The artifact (`WebTorrent-v<version>-darwin-universal.zip`) contains a single `.app` whose binaries are fat Mach-O — it runs natively on both `arm64` (Apple Silicon) and `x86_64` (Intel), no Rosetta needed.

**The fork build is not signed or notarized.** macOS Gatekeeper will refuse to open it on first launch. Pick one workaround:

1. **Right-click → Open** (recommended). After unzipping and dragging `WebTorrent.app` into `/Applications`, right-click (or Control-click) the app and choose **Open**, then click **Open** in the dialog. macOS remembers the exception.

2. **Strip the quarantine attribute** from the command line:

   ```sh
   xattr -cr /Applications/WebTorrent.app
   ```

   `-c` clears all extended attributes (including `com.apple.quarantine`); `-r` recurses into the bundle. Run this once after copying the app to `/Applications`.

3. **System Settings fallback.** If Gatekeeper still blocks it, open **System Settings → Privacy & Security**, scroll to the bottom, and click **Open Anyway** next to the WebTorrent entry that appears after your first blocked launch attempt.

4. **Sign it yourself** with an ad-hoc signature (no Apple Developer account needed):

   ```sh
   codesign --force --deep --sign - /Applications/WebTorrent.app
   ```

   This produces a locally-trusted signature that satisfies Gatekeeper on the same machine. It will *not* satisfy Gatekeeper on other machines — for true distribution you need a Developer ID certificate and notarization.

### Windows and Linux

This fork does not ship Windows or Linux builds. Use upstream:

- [✨ Download WebTorrent Desktop ✨](https://webtorrent.io/desktop/) (official upstream site)
- [Upstream GitHub releases](https://github.com/webtorrent/webtorrent-desktop/releases) for specific installer files
- [Homebrew-Cask](https://github.com/caskroom/homebrew-cask): `brew install --cask webtorrent` (Mac/Linux)
- Try the (unstable) development version by cloning the Git repository. See the
  ["How to Contribute"](#how-to-contribute) instructions.

## Screenshots

<p align="center">
  <img src="https://webtorrent.io/img/screenshot-player3.png" alt="screenshot" align="center">
  <img src="https://webtorrent.io/img/screenshot-main.png" width="612" height="749" alt="screenshot" align="center">
</p>

## How to Contribute

### Get the code

```
$ git clone https://github.com/psifertex/webtorrent-desktop.git
$ cd webtorrent-desktop
$ npm install
```

### Run the app

```
$ npm start
```

### Watch the code

Restart the app automatically every time code changes. Useful during development.

```
$ npm run watch
```

### Run linters

```
$ npm test
```

### Run integration tests

```
$ npm run test-integration
```

The integration tests use Spectron and Tape. They click through the app, taking screenshots and
comparing each one to a reference. Why screenshots?

* Ad-hoc checking makes the tests a lot more work to write
* Even diffing the whole HTML is not as thorough as screenshot diffing. For example, it wouldn't
  catch an bug where hitting ESC from a video doesn't correctly restore window size.
* Chrome's own integration tests use screenshot diffing iirc
* Small UI changes will break a few tests, but the fix is as easy as deleting the offending
  screenshots and running the tests, which will recreate them with the new look.
* The resulting Github PR will then show, pixel by pixel, the exact UI changes that were made! See
  https://github.com/blog/817-behold-image-view-modes

For MacOS, you'll need a Retina screen for the integration tests to pass. Your screen should have
the same resolution as a 2018 MacBook Pro 13".

For Windows, you'll need Windows 10 with a 1366x768 screen.

When running integration tests, keep the mouse on the edge of the screen and don't touch the mouse
or keyboard while the tests are running.

### Package the app

Builds app binaries for Mac, Linux, and Windows.

```
$ npm run package
```

To build for one platform:

```
$ npm run package -- [platform] [options]
```

Where `[platform]` is `darwin`, `linux`, `win32`, or `all` (default).

The following optional arguments are available:

- `--sign` - Sign the application (Mac, Windows)
- `--arch=[list]` - Override the default architecture list for the target platform. Comma-separated; defaults below.
   - `darwin` default: `x64,arm64`. Use `--arch=universal` for a single fat Mach-O binary (recommended for distribution).
   - `linux` default: `x64,armv7l,arm64`.
   - `win32` is x64 only.
- `--skipInstall` - Skip the automatic `npm ci` step. Useful for iterative testing once `node_modules` is already in place.
- `--package=[type]` - Package single output type.
   - `deb` - Debian package
   - `rpm` - RedHat package
   - `zip` - Linux/Mac zip file (Mac zip names include the arch, e.g. `-darwin-universal.zip`)
   - `dmg` - Mac disk image
   - `exe` - Windows installer
   - `portable` - Windows portable app
   - `all` - All platforms (default)

Example — produce just the universal macOS zip used by this fork's release:

```sh
$ npm run package -- darwin --arch=universal --package=zip
```

Note: Even with the `--package` option, the auto-update files (.nupkg for Windows,
-darwin.zip for Mac) will always be produced.

#### Windows build notes

The Windows app can be packaged from **any** platform.

Note: Windows code signing only works from **Windows**, for now.

Note: To package the Windows app from non-Windows platforms,
[Wine](https://www.winehq.org/) and [Mono](https://www.mono-project.com/) need
to be installed. For example on Mac, first install
[XQuartz](http://www.xquartz.org/), then run:

```
$ brew install wine mono
```

(Requires the [Homebrew](http://brew.sh/) package manager.)

#### Mac build notes

The Mac app can only be packaged from **macOS**.

#### Linux build notes

The Linux app can be packaged from **any** platform.

If packaging from Mac, install system dependencies with Homebrew by running:

```
npm run install-system-deps
```
#### Recommended readings to start working in the app

Electron (Framework to make native apps for Windows, OSX and Linux in Javascript):
https://electronjs.org/docs/tutorial/quick-start

React.js (Framework to work with Frontend UI):
https://reactjs.org/docs/getting-started.html

Material UI (React components that implement Google's Material Design.):
https://material-ui.com/getting-started/installation

### Privacy

WebTorrent Desktop collects some basic usage stats to help us make the app better.
For example, we track how well the play button works. How often does it succeed?
Time out? Show a missing codec error?

The app never sends any personally identifying information, nor does it track which
torrents you add.

## License

MIT. Copyright (c) [WebTorrent, LLC](https://webtorrent.io).
