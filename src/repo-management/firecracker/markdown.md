# How Firecracker Manager Their Markdown Files

[Firecracker](https://github.com/firecracker-microvm/firecracker/) use
`mdformat` (a python written program) to lint their files end with `.md`.

This rule is applied by executing the `pre-commit` shell script, and enforced in
their CI.

## pre-commit Script

This script lints every staged file, as of `.md`s, it simply do
`mdformat <file-name>`. The command directly formats that file instead of
produce differences before/after format.

It is a good idea to format these files, but I don't prefer doing it without
`--check` flag. I think it would be better to tell developers that some `.md`
files are not in a preferred form that `mdformat` thinks, instead of changing it
directly and developers may well get confuse on what is going to be committed.

## Install mdformat

Installation guide of `mdformat` is documented
[here](https://github.com/executablebooks/mdformat/blob/master/README.md#installing).
GibHub's Markdown renderer supports syntax enxtensions not included in the GFM
specification, so for full GitHub support we should install it with:

```sh
pip install mdformat-gfm mdformat-frontmatter mdformat-footnote mdformat-gfm-alerts
```

## Configuration in Repository

`mdformat` allows configuration specified in `.mdformat.toml` file. Firecracker
community uses the following configuration:

```toml
wrap = 80
# Defaults from https://mdformat.readthedocs.io/en/stable/users/configuration_file.html
number = false
end_of_line = "lf"
```

As of 80 columns wrapping, this is the default behavior I observed of `gq`
command of Vim.
