# 🔨 Real-Time Live Auction & Bidding Engine (Socket.io)

A live auction floor built with **Node.js, Express and Socket.io**. The server is
authoritative: it validates every bid, runs the countdown clock, and decides when
the auction ends — a client can't fake a bid or extend the clock itself.

## Setup

```bash
npm install
npm run dev   # or npm start
```

Open `http://localhost:5000` in multiple tabs to simulate multiple bidders.

## How to test

1. Start the server: `http://localhost:5000`.
2. Open three tabs: Bidder A (Vikram), Bidder B (Ananya), Viewer C — join the
   same auction ID (`AUC_VINTAGE_99` by default) with different names.
3. Place a bid from Vikram — all 3 screens should update to the new highest bid.
4. Place a higher bid from Ananya — Vikram should see an **"Outbid"** alert banner.
5. Wait until the timer drops below 15 seconds, then place a bid — the clock
   should jump back to 20 seconds (**Anti-Snipe Protection**).
6. Let the clock hit 0 — the room receives `auction:sold` and further bids are rejected.

## Project structure

```text
assignment-15-auction-socket/
├── public/
│   ├── index.html        # Live bidding floor UI
│   ├── app.js             # Client socket handlers & bid button
│   └── style.css          # Dark trading-floor styling
├── sockets/
│   ├── auctionEngine.js   # Bid validation, outbid alerts & anti-snipe logic
│   └── timerManager.js    # Server-side 1s interval countdown clock
├── server.js
├── package.json
└── README.md
```

## Socket event protocol

| Event | Direction | Payload |
|---|---|---|
| `auction:join` | Client → Server | `{ auctionId, username }` |
| `auction:init` | Server → Client | `{ item, bidHistory, timeRemaining }` |
| `auction:time_tick` | Server → Room | `{ auctionId, timeRemaining }` (every 1s) |
| `user:joined` | Server → Room | `{ username, totalViewers }` |
| `bid:place` | Client → Server | `{ auctionId, amount }` |
| `bid:success` | Server → Room | `{ newBid, highestBidder, bidHistory, timeRemaining }` |
| `bid:outbid` | Server → Client | `{ message }` (sent only to the previous highest bidder) |
| `bid:rejected` | Server → Client | `{ reason }` |
| `auction:extended` | Server → Room | `{ timeRemaining, message }` |
| `auction:sold` | Server → Room | `{ winner, finalPrice, status }` |

## Bid validation rules (server-side, in order)

1. Auction must be `active` with time remaining.
2. You can't outbid yourself if you're already the highest bidder.
3. Bid must be at least `currentBid + minIncrement`.
4. A valid bid placed with under 15 seconds left resets the clock to 20s
   (anti-snipe) and broadcasts `auction:extended`.

## Notes

- Auction state is **in-memory** (`auctions` object in `sockets/auctionEngine.js`),
  seeded with one demo item (`AUC_VINTAGE_99`) exactly per the assignment's data model.
- The countdown starts the moment the first bidder joins the room, so it doesn't
  tick down in an empty room.
- `bid:success` includes `bidHistory` in addition to the spec's listed fields, so
  the client can render the live feed without a separate round trip.
