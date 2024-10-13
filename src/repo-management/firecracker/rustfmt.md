# How Firecracker Format Their Rust Code

Like most Rust projects,
[Firecracker](https://github.com/firecracker-microvm/firecracker/) uses
`cargo fmt` (namely `rustfmt`) to format Rust code.

This rule is applied bu executing the `pre-commit` shell script, and enforced in
their CI.

## pre-commit Script

This script lints every staged file, as of `.rs`s, it apply `cargo fmt` on them.
The command directly formats that file instead of produce differences
before/after format.

## Install rustfmt

`rustfmt` (invoked by `cargo fmt` command) could be installed with:

```sh
rustup component add rustfmt
```

And we can then use it with command:

```sh
cargo fmt
```

### Nightly Toolchain

Sometimes we need unstable options of `rustfmt`, e.g. `group_imports` and the
like. These options are not available in stable channel, thus we need to install
nightly channel:

```sh
rustup component add rustfmt --toolchain nightly
```

Then we can use nightly `rustfmt` with:

```sh
cargo +nightly fmt
```
