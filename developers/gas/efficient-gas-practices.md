# Efficient Gas Practices

Minimizing gas usage is essential to ensure cost-effectiveness and optimal performance on the blockchain. In this article we have collected our best practice for handling gas optimization.

The size of the contract state directly affects the CPU cost, particularly during serialization and deserialization processes. As the contract state grows larger, both serialization and deserialization require more computational resources, resulting in increased gas costs. It is important to be aware of this impact and optimize gas usage accordingly when creating smart contracts.

### Working with large states <a href="#working-with-large-states" id="working-with-large-states"></a>

When working with large states they can quickly grow to cost a lot of gas. Converting between data formats used by the blockchain and the formats used internally by the Rust contract can be slow, depending upon how the state is structured. To reduce the workload we should structure our states using formats that are easier to convert between. Notably, prefer using data-structures with fixed sizes (avoiding `Option` and `Vec`.) such that the blockchain can then understand and immediately serialize these by knowing the different lengths of the variables without looking at the actual data within. This makes it possible to serialize and deserialize with just one copy of the whole segment, thus only costing a fixed gas price.

In general it is recommended to always use a `Vec<>` with fix sized elements inside. If using a `struct` it's the same premise, when having a lot of fields, fix sized variables will save you the most amount of gas when used as part of either `Vec` `maps` or `struct`. We have created a [table of all fix sized elements](https://partisiablockchain.gitlab.io/documentation/smart-contracts/gas/table-of-fixed-size-elements.html) on PBC you can freely refer to as a guide when choosing what variable to use in your contracts.

A `Vec` containing thousands of addresses is more effective when taking the above points into account. You can dive into the technicalities of serialize and deserialize by visiting [our Rust docs here](https://partisiablockchain.gitlab.io/language/contract-sdk/pbc_traits/trait.ReadWriteState.html).

#### Example 1 of optimizing a `struct` <a href="#example-1-of-optimizing-a-struct" id="example-1-of-optimizing-a-struct"></a>

For a `struct` to be `serializable_by_copy`, i.e. the method described above, the sizes of the struct's fields must sum to the size of the `struct` in memory, you can read more about it in [the rust docs](https://doc.rust-lang.org/reference/type-layout.html).

As an example of the above:

```rust
pub struct Unaligned {
    a: u8,    // offset 0, size 1
    b: u16,   // offset 2, size 2,
    c: u8,    // offset 4, size 1
    // total size: 5
}
```

Here the `struct` is unaligned because a is only 1 byte in memory and b is placed on 2 in memory. This creates an empty space in memory and therefore makes the `struct` Unaligned. To align the struct you need to switch a and b to use the offset efficiently, this creates the correct ratio of the fields within the struct to the sum of the structs fields placement in memory. See updated version below:

```rust
pub struct Aligned {
    b: u16,  // offset 0, size 2
    a: u8,   // offset 2, size 1
    c: u8,   // offset 3, size 1
    // total size: 4
}
```

#### Example 2 of optimizing a `struct` <a href="#example-2-of-optimizing-a-struct" id="example-2-of-optimizing-a-struct"></a>

To give another example we have created a `struct` looking like this:

```rust
pub struct Unaligned {
    to: Address,            // offset 0, size 21
    from: Address,          // offset 21, size 21
    amount: u128,           // offset 42, size 16 (Offset after here is 58).
    timestamp: i64,         // offset 64, size 8
    // size: 66 bytes using 72 bytes of memory
}
```

The `struct` has two address an amount and a timestamp. The problem here is that the sum of the struct is not total the sum of fields in memory, there are 6 unused bytes between amount and timestamp. To solve this issue we need to do two things. First we would need to introduce: `#[repr(C)]` to manage the exact order of memory ensuring that `Rust` does not reorder or try to fix the issue itself. Secondly we needed to introduce padding of 6 bytes `padding: [u8; 6]` to make sure that the `struct` has the correct ratio. The efficient `struct` can be seen below:

```rust
pub struct Aligned {
    to: Address,            // offset 0, size 21
    from: Address,          // offset 21, size 21
    padding: [u8; 6],       // offset 42, size 6
    amount: u128,           // offset 48, size 16
    timestamp: i64,         // offset 64, size 8
    // size: 72 bytes
}
```

In conclusion, efficient gas practices involve minimizing gas usage when working with large data, optimizing CPU costs based on contract state size and using fix sized elements with the right ratios. By incorporating these practices, developers can create smart contracts that are more cost-effective and performant on the blockchain.
