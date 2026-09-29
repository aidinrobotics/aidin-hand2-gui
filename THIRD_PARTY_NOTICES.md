# Third-Party Notices

The AIDIN Hand Gen2 GUI is licensed under the Apache License 2.0; see [LICENSE](LICENSE) and
[NOTICE](NOTICE). Its `.deb` package also carries third-party software. This file lists each
component, the license it is provided under, and where its source can be obtained. Full
license texts are in [licenses/](licenses).

The package installs the app in `/opt/AIDIN Hand GUI/`. This file, `LICENSE`, `NOTICE` and
`licenses/` are installed in the same folder, next to the two notice files that Electron
provides: `LICENSE.electron.txt` and `LICENSES.chromium.html`.

Components reach the package in four ways, and each section below covers one of them.

- **Electron.** The app runs on Electron, which brings Chromium, Node.js and FFmpeg with it.
- **Compiled into web_bridge.** `resources/bin/aidin_hand2_web_bridge` is a C++ program built
  with uWebSockets and uSockets.
- **Bundled into the page.** The page in `resources/dist/` is built with Vite, which bundles
  the npm packages it imports into `assets/main-<hash>.js`. The bundler removes license
  comments from that file, so this file carries the notices for those packages.
- **Compiled into the joint limit module.** `assets/clamp-<hash>.js` and
  `assets/clamp-<hash>.wasm` are built with Emscripten, which adds parts of its runtime and
  system libraries.

## Components

| Component | Version | License | In the package |
|---|---|---|---|
| [Electron](https://www.electronjs.org) | 44.4.5 | MIT | `aidin-hand2-gui` and the files beside it |
| [Chromium](https://www.chromium.org) | 152.0.7977.130 | BSD-3-Clause and others | Inside Electron |
| [Node.js](https://nodejs.org) | as bundled with Electron 44.4.5 | MIT and others | Inside Electron |
| [FFmpeg](https://ffmpeg.org) | Chromium's copy at revision `2b68d2b` | LGPL-2.1-or-later | `libffmpeg.so` |
| [uWebSockets](https://github.com/uNetworking/uWebSockets) | 20 | Apache-2.0 | `resources/bin/aidin_hand2_web_bridge` |
| [uSockets](https://github.com/uNetworking/uSockets) | as included with uWebSockets | Apache-2.0 | `resources/bin/aidin_hand2_web_bridge` |
| [any_invocable](https://github.com/ofats/any_invocable) | as included with uWebSockets | MIT | `resources/bin/aidin_hand2_web_bridge` |
| [React](https://react.dev) (`react`) | 18.3.1 | MIT | `resources/dist/assets/main-<hash>.js` |
| React DOM (`react-dom`) | 18.3.1 | MIT | `resources/dist/assets/main-<hash>.js` |
| Scheduler (`scheduler`) | 0.23.2 | MIT | `resources/dist/assets/main-<hash>.js` |
| [three.js](https://threejs.org) (`three`) | 0.185.0 | MIT | `resources/dist/assets/main-<hash>.js` |
| [uPlot](https://github.com/leeoniya/uPlot) (`uplot`) | 1.6.32 | MIT | `resources/dist/assets/main-<hash>.js` |
| [urdf-loader](https://github.com/gkjohnson/urdf-loaders) (`urdf-loader`) | 0.13.0 | Apache-2.0 | `resources/dist/assets/main-<hash>.js` |
| [Vite](https://vite.dev) (`vite`) | 8.1.0 | MIT | `resources/dist/assets/main-<hash>.js` |
| [Rolldown](https://rolldown.rs) (`rolldown`) | 1.1.3 | MIT | `resources/dist/assets/main-<hash>.js` |
| [Emscripten](https://emscripten.org) | the release that built `clamp.wasm` | MIT or NCSA | `resources/dist/assets/clamp-<hash>.js` and `clamp-<hash>.wasm` |
| musl libc | as shipped with that Emscripten | MIT | `resources/dist/assets/clamp-<hash>.wasm` |
| libc++ and libc++abi | as shipped with that Emscripten | Apache-2.0 WITH LLVM-exception | `resources/dist/assets/clamp-<hash>.wasm` |
| dlmalloc | as shipped with that Emscripten | CC0-1.0 | `resources/dist/assets/clamp-<hash>.wasm` |

## Electron, Chromium and Node.js

Electron is licensed under the MIT license. Its text is in
[licenses/MIT-electron.txt](licenses/MIT-electron.txt), and the package installs the same text
as `LICENSE.electron.txt`.

Chromium, Node.js and the libraries they include are listed with their full license texts in
`LICENSES.chromium.html`. Electron provides that file with each release, and the package
installs it next to the app. It is about 20 MB, so this repository does not keep a copy. The
same file is inside `electron-v44.4.5-linux-x64.zip` and `electron-v44.4.5-linux-arm64.zip` on
the Electron release page.

Source:

- Electron: https://github.com/electron/electron/tree/v44.4.5
- Chromium: https://chromium.googlesource.com/chromium/src/+/refs/tags/152.0.7977.130

## FFmpeg

Chromium plays audio and video through FFmpeg, and Electron loads `libffmpeg.so` when it
starts. The app itself neither plays nor records audio or video.

The package does not use the `libffmpeg.so` that comes with Electron. It uses the build that
Electron publishes separately on the same release page as `ffmpeg-v44.4.5-linux-x64.zip` and
`ffmpeg-v44.4.5-linux-arm64.zip`. That build leaves out the proprietary codecs, including
H.264 and AAC. AIDIN ROBOTICS does not modify it, and the archive is checked against the
SHA-256 sums that Electron publishes with the release.

FFmpeg is licensed under the GNU Lesser General Public License version 2.1 or later, and some
of its files carry MIT, X11 or BSD-style terms. FFmpeg's license statement and the text of the
LGPL 2.1 are in [licenses/LGPL-2.1-FFmpeg.txt](licenses/LGPL-2.1-FFmpeg.txt), as Chromium
ships them.

`libffmpeg.so` is a separate shared library in `/opt/AIDIN Hand GUI/`, so it can be replaced
with another build that provides the same interface.

Source:

- FFmpeg as Chromium builds it, at the revision that Chromium 152.0.7977.130 pins:
  https://chromium.googlesource.com/chromium/third_party/ffmpeg/+/2b68d2babae73714846961fb0ee47e3b3d2e39a9
- The patch that Electron applies to it:
  https://github.com/electron/electron/tree/v44.4.5/patches/ffmpeg

## uWebSockets, uSockets and any_invocable

web_bridge serves the page and its WebSocket connection with uWebSockets, which runs on
uSockets. Both are licensed under the Apache License 2.0, whose text is in
[licenses/Apache-2.0.txt](licenses/Apache-2.0.txt). Their source files carry the notice
`Authored by Alex Hultman, 2018-<year>. Intellectual property of third-party.`

uWebSockets includes any_invocable by Oleg Fatkhiev in `src/MoveOnlyFunction.h`. It is
licensed under the MIT license, whose text is in
[licenses/MIT-any_invocable.txt](licenses/MIT-any_invocable.txt).

Source:

- uWebSockets: https://github.com/uNetworking/uWebSockets
- uSockets: https://github.com/uNetworking/uSockets
- any_invocable: https://github.com/ofats/any_invocable

## Page libraries

| Package | Copyright | License text |
|---|---|---|
| `react`, `react-dom`, `scheduler` | Copyright (c) Facebook, Inc. and its affiliates. | [licenses/MIT-react.txt](licenses/MIT-react.txt). The three packages carry the same LICENSE file |
| `three` | Copyright © 2010-2026 three.js authors | [licenses/MIT-three.txt](licenses/MIT-three.txt) |
| `uplot` | Copyright (c) 2022 Leon Sorokin | [licenses/MIT-uplot.txt](licenses/MIT-uplot.txt) |
| `urdf-loader` | Copyright 2020 California Institute of Technology | [licenses/Apache-2.0-urdf-loader.txt](licenses/Apache-2.0-urdf-loader.txt) |
| `vite` | Copyright (c) 2019-present, VoidZero Inc. and Vite contributors | [licenses/MIT-vite.txt](licenses/MIT-vite.txt) |
| `rolldown` | Copyright (c) 2024-present VoidZero Inc. & Contributors | [licenses/MIT-rolldown.txt](licenses/MIT-rolldown.txt) |

The modules from `three/examples/jsm` in the bundle, such as `STLLoader` and `OrbitControls`,
are part of the `three` package and are under the same license.

Vite and Rolldown are build tools. The page contains only the runtime helpers that they add to
the bundle: `modulepreload-polyfill` and `preload-helper` from Vite, and Rolldown's module
runtime.

The urdf-loader npm package does not include a LICENSE file. The text in
[licenses/Apache-2.0-urdf-loader.txt](licenses/Apache-2.0-urdf-loader.txt) is the LICENSE of
the urdf-loaders repository at tag `v0.13.0`. The project's README adds this notice:

> Copyright © 2020 California Institute of Technology. ALL RIGHTS RESERVED. United States
> Government Sponsorship Acknowledged. Neither the name of Caltech nor its operating division,
> the Jet Propulsion Laboratory, nor the names of its contributors may be used to endorse or
> promote products derived from this software without specific prior written permission.

Source: each package's page on the npm registry, `https://www.npmjs.com/package/<name>`, and
the repositories linked in the Components table.

## Emscripten

`clamp-<hash>.js` and `clamp-<hash>.wasm` contain the joint limit code of the AIDIN Hand Gen2
SDK, compiled with Emscripten. The SDK is licensed under the Apache License 2.0 by AIDIN
ROBOTICS Inc., and its source is at https://github.com/aidinrobotics/aidin-hand2-sdk.
Emscripten adds its JavaScript runtime, the embind binding library and parts of the system
libraries below.

| Component | License | License text |
|---|---|---|
| Emscripten runtime and embind | MIT, or the University of Illinois/NCSA Open Source License | [licenses/MIT-or-NCSA-emscripten.txt](licenses/MIT-or-NCSA-emscripten.txt) |
| musl libc | MIT | [licenses/MIT-musl.txt](licenses/MIT-musl.txt) |
| libc++ | Apache-2.0 WITH LLVM-exception | [licenses/Apache-2.0-WITH-LLVM-exception-libcxx.txt](licenses/Apache-2.0-WITH-LLVM-exception-libcxx.txt) |
| libc++abi | Apache-2.0 WITH LLVM-exception | [licenses/Apache-2.0-WITH-LLVM-exception-libcxxabi.txt](licenses/Apache-2.0-WITH-LLVM-exception-libcxxabi.txt) |
| dlmalloc | CC0-1.0. Doug Lea released it to the public domain | No text is required |

Source: https://github.com/emscripten-core/emscripten. The system libraries are in its
`system/lib/` directory.

## Installed separately

The package does not include the following software. Each one comes with its own notices.

- **The AIDIN Hand Gen2 SDK.** web_bridge links `libaidin_hand2.so`, which the user installs
  from https://github.com/aidinrobotics/aidin-hand2-sdk. That repository's
  `THIRD_PARTY_NOTICES.md` covers the SDK.
- **Ubuntu libraries.** The packages in the `Depends` field of the `.deb`, such as GTK and
  NSS, and the C and C++ runtime are installed by Ubuntu, with their copyright files in
  `/usr/share/doc/`.
