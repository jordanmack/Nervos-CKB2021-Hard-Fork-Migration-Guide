# Updating PW-SDK for Compatibility With CKB2021

PW-Core has been updated for compatibility with the CKB2021 hard fork. Most notably, PW-Core now supports the CKB2021 address format and Omni Lock. It is highly recommended that developers make all the changes noted below immediately to prevent fragmentation in the ecosystem.

## Impact After Updating

After updating PW-Core, your dapp will begin using Omni Lock and CKB2021 addresses automatically. This means that users may see a new address associated with their account when they visit your site, and their previous assets may be missing. This is because their assets are likely located on the older PW-Lock using the Pre2021 addresses. 

The private keys of your users remain unchanges, but multiple different addresses are now controlled by those private keys. In order to recover and migrate assets, you will need to understand how to these addresses are used in PW-SDK.

## CKB2021 Address Format

PW-Core will read addresses in both CKB2021 (new format) and Pre2021 (old format). No changes need to be made in order to read CKB2021 addresses.

```typescript
// Example of creating an Address instance with a CKB2021 address.

const address = new Address('ckt1qzda0cr08m85hc8jlnfp3zer7xulejywt49kt2rr0vthywaa50xwsqdr2z2v7uxcmthzysxqejkanv72w2rx6uq8s7h4s', AddressType.ckb);
```

When outputting a CKB address using `toCKBAddress()`, a parameter can be passed to specify the version. By default, the newest address format will always be used. Currently, the newest version is CKB2021. 

```typescript
// Example of outputting a CKB address in different versions.

// Create an Address instance with a ETH address.
const address = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth);

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(address.toCKBAddress());

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(address.toCKBAddress(NervosAddressVersion.latest));

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(address.toCKBAddress(NervosAddressVersion.ckb2021));

// ckt1q3uljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqt64l6ed309u7ywl7sta9rqzjmefnwc65gqr9hrgd (Omni Lock; ETH; Pre2021)
console.log(address.toCKBAddress(NervosAddressVersion.pre2021));
```

## Omni Lock / PW-Lock

PW-Core will read addresses using both Omni Lock (new) and PW-Lock (old). No changes are needed to read Omni Lock CKB addresses. 

```typescript
// Example of creating Address instances with Omni Lock and PW-Lock addresses.

const omniLockAddress = new Address('ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp66fgyfnljgqhs4wjvfrq6qfy3k5huljpqqf3hfdz', AddressType.ckb);
const pwLockAddress = new Address('ckt1qpvvtay34wndv9nckl8hah6fzzcltcqwcrx79apwp2a5lkd07fdxxqwkj2pzvlujq9u9t5nzgcxszfyd49l8usghcwxk4', AddressType.ckb);
```

When using interoperability, such as ETH addresses, the lock type can be specified in the contructor. The default is unspecified, which normally defaults to Omni Lock in methods such as `toCKBAddress()` and `toLockScript()`.

```typescript
// Example of creating ETH addresses while specifying the lock type in the constructor.

const address = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth);
const addressOmni = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth, undefined, LockType.omni);
const addressPw = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth, undefined, LockType.pw);
```

When outputting a CKB address using `toCKBAddress()`, the `LockType` specified in the `Address()` constructor will be used by default. A parameter can be passed to `toCKBAddress()` in order to override this to `omni` or `pw`. If a lock type was not specified in the `Address()` constructor or as a parameter to `toCKBAddress()`, Omni Lock will be used as the fallback.

Note: Specifying the lock type is not possible with the CKB address type. It can only be done using address types such as ETH.

```typescript
// Example of outputting CKB addresses using different lock types.

// Create an Address instances for an ETH address.
const address = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth);
const addressOmni = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth, undefined, LockType.omni);
const addressPw = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth, undefined, LockType.pw);

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(address.toCKBAddress()); // No lock type set in constructor or as a parameter. Default to Omni.

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(address.toCKBAddress(undefined, LockType.omni)); // Override with Omni.

// ckt1qpvvtay34wndv9nckl8hah6fzzcltcqwcrx79apwp2a5lkd07fdxxqt64l6ed309u7ywl7sta9rqzjmefnwc65gnazqe7 (PW-Lock; ETH; CKB2021)
console.log(address.toCKBAddress(undefined, LockType.pw)); // Override with PW.

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(addressOmni.toCKBAddress()); // No lock type parameter. Use lock type from constructor, Omni.

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(addressOmni.toCKBAddress(undefined, LockType.omni)); // Override with Omni.

// ckt1qpvvtay34wndv9nckl8hah6fzzcltcqwcrx79apwp2a5lkd07fdxxqt64l6ed309u7ywl7sta9rqzjmefnwc65gnazqe7 (PW-Lock; ETH; CKB2021)
console.log(addressOmni.toCKBAddress(undefined, LockType.pw)); // Override with PW.

// ckt1qpvvtay34wndv9nckl8hah6fzzcltcqwcrx79apwp2a5lkd07fdxxqt64l6ed309u7ywl7sta9rqzjmefnwc65gnazqe7 (PW-Lock; ETH; CKB2021)
console.log(addressPw.toCKBAddress()); // No lock type parameter. Use lock type from constructor, PW.

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(addressPw.toCKBAddress(undefined, LockType.omni)); // Override with Omni.

// ckt1qpvvtay34wndv9nckl8hah6fzzcltcqwcrx79apwp2a5lkd07fdxxqt64l6ed309u7ywl7sta9rqzjmefnwc65gnazqe7 (PW-Lock; ETH; CKB2021)
console.log(addressPw.toCKBAddress(undefined, LockType.pw)); // Override with PW.
```

Outputting a lock script with `toLockScript()` follows the same suit as `toCKBAddress()`. The lock type from the `Address()` constructor will be used, but it can be overridden by passing a lock type to `toLockScript()`. If a lock type was not specified in the `Address()` constructor or as a parameter to `toLockScript()`, Omni Lock will be used as the fallback.

```typescript
// Example of outputting lock hashes using different lock types.

// Create an Address instances for an ETH address.
const address = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth);
const addressOmni = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth, undefined, LockType.omni);
const addressPw = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth, undefined, LockType.pw);

// 0xa7b783daa24df36fdcc140c0c130773a5f53b287c1c201b0e145e2c07c9fa444 (Omni Lock)
console.log(address.toLockScript().toHash()); // No lock type set in constructor or as a parameter. Default to Omni.

// 0xa7b783daa24df36fdcc140c0c130773a5f53b287c1c201b0e145e2c07c9fa444 (Omni Lock)
console.log(address.toLockScript(LockType.omni).toHash()); // Override with Omni.

// 0x247213e26c8163a8c1133842b3833a3624110271923096d0a30668734a31e0d5 (PW-Lock)
console.log(address.toLockScript(LockType.pw).toHash()); // Override with PW.

// 0xa7b783daa24df36fdcc140c0c130773a5f53b287c1c201b0e145e2c07c9fa444 (Omni Lock)
console.log(addressOmni.toLockScript().toHash()); // No lock type parameter. Use lock type from constructor, Omni.

// 0xa7b783daa24df36fdcc140c0c130773a5f53b287c1c201b0e145e2c07c9fa444 (Omni Lock)
console.log(addressOmni.toLockScript(LockType.omni).toHash()); // Override with Omni.

// 0x247213e26c8163a8c1133842b3833a3624110271923096d0a30668734a31e0d5 (PW-Lock)
console.log(addressOmni.toLockScript(LockType.pw).toHash()); // Override with PW.

// 0x247213e26c8163a8c1133842b3833a3624110271923096d0a30668734a31e0d5 (PW-Lock)
console.log(addressPw.toLockScript().toHash()); // No lock type parameter. Use lock type from constructor, PW.

// 0xa7b783daa24df36fdcc140c0c130773a5f53b287c1c201b0e145e2c07c9fa444 (Omni Lock)
console.log(addressPw.toLockScript(LockType.omni).toHash()); // Override with Omni.

// 0x247213e26c8163a8c1133842b3833a3624110271923096d0a30668734a31e0d5 (PW-Lock)
console.log(addressPw.toLockScript(LockType.pw).toHash()); // Override with PW.
```

Builders and Collectors such as such as `SimpleBuilder` and `IndexerCollector` may internally rely on `toLockScript()`. It is recommended that `LockType` be passed to the `Address()` constructor if PW-Lock addresses are being used. If a lock type was not specified in the `Address()` constructor, Omni Lock will be used as the fallback.

```typescript
// Example of specifying the lock type with IndexerCollector. 

// Create an Address instance with a ETH address.
const address = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth);
const addressOmni = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth, undefined, LockType.omni);
const addressPw = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth, undefined, LockType.pw);

// Create a new IndexerCollector and SUDT instance.
const collector = new IndexerCollector(CKB_INDEXER_RPC_URL);
const sudt = new SUDT(SUDT_ID);

// Collect SUDT balances for address.
const balance1 = await collector.getSUDTBalance(sudt, address); // No lock type set in constructor or as a parameter. Default to Omni.
const balance2 = await collector.getSUDTBalance(sudt, address, {lockType: LockType.omni}); // Override with Omni.
const balance3 = await collector.getSUDTBalance(sudt, address, {lockType: LockType.pw}); // Override with PW.
const balanceOmni1 = await collector.getSUDTBalance(sudt, addressOmni); // No lock type parameter. Use lock type from constructor, Omni.
const balanceOmni2 = await collector.getSUDTBalance(sudt, addressOmni, {lockType: LockType.omni}); // Override with Omni.
const balanceOmni3 = await collector.getSUDTBalance(sudt, addressOmni, {lockType: LockType.pw}); // Override with PW.
const balancePw1 = await collector.getSUDTBalance(sudt, addressOmni); // No lock type parameter. Use lock type from constructor, PW.
const balancePw2 = await collector.getSUDTBalance(sudt, addressOmni, {lockType: LockType.omni}); // Override with Omni.
const balancePw3 = await collector.getSUDTBalance(sudt, addressOmni, {lockType: LockType.pw}); // Override with PW.

// The Omni Lock address has a balance of 123, and the PW-Lock address has a balance of 456.

// 123 SUDT Tokens
console.log(`${balance1} SUDT Tokens`);

// 123 SUDT Tokens
console.log(`${balance2} SUDT Tokens`);

// 456 SUDT Tokens
console.log(`${balance3} SUDT Tokens`);

// 123 SUDT Tokens
console.log(`${balanceOmni1} SUDT Tokens`);

// 123 SUDT Tokens
console.log(`${balanceOmni2} SUDT Tokens`);

// 456 SUDT Tokens
console.log(`${balanceOmni3} SUDT Tokens`);

// 456 SUDT Tokens
console.log(`${balancePw1} SUDT Tokens`);

// 123 SUDT Tokens
console.log(`${balancePw2} SUDT Tokens`);

// 456 SUDT Tokens
console.log(`${balancePw3} SUDT Tokens`);
```

## Addresses Compatibility

By default, the new CKB2021 address format will be used with the new Omni Lock. However, existing dapps may need to generate the previous addresses to allow users to migrate properly. In order to properly support both, it is important to understand how these two parameters affect users.

When these parameters are used together, there are a total of four possible combinations:

1. Omni Lock / CKB2021 (new default)
2. PW-Lock / CKB2021
3. Omni Lock / Pre2021
4. PW-Lock / Pre2021 (previous default)

There are four different addresses, but the combinations have different effects. When changing the address version, it is a change to the off-chain encoding of the human readable format. This means that the on-chain lock script does not change, and assets do not need to be migrated. For example, 1 and 3 both use Omni Lock. If you send assets to an address using combo 1, all the same assets will also be on combo 3 because they are the same lock. The same is true with 2 and 4 because they share the same underlying lock.

Generating all four addresses is possible but only two combinations are the most important. Combo 4 was previously the only one possible, and now only combo 1 is only one recommended.

```typescript
// How to generate all four different address combinations.

// Create an Address instance with a ETH address.
const address = new Address('0x7aaff596c5e5e788effa0be946014b794cdd8d51', AddressType.eth);

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(address.toCKBAddress());

// ckt1qpuljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqgp02hlt9k9uhnc3ml6p055vq2t09xdmr23qqx8wz0x (Omni Lock; ETH; CKB2021)
console.log(address.toCKBAddress(NervosAddressVersion.ckb2021, LockType.omni));

// ckt1qpvvtay34wndv9nckl8hah6fzzcltcqwcrx79apwp2a5lkd07fdxxqt64l6ed309u7ywl7sta9rqzjmefnwc65gnazqe7 (PW-Lock; ETH; CKB2021)
console.log(address.toCKBAddress(NervosAddressVersion.ckb2021, LockType.pw));

// ckt1q3uljza4azfdsrwjzdpea6442yfqadqhv7yzfu5zknlmtusm45hpuqt64l6ed309u7ywl7sta9rqzjmefnwc65gqr9hrgd (Omni Lock; ETH; Pre2021)
console.log(address.toCKBAddress(NervosAddressVersion.pre2021, LockType.omni));

// ckt1q3vvtay34wndv9nckl8hah6fzzcltcqwcrx79apwp2a5lkd07fdxx7407ktvte083rhl5zlfgcq5k72vmkx4zl2ye54 (PW-Lock; ETH; Pre2021)
console.log(address.toCKBAddress(NervosAddressVersion.pre2021, LockType.pw));
```

## WitnessArgs Changes

Some of the named definitions for WitnessArgs have been changed to better reflect the current offerings.

- `Builder.WITNESS_ARGS.Secp256k1` is used for the default lock and PW-Lock on the mainnet.
- `Builder.WITNESS_ARGS.Secp256k1Pw` is used for PW-Lock on the testnet.
- `Builder.WITNESS_ARGS.Secp256k1Omni` is used for Omni Lock on the testnet and mainnet.

A convinience method called `calculateWitnessArgs()` has been added to the `Builder` class to assist with this. The WitnessArgs required can change depending on which lock is used. This method will automatically select the correct one based on the lock script provided to it.

```typescript
// This method is called from within a class that extends Builder.
this.calculateWitnessArgs(PWCore.provider.address.toLockScript());
```

If you are using custom builders, you may need to make this modification. If you are relying on the built-in builders this change is already reflected.

## CellDeps Changes

When Omni Lock is used, a new Cell Dep may required. This is usually done from within a custom builder.

```typescript
cellDeps.push(PWCore.config.omniLock.cellDep);
```

If you are using custom builders, you may need to make this modification. If you are relying on the built-in builders this change is already reflected.

## Examples, Tools, and Further Reading

### Examples

- [PW-SDK Examples](https://github.com/lay2dev/pw-core/tree/dev/examples)

### Tools

- [Lumos Address Tool](https://nervosnetwork.github.io/lumos/tools/address-conversion)
- [CKB.tools Address Tool](https://ckb.tools/address)
- [PW-UP Asset Migration Tool](https://pw-up.vercel.app/)

### Further Reading

- [Omni Lock Asset Migration Guide](https://github.com/nervosnetwork/force-bridge/blob/main/docs/asset-migration-guide.md)
- [CKB Address Format](https://github.com/nervosnetwork/rfcs/blob/master/rfcs/0021-ckb-address-format/0021-ckb-address-format.md)
- [Omni Documentation on Cryptape Jungle](https://blog.cryptape.com/omnilock-a-universal-lock-that-powers-interoperability-1)
- [Omni Lock Specification](https://github.com/XuJiandong/docs-bank/blob/master/omni_lock.md)

## Where to Report Bugs and Request Assistance

If you experience problems with any of the tooling, please report your findings on the appropriate GitHub repository for the tool you are having a problem with.

If you need assistance with upgrading, please join the [Nervos Discord](https://discord.gg/nervos) and ask your question in the #dev-chat, #lumos, or #pw-core channels.

Sign up [here](https://share.hsforms.com/16bQP75V4RMK_39KQiuFy1wcadup) to have important technical updates emailed to you directly in the future.
