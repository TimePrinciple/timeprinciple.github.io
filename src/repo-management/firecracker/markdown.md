# How Firecracker Manager Their Markdown Files

[Firecracker](https://github.com/firecracker-microvm/firecracker/) use
`mdformat` (a python written program) to lint their files end with `.md`.

This rule is applied by executing the `pre-commit` shell script, and enforced in
their CI.

## pre-commit Script

This script lints every staged file, as of `.md`s, it simply do `mdformat
<file-name>`. The command directly formats that file instead of produce
differences before/after format.

It is a good idea to format these files, but I don't prefer doing it without
`--check` flag. I think it would be better to tell developers that some `.md`
files are not in a preferred form that `mdformat` thinks, instead of changing it
directly and developers may well get confuse on what is going to be committed.
