# Sccache Release Process

Most of the sccache release process is automated. The [github workflow](https://github.com/mozilla/sccache/actions?query=workflow%3Aci) contains builds for all supported platforms, as well as a release job that is triggered by pushing a new tag to the repository. That job will upload the resulting binary packages to [the GitHub releases page](https://github.com/mozilla/sccache/releases) on the repository.

# Producing a release

We use [`cargo-release`](https://crates.io/crates/cargo-release) to produce releases, since it encapsulates the steps of bumping the version number, creating and pushing a new tag, and releasing to [crates.io](https://crates.io/crates/sccache). You can install it with `cargo install cargo-release`, then simply run `cargo release` in an sccache checkout to do the work. Note that it supports a `--dry-run` option you can use to preview what it will run.

## Things to be aware of

1. You must have authenticated to crates.io using `cargo login` to publish the sccache create there.
2. cargo will not allow publishing a create if there are crates in the `[patch]` section in `Cargo.toml`.

# Fork releases

In `johnkferguson/sccache`, the upstream release job doesn't run. Pushing a tag named `v<version>-fork.<n>`, where `<version>` is the one in `Cargo.toml`, runs `.github/workflows/release.yml` instead. It publishes a GitHub release with `sccache-<tag>-x86_64-unknown-linux-musl.tar.gz`, a static binary with only the `s3` backend, and its `.sha256` file. The tag has to be on a commit that has `release.yml`.
