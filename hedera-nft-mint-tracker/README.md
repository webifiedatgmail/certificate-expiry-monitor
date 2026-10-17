# Hedera NFT Mint Tracker

A live feed of every NFT minted on Hedera. It is one web page, so the same
program runs on an **Android phone** and a **Windows PC**. You need nothing
except a browser such as Chrome or Edge.

## How it works

The page polls the free public Hedera Mirror Node REST API:

```
GET /api/v1/transactions?transactiontype=TOKENMINT&result=success&order=asc&timestamp=gt:<last seen>
```

Each successful `TOKENMINT` transaction that has `nft_transfers` with no sender is an
NFT mint: the new serial numbers are created and sent to the treasury account.
Fungible-token mints have no `nft_transfers`, so the page skips them. Mints made
by smart contracts through the HTS precompile come back as child `TOKENMINT`
transactions, so the page catches those too and labels them "via smart contract".

For each mint the page shows:

- the collection name and symbol (from `/api/v1/tokens/{id}`)
- the serial numbers and the receiving (treasury) account
- links to the token, account and transaction on HashScan
- the NFT metadata, with its name and image read from IPFS when "Load NFT metadata & images" is on

Captured mints are saved in the browser and can be exported to CSV.
"Start from → Where I left off" picks up from the last checked timestamp, so you
don't miss anything between sessions.

## Running it

### Windows PC

1. Download the `hedera-nft-mint-tracker` folder.
2. Double-click `index.html` to open it in Chrome or Edge, then press **Start**.

To run it like an app, serve the folder over http:

```
cd hedera-nft-mint-tracker
python -m http.server 8080
```

Then open http://localhost:8080 and choose **Install** from the address bar.

### Android phone

Opening a local HTML file is awkward on Android, so the easiest way is to host the
folder somewhere, for example with GitHub Pages:

1. In the GitHub repository, open **Settings → Pages**, choose the branch, and save.
2. On the phone, open `https://<user>.github.io/<repo>/hedera-nft-mint-tracker/` in Chrome.
3. Use the Chrome menu → **Add to Home screen**. It then opens full-screen like an app.

The same URL also works on the PC.

## Notes

- Mainnet sees a lot of mints. "24 hours ago" can take a while to catch up, and
  only the newest 500 mints stay on screen (5,000 are kept for CSV export).
- Browsers pause background tabs, especially on Android. Keep the page open in
  the foreground for continuous capture. "Where I left off" fills in the gap
  afterwards.
- The public mirror node is rate limited. The page backs off automatically when
  it gets HTTP 429.
