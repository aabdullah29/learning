This repository is for to start the learning about Near Blockchain

For more help check: [Near SDK Doc](https://www.near-sdk.io/) and [SDk](https://docs.rs/near-sdk/latest/near_sdk/index.html) 

## Install Rust and Wasm toolchain
```
curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup target add wasm32-unknown-unknown
```

## Create new project
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

## Importents things
- #### near_bindgen
    The `#[near_bindgen]` macro used on struct and struct `impl` to be a valid NEAR contract and expose the intended functions to be able to be called externally

- #### #[init] and Default::default()
    By default, the `Default::default()` implementation of a contract will be used to initialize a contract.  But for custom initialization function which takes parameters or performs custom logic use the `#[init]` annotation.
    And use `PanicOnDefault` for prohibit the default implementation.

- #### #[payable]
    Payable Methods can be annotated with `#[payable]` to allow tokens to be transferred with the method invocation.

- #### #[private]
    Private Methods an be annotated with `#[private]` for the panic when this method is called externally.

- #### Serialize and Deserialize
    `#[derive(BorshDeserialize, BorshSerialize, Default)]` or `#[derive(BorshDeserialize, BorshSerialize, PanicOnDefault)]`
    use these macro from `use near_sdk::borsh::{self, BorshDeserialize, BorshSerialize};` for Serialize and Deserialize

- #### #[serde(crate = "near_sdk::serde")]


- #### #[derive(BorshStorageKey, BorshSerialize)]



- #### near_sdk::collections
    The following data structures that exist in the [SDK](https://docs.rs/near-sdk/latest/near_sdk/collections/index.html)

    Data strustures in Rust std and in near sdk collection: 
    `Option<T>` = `LazyOption<T>`
    `Vec<T>` = `Vector<T>`
    `HashMap<K, V>` = `LookupMap<K, V>`
    `HashMap<K, V>` = `UnorderedMap<K, V>`
    `BTreeMap<K, V>` = `TreeMap<K, V>`
    `HashSet<T>` = `LookupSet<T>`
    `HashSet<T>` = `UnorderedSet<T>`

    **persistent collections:**
    UnorderedMap, UnorderedSet, TreeMap, Vector, LazyOption
    When use the persistent collection, the prefix must be constructed like:
    `UnorderedMap::new(b"a"), LookupMap::new(b"t"), LazyOption::new(b"m")`

- #### Contract Interface
  - ##### Public Method and trait implementations
        Methods can be called externally by using the pub identifier and you can mark the function as public but add the #[private] annotation so that it will panic if called from anything but the contract itself.
        ```
        #[near_bindgen]
        impl MyContractStructure {
            pub fn some_method(&mut self) {
                // .. method logic here
            }
        }
        ```

        Functions can also be exposed through trait implementations. `#[near_bindgen]` macro only needs to be attached to the trait implementation, not the trait itself:
        '''
        pub trait MyTrait {
            fn trait_method(&mut self);
        }

        #[near_bindgen]
        impl MyTrait for MyContractStructure {
            fn trait_method(&mut self) {
                // .. method logic here
            }
        }
        ```

  - ##### Contract Mutability
        Contract state mutability is handled automatically based on how `self` is used in the function parameters. The `#[near_bindgen]` macro will generate the respective code to load/deserialize state for any function which uses `self` and serialize/store state only for when `&mut self` is used.

        Read-Only Functions: use `self` or `self` as a parameter
        self (owned value): shift ownership and can avoid by using `Clone/Copy`
        &self (immutable reference): the contract state is only read and value can re-used
        Returning derived data: retuen some data like: `fn update_stats(&self, account_id: AccountId, score: U64) -> Account `
        Mutable Functions: load the existing state, modify, then rewriting state `fn modify_value(&mut self, new_value: u64) {self.integer = new_value;}`
        Pure Functions: functions do not use self at all `fn log_u64(value: u64) {near_sdk::log!("{}", value);}` and `fn return_static_u64() -> u64 {SOME_VALUE}`

  - ##### Private Methods
        **Using callbacks:**
        Contract has to have a callback for a remote cross-contract call, and we can avoid it using `#[private]` this callback method should only be called by the contract itself. Use `#[private]` annotation within the `near_bindgen` macro:

        **internal methods:**
        There are three approaches to write internal methods that can not access outside
        1. Using `fn` instead of `pub fn`
        2. Using `pub(crate) fn`.
        3. Separate impl block
        ```
        #[near_bindgen]
        impl Contract {
            pub fn increment(&mut self) {
                self.internal_increment();
            }
        }
        impl Contract {
            /// This methods is still not exported.
            pub fn internal_increment(&mut self) {
                self.counter += 1;
            }
        }
        ```

  - ##### Payable Methods
        To declare a method as payable, use the `#[payable]` annotation within the `near_bindgen` macro. By default the methods are not payable and they will panic if someone will attempt to transfer tokens.

  - ##### Serialization Protocols
        Near use default JSON and Borsh for serialization. In general, JSON will be used for contract calls and cross-contract calls, where Borsh can be used to optimize using less gas by having smaller parameter serialization and less deserialization computation within the contract.

        - ##### Overriding Serialization Protocol Default: 
            ```
            #[derive(BorshDeserialize, BorshSerialize)]
            pub struct SetMessageInput {
                // Note that the key does not have to be "message" like the argument name.
                msg: String,
            }
            ```

        - ##### JSON wrapper types: 
            To help with serializing certain types to JSON which have unexpected or inefficient default formats, there are some wrapper types in `near_sdk::json_types` that can be used.

        - ##### Base64VecU8:
            Another example of a type you may want to override the default serialization of is `Vec<u8>` which represents bytes in Rust. By default, this will serialize as an array of integers, which is not compact and very hard to use. There is a wrapper type `Base64VecU8` which serializes and deserializes to a `Base-64` string for more compact JSON serialization.


- #### cross-contract
    - ##### Callbacks:
        When interacting with a native Rust (compiled to Wasm) smart contract, cross-contract calls are asynchronous. Callbacks are used to either get the result of a cross-contract call or tell if a cross-contract call has succeeded or failed. There are two techniques to write cross-contract calls: high-level and low-level. 
        `#[ext_contract(trait_name)]` example:
        ```
        #[ext_contract(ext_calculator)]
        trait Calculator {
            fn mult(&self, a: U64, b: U64) -> U128;
        }
        ```
        Let's assume the calculator is deployed on `calc.near`, we can use it `ext_calculator::ext("calc.near".parse().unwrap()).sum(2, 3)`
        ```
        #[near_bindgen]
        impl Contract {
            pub fn sum_a_b(&mut self, a: U128, b: U128) -> Promise {
                let calculator_account_id: AccountId = "calc.near".parse().unwrap();
                // Call the method `sum` on the calculator contract.
                // Any unused GAS will be attached since the default GAS weight is 1. Attached deposit is defaulted to 0.
                ext_calculator::ext(calculator_account_id)
                    .sum(a, b)
            }
        }
        ```

    - ##### .then:
        The common pattern with cross-contract calls is to call a method on an external smart contract, use .then syntax to specify a callback, and then retrieve the result or status of the promise. The callback will typically live inside the same, calling smart contract. 
        We'll show two simple functions that will make a cross-contract call to an allowlist smart contract, asking if the account `assount-deployed-name` is allowlisted. These methods will both return true using different approaches 
        1.`#[callback_unwrap]` we can use this approach by `callback_promise_result()` method:
        ```
        #[private]
        pub fn callback_arg_macro(#[callback_unwrap] val: bool) -> bool {
            val
        }
        ```

        2.`#[callback_result]` we can use this approach by `callback_arg_macro()` method:
        ```
        #[private]
        pub fn callback_promise_result() -> bool {
            assert_eq!(env::promise_results_count(), 1, "ERR_TOO_MANY_RESULTS");
            match env::promise_result(0) {
                PromiseResult::NotReady => unreachable!(),
                PromiseResult::Successful(val) => {
                    if let Ok(is_allowlisted) = near_sdk::serde_json::from_slice::<bool>(&val) {
                        is_allowlisted
                    } else {
                        env::panic_str("ERR_WRONG_VAL_RECEIVED")
                    }
                },
                PromiseResult::Failed => env::panic_str("ERR_CALL_FAILED"),
            }
        }
        ```

        The first method uses a macro on the argument to cast the value into what's desired. In this approach, if the value is unable to be casted, it will panic. If you'd like to gracefully handle the error, you can either use the first approach, or use the `#[callback_result]` macro instead. An example of this can be seen below:
        ```
        #[private]
        pub fn handle_callbacks(
            // New pattern, will gracefully handle failed callback results
            #[callback_result] b: Result<u8, near_sdk::PromiseError>,
        ) {
            if b.is_err() {
                // ...
            }
        }
        ```

- #### Promises
    Transactions can be sent asynchronously from a contract through a [`Promise`](https://docs.rs/near-sdk/latest/near_sdk/struct.Promise.html). This will cause code to be executed in the `future`. Few situations where Promises are uniquely capable:

    1. ##### Sending $NEAR
        Transferring NEAR tokens is the simplest transaction you can send from a smart contract using `Promise`.
        ```
        let amount: u128 = 1_000_000_000_000_000_000_000_000; // 1 $NEAR as yoctoNEAR
        let account_id: AccountId = "example.near".parse().unwrap();
        Promise::new(account_id).transfer(amount);
        ```

    2. ##### Creating accounts
        You might want to create an account from a contract for many reasons that can be done by using `Promise` and an account with no balance is almost unusable, you probably want to combine this with the token transfer.
        ```
        Promise::new("subaccount.example.near".parse().unwrap())
        .create_account()
        .add_full_access_key(env::signer_account_pk())
        .transfer(250_000_000_000_000_000_000_000); // 2.5e23yN, 0.25N
        ```

    3. ##### Deploying contracts
        You might want your smart contract to deploy subsequent smart contract code.
        ```
        const CODE: &[u8] = include_bytes!("./path/to/compiled.wasm");
        Promise::new("subaccount.example.near".parse().unwrap())
            .create_account()
            .add_full_access_key(env::signer_account_pk())
            .transfer(3_000_000_000_000_000_000_000_000) // 3e24yN, 3N
            .deploy_contract(CODE.to_vec())
        ```

- #### Testing
    Testing contract functionality can be done through the cargo test framework. These tests will run with a mocked blockchain and will allow testing function calls directly without having to set up/deploy to a network and sign serialized transactions on this network.
    
    ##### Unit Tests
    Use `VMContextBuilder` to allows for modifying the context of the mocked blockchain.
    The `testing_env!` macro will initialize the blockchain interface with the `VMContext` which is either initialized through `VMContextBuilder` or manually through itself.
    To test `read-only` function calls, set `is_view` to `true` on the `VMContext`. 

    ##### Integration Tests
    You'll probably want to use integration tests when:

    - There are cross-contract calls.
    - There are multiple users with balance changes.
    - You'd like to gather information about gas usage and execution outcomes on-chain.
    - You want to assert the use-case execution flow of your smart contract logic works as expected.
    - You want to assert given execution patterns do not work (as expected).  

    **Setup:** 
    Unlike unit tests (which would often live in the `src/lib.rs` file of the contract), integration tests in Rust are located in a separate directory at the same level as `/src`, called `/tests`.
    Find more detail and examples [here](https://www.near-sdk.io/testing/integration-tests).

    ##### Simulation Testing to Workspaces
    We use asynchronous call because in simulation function calls return values that implement Future trait. We can use [tokio](https://tokio.rs/), [async-std](https://async.rs/) and [smol](https://github.com/smol-rs/smol) for asynchronous and use [sandbox](https://github.com/near/sandbox) for local blockchain which we will use in testing.
    Find more detail and examples [here](https://www.near-sdk.io/testing/workspaces-migration-guide).


- #### Building Contracts
    **Build Smart Contract in wasm:** `cargo build --target wasm32-unknown-unknown --release`
    Mention the root of project in `cargo.toml` if we build multiple contract in one project
    and we can set custom flas's in `cargo,toml` like:
    ```
    [target.wasm32-unknown-unknown]
    rustflags = ["-C", "link-arg=-s"]
    ```

- #### Upgrading Contracts
    When you do some change in exixting contract then you can re-deploy it, there are some different situation for re-deploy the contract for detail you can visit the [link](https://www.near-sdk.io/upgrading/prototyping).

- #### Reducing Contract Size
    We can reduse the `wasm` file size with the help of following tips:
    use `env::panic(b"ERR_NOT_OWNER")` instead of `assert_eq!(a,b,"message")`
    use `option.unwrap_or_else(|| env::panic_str("Token not found"))` instead of  `Option.expect("Token not found")`
    use `env::panic_str("ERR_MSG_HERE")` instead of `panic!("ERR_MSG_HERE")`

- #### Best Practices
    use `require!` early in any function: `require!(env::predecessor_account_id() == self.owner_id, "Owner's method")`
    use `log!` for debug: `log!("Transferred {} tokens from {} to {}", amount, sender_id, receiver_id)` or
    `env::log_str(format!("Transferred {} tokens from {} to {}", amount, sender_id, receiver_id).as_ref())`
    use `Promise` if your method makes a cross-contract call: `pub fn withdraw_100(&mut self, receiver_id: AccountId) -> Promise {...}`
    use again and again `borsh`, `base64`, `bs58`, `serde`, `serde_json` from `near-sdk`

