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
