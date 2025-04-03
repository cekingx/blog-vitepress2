---
title: 'Ethereum Message Signing'
date: 2025-04-02
author: Dirga Yasa
avatar: https://avatars.githubusercontent.com/u/26251830?v=4
---

![Banner](./2025-04-02_ethereum-message-signing/banner.png)

## Kenapa Message Signature
Setiap interaksi yang dilakukan kepada sebuah smart contract pasti memerlukan gas fee. Untuk mengoptimalkan penggunaan gas fee pada user, setiap transaksi untuk melakukan persetujuan terhadap sesuatu bisa diubah menjadi Signed Message. Dengan Signed Message maka seorang user dapat memberikan signature kepada sebuah pesan untuk diverifikasi oleh smart contract yang dituju. Dengan hal seperti ini maka user yang melakukan persetujuan tidak perlu membayar gas fee.

## Signed Message
EIP-191 adalah bentuk sederhana dari Signed Message. Cara kerjanya adalah menambahkan string `\x19Ethereum signed message:\n` dan `len(message)`.
Misalnya untuk message `Hello World`, panjang message tersebut adalah 11. Maka message yang akan dihasilkan adalah `\x19Ethereum signed message:\n11Hello World`. Kemudian message tersebut diberikan signature oleh user. Pada library ethers, method yang digunakan untuk memberikan signature adalah `signMessage()`

## Signed Typed Data