# Property Registry & Transfer dApp

A student prototype showing how a smart contract can record real estate registration, ownership transfers and municipal notices transparently on the Ethereum **Sepolia** test network.

> **Course:** MG3012 - Blockchain Technology for Business | **Assignment 2:** Mini dApp
> **Program:** BS Financial Technology, 5th Semester (Fall 2026), FAST-NUCES
> **Instructor:** Dr. Usama Arshad

---

## Deployment Details

| Item | Value |
|---|---|
| Network | Ethereum Sepolia testnet (Chain ID 11155111) |
| Contract address | `PASTE_CONTRACT_ADDRESS_HERE` |
| Etherscan link | `https://sepolia.etherscan.io/address/PASTE_CONTRACT_ADDRESS_HERE` |
| Deployment transaction hash | `PASTE_DEPLOYMENT_TX_HASH_HERE` |
| Etherscan verified (bonus) | Yes / No |

---

## 1. Overview

This dApp combines two ideas in one contract:

- **Property registration and transfer:** owners register a property and can transfer it to a new wallet by paying a fee that depends on the zone and the property type. Each transfer becomes a permanent record.
- **Notices and fines:** a registrar wallet issues notices (unpaid property tax, illegal construction, encroachment). The property owner pays them, and the notice status changes on-chain.

Both features share one property record, so the system can apply a simple automated "no-dues" rule: **a property with 3 or more unpaid notices cannot be transferred until it clears them.**

## 2. Problem Statement

Property ownership records, transfer fees and notices are usually held in separate databases controlled by the operator. This makes it hard for buyers, sellers or auditors to confirm that records are complete and unchanged, and manual handling can lead to errors, disputes, fraud or record manipulation.

## 3. Proposed Blockchain Solution

Store ownership, transfers and notice states in a smart contract so that:

- every transfer and payment has a public transaction hash,
- records cannot be silently edited or deleted,
- business rules (exact fee, owner-only transfer, no double payment, no-dues check) are enforced by code,
- anyone can audit the history using events and Etherscan.

## 4. Key Features

- MetaMask wallet connection with Sepolia network check and one-click network switch
- Property registration (ID + type: Residential Plot, House, Commercial)
- Ownership transfer to a new wallet with zone- and type-based fees (Islamabad, Rawalpindi, Lahore)
- Registrar-only notice issuing (3 notice types)
- Owner-only notice payment, with duplicate-payment protection
- Live blockchain records: latest transfers, latest notices, totals
- Pending / confirmed / error messages with Etherscan transaction links
- Illustrative PKR estimates (clearly marked as simulated)

## 5. Technologies Used

- Solidity ^0.8.20 (Remix IDE)
- Ethereum Sepolia testnet and MetaMask
- HTML, CSS, JavaScript (no frameworks)
- ethers.js v5.7.2 (loaded from CDN, same as Week 8)

## 6. Smart Contract (`SmartContract.sol`)

The contract `PropertyRegistry` has **5 functions**. It also has a constructor and public state variables (Solidity creates free read getters for these).

| # | Function | Type | Who can call | MetaMask approval? | What it does |
|---|---|---|---|---|---|
| 1 | `registerProperty(propertyId, propertyType)` | Write | Anyone | Yes (gas) | Creates a property record owned by the caller. Rejects duplicate IDs. |
| 2 | `transferProperty(propertyId, newOwner, zone)` | Write, payable | Current owner | Yes (gas + fee) | Checks exact fee and the no-dues rule, changes the owner, records the transfer, forwards ETH to the registrar. |
| 3 | `issueNotice(propertyId, noticeType)` | Write | Registrar (deployer) only | Yes (gas) | Creates an unpaid notice and increases the property's unpaid count. |
| 4 | `payNotice(noticeId)` | Write, payable | Property owner | Yes (gas + fine) | Checks exact fine, marks the notice paid, forwards ETH to the registrar. |
| 5 | `getProperty(propertyId)` | Read (view) | Anyone | No | Returns owner, type, transfer count, total fees paid, unpaid notices. |

**Public read getters** used by the frontend: `registrar`, `transferCount`, `noticeCount`, `zoneBaseFee`, `typeMultiplier`, `noticeFine`, `transferRecords`, `notices`.

**Events:** `PropertyRegistered`, `PropertyTransferred`, `NoticeIssued`, `NoticePaid`.

**Payment safety:**
- Exact amounts only: `msg.value` must equal the fee or fine, so no change or refund logic is needed.
- State is updated **before** ETH is sent (prevents double payment and re-entry problems).
- Funds are forwarded straight to the registrar address, so the contract never holds a balance.

### Demo prices (Sepolia test ETH)

| Zone | Base transfer fee |
|---|---|
| Islamabad | 0.0010 ETH |
| Rawalpindi | 0.0006 ETH |
| Lahore | 0.0008 ETH |

Type multiplier: Residential Plot x1, House x2, Commercial x3.

| Notice | Fine |
|---|---|
| Unpaid property tax | 0.0010 ETH |
| Illegal construction | 0.0020 ETH |
| Encroachment | 0.0005 ETH |

These are demo numbers, not real stamp duty, CVT or authority rates. The interface shows PKR estimates using a made-up rate of 1 test ETH = PKR 1,000,000, labelled as simulated.

## 7. Architecture

```
  User (browser)
      |  clicks a button
      v
  index.html + style.css + JavaScript
      |  uses ethers.js
      v
  MetaMask  --- user approves transaction --->  Sepolia blockchain
                                                PropertyRegistry contract
      ^                                                 |
      |------------ results, events, tx hash -----------|
```

- **Backend:** the smart contract on Sepolia
- **Frontend:** one HTML page with inline JavaScript
- **Wallet:** MetaMask signs every write transaction

## 8. How to Deploy the Contract

1. Install MetaMask, create a wallet, and enable **Show test networks**. Select **Sepolia**.
2. Get free Sepolia test ETH from a faucet (about 0.02 ETH is plenty).
3. Open [Remix IDE](https://remix.ethereum.org) and create a file named `SmartContract.sol`. Paste the contract code.
4. In **Solidity Compiler**, choose version 0.8.20 or newer and click **Compile**.
5. In **Deploy & Run**, set Environment to **Injected Provider - MetaMask**. Confirm the account and that the network is Sepolia.
6. Select `PropertyRegistry` and click **Deploy**. Approve in MetaMask.
7. Copy the deployed contract address from Remix ("Deployed Contracts") and save the transaction hash.

> The wallet that deploys the contract becomes the **registrar** (it can issue notices and receives payments).

## 9. How to Configure the Frontend

Open `index.html`, find this line near the bottom, and paste your address:

```js
const CONTRACT_ADDRESS = "PASTE_YOUR_SEPOLIA_CONTRACT_ADDRESS_HERE";
```

The ABI is already included in the file and matches the contract. If you change the contract, update the `ABI` list too.

## 10. How to Run

MetaMask works best when the page is served from a local web server (not opened as a file):

```bash
# Option A (Node.js)
npx http-server .

# Option B (Python)
python -m http.server 8000
```

Then open the shown address (for example `http://localhost:8080` or `http://localhost:8000`) in a browser with MetaMask installed.

## 11. How to Test

You need two wallets: the **registrar** (the deployer) and a second **owner** wallet. A third address is the buyer.

1. Click **Connect MetaMask**. Check that your address and the green **Sepolia** badge appear.
2. With the owner wallet, **register** property `ISB-F7-101` as a House. Approve in MetaMask.
3. **Look up** `ISB-F7-101`. Check the record shows 0 transfers and 0 unpaid notices.
4. As the registrar wallet, **issue a notice** for `ISB-F7-101`.
5. As the owner, **check notices** for the property, then **pay** the notice. Status changes to Paid.
6. As the owner, **transfer** the property to another wallet address. Approve. Check the hash link, confirmation message and the transfers table. Look up the property again: the owner has changed.
7. Try invalid actions to see clear errors: unregistered property, duplicate registration, paying a paid notice, issuing a notice from a non-registrar wallet, transferring from a wallet that is not the owner, wrong network.
8. Issue 3 notices without paying and check that transfer is blocked until one is paid.

## 12. Screenshots

| Screenshot | File |
|---|---|
| Contract deployed in Remix, with address visible | `screenshots/1_remix_deployment.png` |
| Frontend connected to MetaMask on Sepolia | `screenshots/2_wallet_connected.png` |
| Successful transaction with hash and updated record | `screenshots/3_transaction_success.png` |

## 13. Limitations

- Prototype only. It is **not** integrated with CDA, RDA, LDA, the Punjab Land Records Authority, e-Stamp, Raast, JazzCash, Easypaisa, or any real land registry.
- Anyone can register any unused property ID, because there is no link to real title documents or a national database.
- No identity verification (CNIC / KYC) of buyers and sellers.
- One registrar wallet for all notices; no multiple officers or appeal process.
- Uses Sepolia test ETH, not PKR. Prices are demo values and no sale price is handled, only the transfer fee.
- Public blockchain data is visible to everyone, so real use would need privacy and legal review.
- The records tables show only the latest entries; the notice check scans the latest 100 notices.

## 14. Future Improvements

- Link to verified land records and title documents (store a document hash, e.g. on IPFS)
- Identity verification for buyers and sellers
- Escrow for the sale price, released automatically on transfer
- Officer role management (several authorised wallets)
- PKR payment rails and a stablecoin or token for payments
- Notice dispute and appeal workflow
- Event-based indexing for faster history search

## 15. Academic Information

- Course: MG3012 - Blockchain Technology for Business
- Assignment 2: Mini dApp (Solidity + HTML/JS)
- Student: _Your name and roll number here_
- Institution: FAST-NUCES
- License: MIT (educational use)

> Never commit private keys, seed phrases or `.env` files to this repository.
