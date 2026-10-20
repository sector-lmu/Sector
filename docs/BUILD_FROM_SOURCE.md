# Inspecting and building Sector from source

Sector's original application code is open source under the [MIT license](../LICENSE).
You can read it, change it and build its components yourself. You do not need to
run the downloaded EXE to inspect the project. Dependencies, fonts and vehicle
logos retain their [own terms](../src/ui/electron/THIRD_PARTY_NOTICES.md).

**Current limitation:** a clean clone can build and preview the desktop UI, but
does not contain every input needed to reproduce the complete working Windows
package. Native telemetry and Active Reset also require the matching LMU SDK,
local compatibility profiles and recorded verification inputs. Those are not all
distributed in Git. This guide distinguishes a source build from a package that
reuses downloaded binaries.

## Get the code for the version you want to inspect

Install Git, then open PowerShell in a folder of your choice:

```powershell
git clone https://github.com/sector-lmu/Sector.git
cd Sector
git checkout v1.0.0
```

This checks out the 1.0.0 release tag. A detached HEAD at a release tag is normal.
Use the tag matching your download when reviewing another version. `prod` is the
published branch; `dev` includes ongoing work that may not be released yet.

For changes of your own, create a local branch:

```powershell
git switch -c my-sector-changes
```

## Where to look

| Area | Source |
| --- | --- |
| Desktop, windows and privileged UI actions | [`src/ui/electron/main`](../src/ui/electron/main) |
| Interface and overlay widgets | [`src/ui/electron/renderer`](../src/ui/electron/renderer) |
| EAC and offline-session checks in the desktop | [`reset-access.cjs`](../src/ui/electron/main/reset-access.cjs) |
| Runtime supervision and reset commands | [`src/python/sector/runtime`](../src/python/sector/runtime) |
| Session-context validation | [`context_gate.py`](../src/python/sector/telemetry/context_gate.py) |
| Native checkpoint, physics and restoration logic | [`src/native`](../src/native) |
| Read-only overlay telemetry | [`overlay_reader.cpp`](../src/native/telemetry/overlay_reader.cpp) |
| Tests | [`tests`](../tests) and [`desktop tests`](../src/ui/electron/tests) |
| Packaging, copied files and binary/profile checks | [`release.mjs`](../src/ui/electron/build/release.mjs) |

The [source map](SOURCE_LAYOUT.md) describes the components in more detail.

Active Reset is intended for offline practice. The desktop blocks reset actions
when EAC is detected; the runtime also checks session context, game compatibility
and EAC before attachment. Missing or unsupported context is rejected. These
checks are safeguards, not a claim that software cannot contain bugs. The overlay
reader is a separate path. Do not remove these checks to use reset online or
interfere with other players.

## Build the desktop preview

Use Windows x64 and Node.js 22.12 or later with npm. From the repository root:

```powershell
cd src/ui/electron
npm ci
node node_modules/electron/install.js
npm run build
npm run preview
```

`npm ci` installs the locked dependencies and can execute their installation
scripts. Electron's explicit install command ensures its runtime is present.
Review the package and lockfile before running installation commands if that is
part of your audit.

Preview uses simulated data and a separate local profile. It does not start the
reset listener or attach to LMU. It lets you inspect and change the interface and
widgets, but is not a functional game-connected build.

To run the desktop unit tests from that same folder:

```powershell
npm test
```

With Python 3.11 or later, run the Python tests from the repository root:

```powershell
python scripts/sector.py test
```

Tests and previews do not prove that a particular live reset scenario works.
See [verification and known limits](TEST_RESULTS.md).

## Build the game-connected components and portable app

The complete build needs additional prerequisites:

- Windows x64, PowerShell 7 and Zig 0.16.0.
- A supported local LMU installation and matching shared-memory SDK headers.
- Locally verified compatibility profiles for that exact game build.
- Recorded fixture inputs for the reset-host verification.
- The pinned embedded Python archive used by the packager.

The [release guide](RELEASE.md) lists the build, packaging and packaged-EXE test
commands. Its full build command is for a configured development environment:

```powershell
./scripts/build/Build-ResetDesktop.ps1 -ZigPath 'C:\tools\zig\zig.exe' -FrameDirectory 'C:\path\to\verified-frames'
```

Replace both paths with your actual prerequisites. This command is not expected
to work on a fresh clone alone. In particular, the telemetry profile contains
verified installation file hashes and a local installation path; the build checks
those files before proceeding. A version string or an arbitrary hash does not
establish compatibility. Do not invent profile values or skip the checks.

Game binaries, proprietary SDK headers, dumps and local research captures are
not included in the repository. If you do not have the required inputs, you can
still inspect the source and build the UI preview. There is not yet a fully
self-contained, clean-clone recipe for reproducing the entire release.

Copying DLLs and EXEs from a downloaded Sector package into a source checkout
still means trusting those downloaded binaries. That is not a fully independent
source build. Likewise, a successful build does not currently guarantee a
byte-for-byte match with a published package.

## Check a downloaded archive

GitHub releases include `SHA256SUMS.txt`. After downloading the Windows ZIP, run:

```powershell
Get-FileHash -Algorithm SHA256 -LiteralPath '.\Sector-1.0.0-win-x64.zip'
```

Compare the result with the filename's entry in `SHA256SUMS.txt`. This checks that
your file matches the published checksum; it is not an independent security
audit or proof that the executable was built from the public source.

## Questions and contributions

Use [GitHub Issues](https://github.com/sector-lmu/Sector/issues) for build problems,
bugs or suggestions. Include the version or commit, the command and its error.
Do not post game dumps, credentials or private paths. See
[Contributing](../CONTRIBUTING.md) before submitting a change.
