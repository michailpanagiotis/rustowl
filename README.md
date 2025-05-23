<div align="center">
    <h1>
      <picture>
        <source media="(prefers-color-scheme: dark)" srcset="docs/assets/rustowl-logo-dark.svg">
        <img alt="RustOwl" src="docs/assets/rustowl-logo.svg" width="400">
      </picture>
    </h1>
    <p>
        Visualize ownership and lifetimes in Rust for debugging and optimization
    </p>
    <p>
        <img src="docs/assets/readme-screenshot-3.png" />
    </p>
</div>

RustOwl visualizes ownership movement and lifetimes of variables.
When you save Rust source code, it is analyzed, and the ownership and lifetimes of variables are visualized when you hover over a variable or function call.

RustOwl visualizes those by using underlines:

- 🟩 green: variable's actual lifetime
- 🟦 blue: immutable borrowing
- 🟪 purple: mutable borrowing
- 🟧 orange: value moved / function call
- 🟥 red: lifetime error
    - diff of lifetime between actual and expected, or
    - invalid overlapped lifetime of mutable and shared (immutable) references

Detailed usage is described [here](docs/usage.md).

Currently, we offer VSCode extension, Neovim plugin and Emacs package.
For these editors, move the text cursor over the variable or function call you want to inspect and wait for 2 seconds to visualize the information.
We implemented LSP server with an extended protocol.
So, RustOwl can be used easily from other editor.

## Support

If you're looking for support, please consider checking all issues, existing discussions, and [starting a discussion](https://github.com/cordx56/rustowl/discussions/new?category=q-a) first!

## Quick Start

Here we describe how to start using RustOwl with VS Code.

### Prerequisite

- `rustup` installed
    - You can install `rustup` from [this link](https://rustup.rs/).
- Visual Studio Code (VS Code) installed

We tested this guide on macOS Sequoia 15.3.2 on arm64 architecture with VS Code 1.99.1 and `rustup` 1.28.1.

We also tested this guide on Ubuntu 25.04.
On Ubuntu, you need to run `apt install build-essential` before installing.

After installation, the extension will automatically run RustOwl when you save any Rust program in cargo workspace.
The initial analysis may take some time, but from the second run onward, compile caching is used to reduce the analysis time.

We tested on Windows 11 Education 23H2 on amd64 architecture.

### Install RustOwl

Install from crates.io:

```bash
cargo install rustowl -F installer --locked
```

Or, install from GitHub release (using sh, Unix-like / WSL only):

```bash
curl -L "https://github.com/cordx56/rustowl/releases/latest/download/install.sh" | sh
```

### VS Code

You can install VS Code extension from [this link](https://marketplace.visualstudio.com/items?itemName=cordx56.rustowl-vscode).

## Other editor support

We support Neovim and Emacs.
You can also create your own LSP client.
If you would like to implement a client, please refer to the [The RustOwl LSP specification](docs/lsp-spec.md).

### Neovim

Minimal setup with [lazy.nvim](https://github.com/folke/lazy.nvim):

```lua
{
  'cordx56/rustowl',
  version = '*', -- Latest stable version
  build = 'cd rustowl && cargo install --path . -F installer --locked',
  lazy = false, -- This plugin is already lazy
  opts = {},
}
```

<details>
<summary>Recommended configuration: <b>Click to expand</b></summary>

```lua
{
  'cordx56/rustowl',
  version = '*', -- Latest stable version
  build = 'cd rustowl && cargo install --path . --locked',
  lazy = false, -- This plugin is already lazy
  opts = {
    client = {
      on_attach = function(_, buffer)
        vim.keymap.set('n', '<leader>o', function()
          require('rustowl').toggle(buffer)
        end, { buffer = buffer, desc = 'Toggle RustOwl' })
      end
    },
  },
}
```

</details>

Default options:

```lua
{
  auto_attach = true, -- Auto attach the RustOwl LSP client when opening a Rust file
  auto_enable = false, -- Enable RustOwl immediately when attaching the LSP client
  idle_time = 500, -- Time in milliseconds to hover with the cursor before triggering RustOwl
  client = {}, -- LSP client configuration that gets passed to `vim.lsp.start`
}
```

When opening a Rust file, the Neovim plugin creates the `Rustowl` user command:

```vim
:Rustowl {subcommand}
```

where `{subcommand}` can be one of:

- `start_client`: Start the rustowl LSP client.
- `stop_client`: Stop the rustowl LSP client.
- `restart_client`: Restart the rustowl LSP client.
- `enable`: Enable rustowl highlights.
- `disable`: Disable rustowl highlights.
- `toggle`: Toggle rustowl highlights.


### Emacs

Elpaca example:

```elisp
(elpaca
  (rustowlsp
    :host github
    :repo "cordx56/rustowl"
    :files (:defaults "emacs/*")))
```

### RustRover / IntelliJ IDEs

There is a [third-party repository](https://github.com/siketyan/intellij-rustowl) that supports IntelliJ IDEs.


## Build manually

Here, we describe manual install instructions from source code.

### RustOwl

#### Prerequisite

- `rustup` and `cargo` installed
    - You can install `rustup` from [this link](https://rustup.rs/).
    - You need to set up the `PATH` environment variable. To do this, follow the instructions provided by the `rustup` installer. For example, in bash, run `export PATH=$HOME/.cargo/bin:$PATH`.

RustOwl has been tested on macOS Sequoia 15.3.2 on arm64 architecture with `rustup` 1.28.1.
We have not tested the installation of dependencies from other package repositories, such as Homebrew.
You may need to uninstall any Rust-related packages installed through those repositories first.
Other dependencies are locked in the configuration files and will be installed automatically.

We have also tested this on Ubuntu 25.04 on arm64 architecture.
Additional dependencies may be required.
We have confirmed that running `apt install build-essential` is necessary on a freshly installed Ubuntu for linking.

#### Build & Run

```bash
cd rustowl
cargo install --path . -F installer --locked
rustowl check
```


### VSCode extension

#### Prerequisite

- VSCode installed
    - You can install VSCode from [this link](https://code.visualstudio.com/).
- Node.js installed
- `yarn` installed
    - After installing Node.js, You can install `yarn` by running `npm install -g yarn`.

VSCode extension has been tested on macOS Sequoia 15.2 on arm64 architecture with Visual Studio Code 1.96.4, Node.js v20.16.0, and `yarn` 1.22.22.
Other dependencies are locked in the configuration files and will be installed automatically.

#### Build & Run

First, install the dependencies.

```bash
cd vscode
yarn install --frozen-lockfile
```

Then open `vscode` directory in VSCode.

A notification to install the recommended VSCode extension will appear in the bottom right corner of VSCode.
Click the install button, wait for the installation to finish, and then restart VSCode.

Open `vscode` directory again, and press the `F5` key in the VSCode window.
A new VSCode window with the extension enabled will appear.

Open cargo workspace directory in the new VSCode window.

When you save Rust files, decoration indicating the movement of ownership and lifetimes will appear in the editor.


## Note

In this tool, due to the limitations of VSCode's decoration specifications, characters with descenders, such as g or parentheses, may occasionally not display underlines properly.
Additionally, we observed that the `println!` macro sometimes produces extra output, though this does not affect usability in any significant way.
