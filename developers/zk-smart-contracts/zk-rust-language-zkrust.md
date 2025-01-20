# ZK Rust Language (ZKRust)

ZK-Rust is the main tool for writing Zero-Knowledge Computations for Partisia Blockchain. ZK-Rust resembles [Rust](https://rust-lang.com) in syntax, with a restricted feature set, and an alternative standard library tailored for Partisia Blockchain.

Further reading:

* [Zero-knowledge Language Reference](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/zk-rust-reference.html)
* Zero-knowledge Contract FAQ
* [Zero-knowledge Language Feature Checklist](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/zk-language-features.html)

### Concepts <a href="#concepts" id="concepts"></a>

Zk-Rust's most significant difference from Rust, is it's support for representing [Secret-shared values](https://en.wikipedia.org/wiki/Secret_sharing), the basic building blocks for writing Zero-knowledge computations. This secret-sharing support extends from new syntax to types to runtime behavior.

#### Secret-shared integers <a href="#secret-shared-integers" id="secret-shared-integers"></a>

The secret-sharing supported in Zk-Rust is an alternative method of representing integers. This representation is used when working with the [`Sbi`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_zk_core/struct.Sbi.html) types (`Sbi1`, `Sbi8`, `Sbi16`, etc.,) which stands for "Secret-shared Binary Integer". These types are fully fledged integer types, supporting a variety of infix operations (`+`, `-`, `^`, `|`, etc.) All of these operations are performed through secret-sharing, ensuring that the result is itself secret-shared:

```rust
use pbc_zk::Sbi32;

pub fn my_computation() -> Sbi32 {
    let public_1: i32 = 93
    let secret_1: Sbi32 = Sbi32::from(public_1);
    let secret_2: Sbi32 = Sbi32::from(231);
    let secret_3: Sbi32 = secret_1 + secret_2;
    secret_3 * Sbi32::from(2)
}
```

As seen above, secrecy is _contagious_: All operations involving a secret-shared type produces a new secret-shared type.

This comes from the more general rule that _it must be impossible to learn any secret information at the public level_, a kind of rule known as [Information Flow Control](https://en.wikipedia.org/wiki/Information_flow_\(information_theory\)) (IFC). The secret-sharing IFC rule is enforced by The Partisia Blockchain, which prevents leakage of sensitive information.

The following program will not compile, for example:

```rust
use pbc_zk::Sbi32;

pub fn my_computation() -> i32 {
    let secret_1: Sbi32 = Sbi32::from(93);
    let secret_2: Sbi32 = Sbi32::from(231);
    let declassify: i32 = secret_1 + secret_2; // !! cannot assign secret to public variable!
    declassify * 2
}
```

New `SbiN` values can be created from public integers using `SbiN::from(iN)`.

#### Secret Branching <a href="#secret-branching" id="secret-branching"></a>

As we've seen above, the operations available on `Sbi` is constrained, due to IFS. Another situation where Secret-shared values are less flexible than their public counterparts is when branching.

Consider for example:

```rust
use pbc_zk::Sbi32;

pub fn my_computation() -> i32 {
    let secret: Sbi32 = ...;
    let mut public: i32 = 9;
    if secret == Sbi32::from(4) {
        public = 5;  // !! Cannot assign secret to public variable!
    }
    public
}
```

The above program is disallowed by ZkRust, because it is trying to assign to a public value while branching upon a secret. This is clearly a break of IFC, as the value of `public` after the branch would be based upon the value of `secret`, which again is a break of IFC.

It is possible to assign secrets in secret-branches:

```rust
use pbc_zk::Sbi32;

pub fn my_computation() -> Sbi32 {
    let secret: Sbi32 = ...;
    let mut secret_2: Sbi32 = Sbi32::from(9);
    if secret == Sbi32::from(4) {
        secret_2 = Sbi32::from(5);
    }
    secret_2
}
```

Here the assignment is allowed, as there is no declassification. Note though that this branch is compiled differently from public branches, and might result in some performance overhead.

#### Secret variables <a href="#secret-variables" id="secret-variables"></a>

Each contract possess a list of secret variables. These variables possess:

* A unique id.
* Secret data, secret-shared between contract's computation nodes. Loadable in the computation by `load_sbi::<S: SecretBinary>(id: i32): S`.
* Public information called **metadata**. Loadable in the computation by `load_metadata::<T: Public>(id: i32): T`.
* An owner (blockchain account), who uploaded the variable, or in the case of computation outputs, the contract itself. This information is not available for zk-computation.

Contract users can add new variables (called inputs at this point) by invoking the contract's [`#[zk_on_secret_input]`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_contract_codegen/attr.zk_on_secret_input.html). Variables can additionally be created as the result of running a computation. The public part of the ZK-contract is responsible for managing the variables, including opening (revealing the output of a computation) and deleting them.

Variable ids can be iterated by calling [`pbc_zk::secret_variable_ids(): Iter<i32>`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_zk/fn.secret_variable_ids.html):

```rust
use pbc_zk::{Sbi32, secret_variable_ids, load_sbi};

pub fn sum_all_variables() -> Sbi32 {
    let mut sum = Sbi32::from(0);
    for variable_id in secret_variable_ids() {
        sum = sum + load_sbi::<Sbi32>(variable_id);
    }
    sum
}
```

#### Type Structures <a href="#type-structures" id="type-structures"></a>

Rust's `struct` keyword is supported for creating structure types. These structs can either contain purely public types, or purely secret-shared types.

```rust
use pbc_zk::{Sbi16, SecretBinary};

// public
struct SomeData {
    uid: i64,
    info: i32,
}

#[derive(SecretBinary)]
struct Point {
    x: Sbi16,
    y: Sbi16,
}

#[derive(SecretBinary)]
struct Triangle {
    x: Point,
    y: Point,
    z: Point,
}
```

Types with `#[derive(SecretBinary)]` implements the [`SecretBinary`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_zk/trait.SecretBinary.html) trait, allowing them to be loaded using the [`load_sbi`](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_zk/fn.load_sbi.html) function. Note that variables does not possess explicit types, and can be loaded as any type of equal or fewer bits. For example, let's say variable with id `42` has 64 bits of data. This variable can be loaded using any of the following, producing different values for each:

* `load_sbi::<Sbi32>(42)`, due to `32 <= 64`.
* `load_sbi::<Sbi64>(42)`, due to `64 <= 64`.
* `load_sbi::<Point>(42)`, due to `16 + 16 <= 64`.

Attempting to load variable `42` as `Triangle` would result in a runtime exception, as it would attempt to load `(16 + 16)*3 = 96` bits, which variable `42` cannot provide.

#### Libraries <a href="#libraries" id="libraries"></a>

As shown in the previous examples, it is possible to import functions and data structures from the `pbc_zk` module. See [Zero-knowledge Language Reference](https://partisiablockchain.gitlab.io/documentation/smart-contracts/zk-smart-contracts/zk-rust-reference.html) for what is available.
