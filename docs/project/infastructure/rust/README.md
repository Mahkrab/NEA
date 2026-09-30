# Why rust?

Rust is a powerful, type-safe, memory-safe, and extremely fast programming language. It is well-suited for building secure, and effecient systems, where speed, security, and reliability are important. Rust's ownership model and strong type system help prevent memory, type, and concurrency errors, making it a suitable choice for systems that need be dependable and secure. 
Rusts performance is comparable to C and C++, making it a rival in speed and reliability, with less bloat than C++, and modern feautures allow for more expressive and maintainable code, whilst still being low-level enough to write efficient, and complex code. 
Rust has a growing ecosystem of libraries and tools, making it effectient to build complex systems, and the community is active and supportive, providing resources and guidence for developers. 

# Building Vayu

I am starting with one Cargo application in `Vayu`, within the same Git repository as my documentation. I have pinned Rust to `1.96.0` so the compiler and checks stay consistent. Rustup reads [rust-toolchain.toml](/rust-toolchain.toml) and installs that version when needed.

From the project root, I can run the application and checks with:

```zsh
cd Vayu 
cargo run --locked # Build and run the Vayu project
cargo fmt --all -- --check # Checks if files are formatted correctly without updating
cargo clippy --all-targets --locked -- -D warnings # Runs the clippy linter on all targets
cargo test --locked #  Executes all unit, integration and documentation tests
cargo doc --no-deps --locked # Generates the project and dependancy docs
```

The [lockfile](/Vayu/Cargo.lock) records dependency versions, while generated files in `Vayu/target` are ignored from git.

# How rust is set up for this project

I am developing Vayu on Ubuntu `24.04.5 LTS`, running through WSL2 on Windows. This lets me use a Linux environment for the project, with `zsh 5.9` as my shell. The Rust compiler and Cargo are both version `1.98.1`, and the application uses Rust edition `2024` (the most recent). The edition sets the language rules used by the project, while the compiler version is the version of Rust used to build it.

I have kept the Rust application in `Vayu`, within the same git repository as the documentation, as this prevents having to deal with git submodules. This means the code and its evidence can be committed and tagged together, using the process above. [Vayu/src/main.rs](/Vayu/src/main.rs) is the starting point of the application. 

I use rustup to manage the Rust installation. To keep the compiler version consistent, I have added [rust-toolchain.toml](/rust-toolchain.toml) at the root of the repository:

```toml
[toolchain]
channel = "1.98.1"
profile = "minimal"
components = ["rustfmt", "clippy"]
```

Rustup reads this file when I use Rust commands within the project, and selects the version given. The minimal profile provides the compiler, Cargo and standard library, with `rustfmt` and `clippy` added for formatting and checking the code. This also gives me a record of the version needed when working on another machine.

I can check which version is being used from the project root:

```bash
cargo show # Displays crate metadata

Default host: x86_64-unknown-linux-gnu
rustup home:  /home/mahkrab/.rustup

installed toolchains
--------------------
stable-x86_64-unknown-linux-gnu (default)
1.96.0-x86_64-unknown-linux-gnu
1.98.1-x86_64-unknown-linux-gnu (active)

active toolchain
----------------
name: 1.98.1-x86_64-unknown-linux-gnu
active because: overridden by '/home/mahkrab/documents/programs/NEA/rust-toolchain.toml'
installed targets:
x86_64-unknown-linux-gnu

rustc --version # Shows the compiler version
cargo --version # Shows the Cargo version
```

Cargo manages building and running the application. [Vayu/Cargo.toml](/Vayu/Cargo.toml) records the package name, `vayu`, its version, `0.1.0`, and the Rust edition. It also records the Apache-2.0 licence, and disables publishing the package. 

I will keep both [Cargo.toml](/Vayu/Cargo.toml) and [Cargo.lock](/Vayu/Cargo.lock) in git. The lockfile records the dependency versions used by Cargo, so they can be reused when building the same version of the project. Generated files go in `Vayu/target`, which is ignored by git, as these can be rebuilt from the source.

From the project root, I used these commands to run the application and check the setup:

```zsh
cd Vayu
cargo run --locked # Builds and runs the application
cargo fmt --all -- --check # Checks the formatting without changing the files
cargo clippy --all-targets --locked -- -D warnings # Checks the code, treating warnings as errors
cargo test --locked # Runs the tests
cargo doc --no-deps --locked # Builds the documentation for the application
```