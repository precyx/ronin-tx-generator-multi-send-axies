# Multi Send Axie - Ronin Transaction Generator
Tool for generating transactions for sending multiple axies to different addresses.


## 1. How does it work
The tool is using `safeTransferFrom` on the Axie Contract on the Ronin Chain and signs the transactions with the mnemonic / private key.

```typescript
// create transaction
let tx = await AxieContract.populateTransaction.safeTransferFrom(fromRoninAddress, _ronin_address, _token_id);

// sign transaction
let signedTx = await wallet.signTransaction(tx);
```

## 2. Demo Use

Before using it you can test it out with the Demo seed:

**Ronin Private Key:**  
`0xf1533e116864a43d4fb7c4cd1ce550ec8c8dfeb4893fce57ea91296d156b9b37`

**Ronin Public Address:**  
`ronin:1743253c2ab78a7f086ab5c642da9b7970fe4fa8`

---

**Ronin Seed:**  
`rural control flavor ability van shrimp pig garment baby clarify express submit`

