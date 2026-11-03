sccache includes support for caching Rust compilation. This includes many caveats, and is primarily focused on caching rustc invocations as produced by cargo. A (possibly-incomplete) list follows:
* `--emit` is required.
* `--crate-name` is required.
* Only `link`, `metadata` and `dep-info` are supported as `--emit` values, and `link` must be present.
* `--out-dir` is required.
* `-o file` is not supported.
* Compilation from stdin is not supported, a source file must be provided.
* Values from `env!` require Rust >= 1.46 to be tracked in caching.
* Procedural macros that read files from the filesystem may not be cached properly.
* `rustc`'s incremental compilation needs to be disabled. See [The Cargo Book](https://doc.rust-lang.org/cargo/reference/profiles.html#incremental)
* Crates that invoke the system linker cannot be cached. Examples are `bin`, `dylib`, `cdylib`, and `proc-macro` crates. Setting `SCCACHE_RUST_CRATE_TYPE_ALLOW_HASH` in the build environment (the one cargo runs sccache in) caches them anyway, except under `cargo check`, and never compiles them on a distributed build server. sccache doesn't know the linker's inputs, so its value goes into the cache key of these crates: give machines with different linkers or system libraries different values. Known gaps:
  * Only the crate's direct dependencies are hashed. Native code bundled into a transitive dependency's rlib can change without changing the key, for example after changing `CC` or `CFLAGS`, and the old binary comes back. Clear the cache or change the variable's value after such a change.
  * Only the main artifact is cached. Side files the link produces are not: `.pdb` files and import libraries on Windows (dependents of a cached `dylib` fail to link), `.dSYM` and unpacked object files on macOS, `.dwo`/`.dwp` files with `split-debuginfo`. A hit can lose debug info.

If you are using Rust 1.18 or later, you can ask cargo to wrap all compilation with sccache by setting `RUSTC_WRAPPER=sccache` in your build environment.
