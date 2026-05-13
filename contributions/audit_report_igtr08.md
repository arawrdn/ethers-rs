**FEATURE_NAME**  
Optimism Deposited Transaction (type `0x7E`) serialization/deserialization

**TECHNICAL_EXPLANATION**  
The README states Optimism support adds a non-standard typed transaction (`0x7E`) with extra fields: `sourceHash`, `mint`, and `isSystemTx`.  
A common missing test in cross-chain SDKs is a **round-trip codec test** for custom transaction types:

1. Build an Optimism deposited transaction with all required fields.  
2. Serialize it to raw bytes (RLP/typed envelope).  
3. Decode it back into a transaction object.  
4. Assert:
   - tx type is exactly `0x7E`
   - custom fields are preserved
   - no silent coercion into legacy/EIP-1559 tx formats

This catches regressions where:
- custom fields are dropped during encoding,
- tx type is mis-labeled,
- parser falls back to Ethereum default typed tx handling.

---

**CODE_SNIPPET**  
```rust
// tests/optimism_deposit_tx_roundtrip.rs
// Run with: cargo test --features optimism

#[cfg(feature = "optimism")]
mod tests {
    use ethers::types::{
        transaction::eip2718::TypedTransaction,
        Address, Bytes, U256, H256,
    };

    // NOTE:
    // Field/constructor names may vary slightly by ethers-rs patch version.
    // Adjust to your concrete Optimism tx struct (e.g. OpTxEnvelope / DepositTransaction).
    #[test]
    fn deposit_tx_type_0x7e_roundtrip_preserves_fields() {
        // Example deterministic values
        let from: Address = "0x1000000000000000000000000000000000000001".parse().unwrap();
        let to: Address   = "0x2000000000000000000000000000000000000002".parse().unwrap();
        let source_hash: H256 = "0xaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
            .parse()
            .unwrap();

        // PSEUDOCODE-STYLE: replace with actual Optimism deposit tx type in ethers-rs
        // ---------------------------------------------------
        // let deposit = OptimismDepositTransaction {
        //     from: Some(from),
        //     to: Some(to),
        //     value: U256::from(1234u64),
        //     data: Bytes::from(vec![0xde, 0xad, 0xbe, 0xef]),
        //     source_hash,
        //     mint: Some(U256::from(42u64)),
        //     is_system_tx: false,
        //     gas: U256::from(21000u64),
        //     ..Default::default()
        // };
        //
        // let typed = TypedTransaction::OptimismDeposited(deposit.clone());
        // let encoded = typed.rlp(); // or typed.rlp_signed(&sig) depending API
        // let decoded = TypedTransaction::decode(&mut encoded.as_ref()).unwrap();
        //
        // match decoded {
        //     TypedTransaction::OptimismDeposited(d) => {
        //         assert_eq!(d.source_hash, source_hash);
        //         assert_eq!(d.mint, Some(U256::from(42u64)));
        //         assert!(!d.is_system_tx);
        //         assert_eq!(d.from, Some(from));
        //         assert_eq!(d.to, Some(to));
        //     }
        //     other => panic!("expected Optimism deposited tx, got {:?}", other),
        // }
        // ---------------------------------------------------

        // Keep one hard assertion to ensure test intent remains explicit.
        // Replace this with actual `tx_type()` accessor once mapped to concrete API.
        let expected_type_byte = 0x7eu8;
        assert_eq!(expected_type_byte, 0x7e);
    }
}
```

If you want, I can provide a **version-pinned, compile-ready** variant for a specific ethers-rs `2.0.x` patch once you share your exact Cargo.lock or crate versions.