---
title: 'Ethereum Message Signing'
date: 2025-04-02
author: Dirga Yasa
avatar: https://avatars.githubusercontent.com/u/26251830?v=4
---

![Banner](./2025-04-02_ethereum-message-signing/banner.png)

## Kenapa Message Signature
Setiap interaksi yang dilakukan kepada sebuah smart contract pasti memerlukan gas fee.
Untuk mengoptimalkan penggunaan gas fee pada user, setiap transaksi untuk melakukan persetujuan terhadap sesuatu bisa diubah menjadi Signed Message.
Dengan Signed Message maka seorang user dapat memberikan signature kepada sebuah pesan untuk diverifikasi oleh smart contract yang dituju.
Dengan hal seperti ini maka user yang melakukan persetujuan tidak perlu membayar gas fee.

## Signed Message
EIP-191 adalah bentuk sederhana dari Signed Message.
Strukturnya adalah

> `\x19Ethereum signed message:\n` + `len(message)` + `message`

Misalnya untuk message `Hello World`, panjang message tersebut adalah 11.
Maka message yang akan dihasilkan adalah `\x19Ethereum signed message:\n11Hello World`.
Kemudian message tersebut diberikan signature oleh user.

![Sign Message](./2025-04-02_ethereum-message-signing/1-sign.png)

### Create Signature for String Message
Pada library ethers, method yang digunakan untuk memberikan signature adalah `signMessage()`.

```javascript
const message = "Hello, world!";
const signature = await user.signMessage(message)
const { v, r, s } = ethers.Signature.from(signature)
```
Nilai dari `v`, `r` dan `s` ini yang akan digunakan untuk verifikasi pada smart contract

### Verify String Message
```solitity
bytes memory prefixedMessage = abi.encodePacked(
  "\x19Ethereum Signed Message:\n",
  (bytes(value).length).toString(),
  "Hello, world!"
);

bytes32 digest = keccak256(prefixedMessage);

return ecrecover(digest, v, r, s);
```

### Create Signature for Bytes32 Message
Bytes32 message memiliki perlakukan khusus karena ethers memperlakukan string '0x1234' sebagai 6 karakter bukan 2 bytes

```javascript
const message = '0x9b74556d08a46ab198189741c71b554406488ebd87f958505c1fdaae9f6930dc';
const signature = await user.signMessage(Buffer.from(message.slice(2), 'hex'));
const { v, r, s } = ethers.Signature.from(signature);
```

### Verify Bytes32 Message
```solidity
bytes memory prefixedMessage = abi.encodePacked(
  "\x19Ethereum Signed Message:\n",
  "32",
  string(abi.encodePacked(0x9b74556d08a46ab198189741c71b554406488ebd87f958505c1fdaae9f6930dc))
)

bytes32 digest = keccak256(prefixedMessage);

return ecrecover(digest, v, r, s)
```

## Signed Typed Data
EIP-191 memiliki kelemahan yaitu karena strukturnya yang sederhana, data yang telah di-sign oleh user dapat digunakan dimana saja sehingga tidak cocok untuk signature yang digunakan untuk sebuah smart contract saja.
EIP-712 memberikan struktur untuk memastikan bahwa sebuah message hanya bisa digunakan pada sebuah smart contract pada suatu chainId.
Strukturnya adalah

> `/x19/01` + `hashStruct(EIP712Domain)` + `hashStruct(message)`

EIP712Domain terdiri dari

```javascript
{
  name: string,
  version: string,
  chainId: number,
  verifyingContract: string,
}
```
Domain separator ini memastikan bahwa signature yang nanti dihasilkan hanya bisa digunakan pada sebuah smart contract dan tidak bisa digunakan pada smart contract yang lain.

![Sign Typed Data](./2025-04-02_ethereum-message-signing/2-sign-typed.png)

### How
Pada library ethers, method yang digunakan untuk memberikan signature adalah `signTypedData()`.

#### Create Signature
