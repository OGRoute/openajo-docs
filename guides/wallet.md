# Set up a wallet

OpenAjo runs on Stellar. You sign every action with your own wallet — OpenAjo never holds your keys. On testnet, this uses free test funds, so it costs nothing to try.

## 1. Install Freighter

[Freighter](https://freighter.app) is a browser extension wallet for Stellar. Install it for Chrome, Firefox, or Edge, then create a wallet and **back up your recovery phrase**. Anyone with that phrase controls your funds.

## 2. Switch to Testnet

Open Freighter → settings → **Network** → select **Test Net**. OpenAjo's live deployment is on testnet; make sure your wallet matches or the app will warn you.

## 3. Fund your account

A new Stellar account needs a small XLM balance to exist. On testnet, fund it for free:

1. Copy your public key from Freighter (starts with `G`).
2. Open `https://friendbot.stellar.org/?addr=YOUR_PUBLIC_KEY` in a browser, replacing `YOUR_PUBLIC_KEY`.

Your account now holds 10,000 test XLM. That is play money with no real value — it exists so you can try the full flow.

## 4. Connect

Open the OpenAjo web app and click **Connect wallet**. Freighter asks for permission the first time; approve it. Your address appears in the top bar. You are ready to [start](start-a-circle.md) or [join](join-and-contribute.md) a circle.

## What you are signing

Every time you create, join, contribute, or settle, Freighter shows you a transaction to approve. Read it. You are authorizing:

* A **deposit or contribution** — a token transfer from you into the contract.
* Or a **settlement** — which moves no money of yours; it just advances the circle.

You never sign away control of your account, and no OpenAjo server can move your funds.
