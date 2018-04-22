# ptags
A parallel [universal-ctags](https://ctags.io) wrapper for git repository

[![Build Status](https://travis-ci.org/dalance/ptags.svg?branch=master)](https://travis-ci.org/dalance/ptags)
[![Build status](https://ci.appveyor.com/api/projects/status/9a5jhfnxnmf7x0xo?svg=true)](https://ci.appveyor.com/project/dalance/ptags)
[![Crates.io](https://img.shields.io/crates/v/ptags.svg)](https://crates.io/crates/ptags)
[![codecov](https://codecov.io/gh/dalance/ptags/branch/master/graph/badge.svg)](https://codecov.io/gh/dalance/ptags)

## Description

**ptags** is a [universal-ctags](https://ctags.io) wrapper to have the following features.
- Search git tracked files only ( `.gitignore` support )
- Call `ctags` command in parallel for acceleration
    - Up to x5 faster than universal-ctags

## Install
Download from [release page](https://github.com/dalance/ptags/releases/latest), and extract to the directory in PATH.

Alternatively you can install by [cargo](https://crates.io).

```
cargo install ptags
```

## Requirement

**ptags** uses `ctags` and `git` command internally.
The tested version is below.

| Command   | Version                                               |
| --------- | ----------------------------------------------------- |
| `ctags`   | Universal Ctags 0.0.0(f9e6e3c1) / Exuberant Ctags 5.8 |
| `git`     | git version 2.14.2                                    |
| `git-lfs` | git-lfs/2.3.3                                         |

## Usage

```
ptags 0.1.12-pre
dalance@gmail.com
A parallel universal-ctags wrapper for git repository

USAGE:
    ptags [FLAGS] [OPTIONS] [--] [DIR]

FLAGS:
        --config               Generate configuration sample file
        --exclude-lfs          Exclude git-lfs tracked files
    -h, --help                 Prints help information
        --include-ignored      Include ignored files
        --include-submodule    Include submodule files
        --include-untracked    Include untracked files
    -s, --stat                 Show statistics
        --unsorted             Disable tags sort
        --validate-utf8        Validate UTF8 sequence of tag file
    -V, --version              Prints version information
    -v, --verbose              Verbose mode

OPTIONS:
        --bin-ctags <bin_ctags>           Path to ctags binary [default: ctags]
        --bin-git <bin_git>               Path to git binary [default: git]
        --completion <completion>         Generate shell completion file [possible values: bash, fish,
                                          zsh, powershell]
    -e, --exclude <exclude>...            Glob pattern of exclude file ( ex. --exclude '*.rs' )
    -c, --opt-ctags <opt_ctags>...        Options passed to ctags
    -g, --opt-git <opt_git>...            Options passed to git
        --opt-git-lfs <opt_git_lfs>...    Options passed to git-lfs
    -f, --file <output>                   Output filename ( filename '-' means output to stdout ) [default: tags]
    -t, --thread <thread>                 Number of threads [default: 8]

ARGS:
    <DIR>    Search directory [default: .]
```

You can pass options to `ctags` by`-c`/`--ctags_opt` option like below.

```
ptags -c --links=no -c --languages=Rust
```

Searched file types per options are below.
`--include-submodule` and `--include_untracked` are exclusive.
This is the restriction of `git ls-files`.
Any include/exclude options without the above combination can be used simultaneously.

| File type     | Default  | --exclude-lfs | --include-ignored | --include-submodule | --include-untracked |
| ------------- | -------- | ------------- | ----------------- | ------------------- | ------------------- |
| tracked       | o        | o             | o                 | o                   | o                   |
| untracked     | x        | x             | x                 | x                   | o                   |
| ignored       | x        | x             | o                 | x                   | x                   |
| lfs tracked   | o        | x             | o                 | o                   | o                   |
| in submodules | x        | x             | x                 | o                   | x                   |

You can override any default option by `~/.ptags.toml` like below.
The complete example of `~/.ptags.toml` can be generated by `--config` option.

```toml
thread = 16
bin_ctags = "ctags2"
bin_git = "git2"
```

## Benchmark

### Environment
- CPU: Ryzen Threadripper 1950X
- MEM: 128GB
- OS : CentOS 7.4.1708

### Data

| Name    | Repository                           | Revision     | Files  | Size[GB] |
| ------- | ------------------------------------ | ------------ | ------ | -------- |
| source0 | https://github.com/neovim/neovim     | f5b0f5e17    | 2370   | 0.1      |
| source1 | https://github.com/llvm-mirror/llvm  | ddf9edb4020  | 29670  | 1.2      |
| source2 | https://github.com/torvalds/linux    | 071e31e254e0 | 52998  | 2.2      |
| source3 | https://github.com/chromium/chromium | d79c68510b7e | 293205 | 13       |

### Result

**ptags** is up to x5 faster than universal-ctags.

| Command       | Version                         | source0         | source1         | source2          | source3         |
| ------------- | ------------------------------- | --------------- | --------------- | ---------------- | --------------- |
| `ctags -R`    | Universal Ctags 0.0.0(f9e6e3c1) | 0.41s ( x1 )    | 3.42s ( x1 )    | 23.64s ( x1 )    | 32.23 ( x1 )    |
| `ptags -t 16` | ptags 0.1.4                     | 0.13s ( x3.15 ) | 0.58s ( x5.90 ) | 4.24s  ( x5.58 ) | 7.27s ( x4.43 ) |

