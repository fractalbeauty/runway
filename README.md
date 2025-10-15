# Runway
Runway is an asset uploader for Roblox projects.

It maps paths in your project to asset strings
and supports fast local development or uploads using Open Cloud.

Runway borrows from [Tarmac](https://github.com/rojo-rbx/tarmac)
but doesn't do all of its resource compilation (specifically spritesheet packing and DPI scaling).
Instead, Runway's inputs/outputs should (ideally) be simple and reliable enough
to build other tools on top of them.

Runway applies alpha bleeding to all `png` assets that have transparency.

## Installation

### With Aftman (recommended)
Install [Aftman](https://github.com/LPGhatguy/aftman),
then add an entry to the `[tools]` section of `aftman.toml`:
```toml
runway = "fractalbeauty/runway@0.2.0"
```

### From releases
Pre-built binaries are available [here](https://github.com/fractalbeauty/runway/releases).

### From source
Clone the repository and run:
```bash
cargo install --path .
```

### Usage

### Configuration
Runway is configured with a `runway.toml` file at the root of your project.
Paths within `runway.toml` should be relative to the project root.

Config files require a name and can have any number of targets, inputs, and codegen outputs.
The minimum (useful) config is a name and one of each:
```toml
name = "my-project"

[[target]]
type = "local"

[[input]]
glob = "assets/**/*.png"

[[codegen]]
format = "lua"
path = "src/assets.lua"
```
With this config, we can run `runway sync --target local` to make our images accessible from Studio
using the asset strings listed in `assets.lua`.

Input globs use [`.gitignore`'s syntax](https://git-scm.com/docs/gitignore#_pattern_format).
You can add another glob by adding another `[[input]]` section.

To upload assets to Roblox using the Open Cloud API, use the `roblox` target type.
Syncing to Roblox requires `--api-key` and either `--user-id` or `--group-id`.
We can also give each target a key which is used by the `--target` argument and keys the upload state.
```toml
[[target]]
key = "production"
type = "roblox"
```

Runway can output asset paths as `json`, `lua`, `ts`, or `d.ts` files.
You can specify multiple outputs by adding more `[[codegen]]` sections.
There are some additional options available per output:
```toml
[[codegen]]
format = "ts"
path = "src/assets.ts"

flatten = false # Defaults to false, makes the output map flat instead of nesting by path
strip_prefix = "assets" # Defaults to none, removes leading path from output map
strip_extension = true # Defaults to true, removes extension from output map
```

### State
Syncing will generate `runway-state.toml` and `runway-state.local.toml` files
containing the uploaded asset IDs and hashes of their contents for detecting changes.

The local state file should not be checked in to version control.
The Roblox state file is useful for skipping uploading assets to Roblox that haven't changed.

Local syncs will also create a `.runway` directory with copies of locally synced assets.
This folder should not be checked in and can be safely deleted at any time.

### Global options

* `-h`, `--help`, `-V`, `--version`
	* Does that you think
* `-v`, `--verbose`,
	* Logs more details. Use twice for even more verbosity
* `-q`, `--quiet`
	* Logs less output
* `-t`, `--target <key>`
	* Target's key or type if key is unspecified
* `-c`, `--config [path]`
	* Path to file or directory containing config
	* Defaults to current directory

#### Syncing to Roblox

These options can also be read from the listed environment variables.

* `-a`, `--api-key <key>`, `RUNWAY_API_KEY=`
	* [Open Cloud API key](https://create.roblox.com/docs/cloud/open-cloud/managing-api-keys)
* `-u`, `--user-id <id>`, `RUNWAY_USER_ID=`
	* User ID to upload as
* `-g`, `--group-id <id>`, `RUNWAY_GROUP_ID=`
	* Group ID to upload as

### `runway sync`

Finds files matched by configured inputs
and syncs changed assets to the specified target,
then generates configured outputs.

Examples:
```
runway sync --target local
runway sync --target roblox --api-key <key> --user-id <id>
```

Additional options:
* `-f`, `--force`
	* Skips checking if files are changed and syncs everything

### `runway watch`

Watches a project for new/changed inputs and runs the sync process automatically.

### `runway codegen`

Reads previous state and generates outputs without syncing.

### Supported asset types

See the [Open Cloud assets docs](https://create.roblox.com/docs/cloud/open-cloud/usage-assets) for more details.

`bmp` and `tga` images don't seem to work.
Models haven't been tested yet.

| Extension           | Local | Roblox  |
| ------------------- |:-----:|:-------:|
| `.png`              | Yes   | Yes     |
| `.jpg`<br />`.jpeg` | Yes   | Yes     |
| `.bmp`              | No    | No      |
| `.tga`              | No    | No      |
| `.mp3`              | Yes   | Yes[^1] |
| `.ogg`              | Yes   | Yes[^1] |
| `.fbx`              | ?     | ?       |

[^1]: Some extra manual work might be necessary because of
[audio asset privacy](https://create.roblox.com/docs/sound/assets#audio-asset-privacy-system)

## License
Runway is available under the MIT license. See [LICENSE.txt](LICENSE.txt).

Alpha bleeding uses code from the Tarmac source
([`alpha_bleed.rs`](https://github.com/Roblox/tarmac/blob/ea0d23dc5138d40f7b3de05544a07414416a909a/src/alpha_bleed.rs),
[`image.rs`](https://github.com/Roblox/tarmac/blob/master/src/image.rs)), licensed under the MIT license.
