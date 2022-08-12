This repository is for to start the learning about Near Blockchain

For more help check: [Near SDK Doc](https://www.near-sdk.io/) and [SDk](https://docs.rs/near-sdk/latest/near_sdk/index.html) 

### 1. Install Rust and Wasm toolchain
    ```
    curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
    source $HOME/.cargo/env
    rustup target add wasm32-unknown-unknown
    ```

### 2. Create new project
    `npx create-near-app --contract=rust my-project`

    Install cargo-generate for work on existing projetc that can be on github
    ```
    cargo install cargo-generate --features vendored-openssl

    cargo generate --git https://github.com/near-examples/rust-status-message --name my-project
    cd my-project
    ```

    we can use `cargo new --lib <crate-name>` but `.toml` file should be like this:
    ```
    [lib]
    crate-type = ["cdylib"]

    [profile.release]
    codegen-units = 1
    # Tell `rustc` to optimize for small code size.
    opt-level = "z"
    lto = true
    debug = false
    panic = "abort"
    # Opt into extra safety checks on arithmetic operations https://stackoverflow.com/a/64136471/249801
    overflow-checks = true
    ```

### 3. Importents things
    ##### near_bindgen
        The `#[near_bindgen]` macro used on struct and struct `impl` to be a valid NEAR contract and expose the intended functions to be able to be called externally

    ##### #[init] and Default::default()
        By default, the `Default::default()` implementation of a contract will be used to initialize a contract.  But for custom initialization function which takes parameters or performs custom logic use the `#[init]` annotation.
        And use `PanicOnDefault` for prohibit the default implementation.

    ##### #[payable]
        Payable Methods can be annotated with `#[payable]` to allow tokens to be transferred with the method invocation.

    ##### #[private]
        Private Methods an be annotated with `#[private]` for the panic when this method is called externally.

    ##### Serialize and Deserialize
        `#[derive(BorshDeserialize, BorshSerialize, Default)]` or `#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]`
        use these macro from `use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};` for Serialize and Deserialize

    ##### near_sdk::collections
        The following data structures that exist in the [SDK](https://docs.rs/near-sdk/latest/near_sdk/collections/index.html)

        Data strustures in Rust std and in near sdk collection: 
        `Option<T>` = `LazyOption<T>`
        `Vec<T>` = `Vector<T>`
        `HashMap<K, V>` = `LookupMap<K, V>`
        `HashMap<K, V>` = `UnorderedMap<K, V>`
        `BTreeMap<K, V>` = `TreeMap<K, V>`
        `HashSet<T>` = `LookupSet<T>`
        `HashSet<T>` = `UnorderedSet<T>`

