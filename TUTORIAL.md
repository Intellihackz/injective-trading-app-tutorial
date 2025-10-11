# Building a Spot Trading Interface on Injective Protocol

Welcome! In this tutorial, we're going to build a simple but fully functional trading interface on Injective Protocol from scratch. By the end, you'll have a React app where users can connect their MetaMask wallet, see live market data, browse real-time order books, and place actual trades on Injective's testnet.

Don't worry if you've never worked with blockchain protocols before - I'll walk you through every step and explain not just *what* we're doing, but *why* we're doing it. Think of this as us coding together!

## Table of Contents

- [Prerequisites](#prerequisites)
- [What We're Building](#what-were-building)
- [How This Will Work](#how-this-will-work)
- [Step 1: Setting Up Our Project](#step-1-setting-up-our-project)
- [Step 2: Connecting MetaMask Wallet](#step-2-connecting-metamask-wallet)
- [Step 3: Fetching Available Markets](#step-3-fetching-available-markets)
- [Step 4: Building a Live Order Book](#step-4-building-a-live-order-book)
- [Step 5: Creating Our Trading Form](#step-5-creating-our-trading-form)
- [Step 6: Making Orders Actually Work - Transaction Signing](#step-6-making-orders-actually-work---transaction-signing)
- [Step 7: Showing User Account Information](#step-7-showing-user-account-information)
- [Step 8: Making It Look Professional - Styling & Polish](#step-8-making-it-look-professional---styling--polish)
- [Conclusion](#conclusion)

## Prerequisites

Before we jump in, let's make sure you have everything you need. Don't worry - we won't need anything too fancy:

- **Node.js 18+** - We'll be using modern React features
- **Basic React knowledge** - You should be comfortable with hooks, state, and useEffect
- **MetaMask wallet** - Install it if you haven't already, and make sure it's configured
- **Basic trading concepts** - Know what buy/sell orders are and how order books work
- **Some TypeScript familiarity** - Helpful but not required; I'll explain as we go

If you're missing any of these, no worries! You can pick them up as we go along.

## Complete Code Repository

📁 **[View the complete source code on GitHub](https://github.com/Intellihackz/inject)**

### 📖 How to Use This Tutorial

This tutorial can be used in two ways:

1. **Learning Mode** - Follow along step-by-step and build everything from scratch to understand how each piece works
2. **Reference Mode** - If you've already cloned the completed repository, use this tutorial to understand the implementation details of each feature

> **💡 Tip**: If you're working with the completed code, look for the **"📝 Note"** sections that indicate when functionality is already implemented. These sections explain how the existing code works rather than asking you to write it.

## What We're Building

Here's what your trading interface will be able to do by the end of this tutorial:

- **Connect MetaMask wallets** - Users can connect and see their Injective address
- **Browse live markets** - See all available trading pairs on Injective
- **View real-time order books** - Live buy/sell orders updating in real-time
- **Place actual trades** - Submit limit and market orders to the blockchain
- **Track balances** - See token balances and active positions
- **Handle transactions** - Sign and broadcast everything through MetaMask

Pretty cool, right? And we'll build it all from scratch so you understand every piece.

## How This Will Work

Let me quickly explain the architecture so you know what we're building:

**Frontend**: We'll use React with TypeScript for our user interface
**Blockchain**: All trading happens on Injective Protocol (we'll use testnet for safety)
**Wallet Connection**: MetaMask handles all the wallet interactions
**Real-time Data**: WebSocket connections give us live market data

The main tools we'll use are:

- `@injectivelabs/sdk-ts` - This is Injective's main SDK that handles everything
- `@injectivelabs/networks` - Network configuration for testnet/mainnet
- React hooks - For managing state and side effects
- Plain CSS - To make it look good

Alright, let's start building!

---

## Step 1: Setting Up Our Project

Alright, let's start building! The first thing we need to do is create a new React project and install Injective's SDK. I'll walk you through each step.

### Creating Our React App

Let's start by creating a fresh React TypeScript project using Vite. Open your terminal and run:

```bash
npm create vite@latest injective-trading
```

Select React and Typescript

```bash
cd injective-trading
npm install
```

This creates a new React app with TypeScript and Vite already configured. Vite is much faster than traditional bundlers and gives us a great development experience with instant hot reload!

### Installing Injective's SDK

Now we need to install Injective's TypeScript SDK. This is the magic package that gives us access to markets, order books, wallet connections, and everything else we need:

```bash
npm install @injectivelabs/sdk-ts @injectivelabs/networks
```

Let me explain what these packages do:

- **`@injectivelabs/sdk-ts`** - This is the main SDK with all the trading functions
- **`@injectivelabs/networks`** - This helps us configure whether we're using testnet or mainnet

### Setting Up Our App Structure

Let's start by setting up the basic structure of our trading interface. We'll create the main component that will house all our trading functionality.

First, let's set up our essential imports and types:

```tsx
// filepath: src/App.tsx
import {
  getInjectiveAddress,
  IndexerGrpcSpotApi,
  IndexerGrpcSpotStream,
  ChainRestBankApi,
  ChainGrpcExchangeApi,
  MsgCreateSpotLimitOrder,
  MsgCreateSpotMarketOrder,
} from "@injectivelabs/sdk-ts";
import { getNetworkEndpoints, Network } from "@injectivelabs/networks";
import { useState, useEffect, useRef } from "react";

// TypeScript interfaces for our data structures
interface TradingPair {
  ticker: string;
  baseToken: { symbol: string; decimals: number };
  quoteToken: { symbol: string; decimals: number };
  marketId: string;
  minPriceTickSize: string;
  minQuantityTickSize: string;
  priceTensMultiplier: number;
  quantityTensMultiplier: number;
}
```

Next, let's create our main component with the essential API clients:

```tsx
// filepath: src/App.tsx
function App() {
  // Initialize Injective API clients
  const endpoints = getNetworkEndpoints(Network.Testnet);
  const indexerGrpcSpotApi = new IndexerGrpcSpotApi(endpoints.indexer);
  const indexerGrpcSpotStream = new IndexerGrpcSpotStream(endpoints.indexer);
  const chainRestBankApi = new ChainRestBankApi(endpoints.rest);
  const chainGrpcExchangeApi = new ChainGrpcExchangeApi(endpoints.grpc);

  return (
    <div className="App">
      <h1>Injective Trading Interface</h1>
      <p>We're building something awesome!</p>
    </div>
  );
}

export default App;
```

Let me break down what's happening here:

**Network Configuration**:

```tsx
const endpoints = getNetworkEndpoints(Network.Testnet);
```

This line grabs all the API endpoints for Injective's testnet. When you're ready to go live, you'd just change `Network.Testnet` to `Network.Mainnet`.

**API Clients**: Those `new IndexerGrpcSpotApi()` lines create connections to different parts of Injective:

- `indexerGrpcSpotApi` - Gets market data and order books
- `indexerGrpcSpotStream` - Streams real-time updates  
- `chainRestBankApi` - Checks wallet balances
- `chainGrpcExchangeApi` - Get wallet trading positions

These clients map to Injective's documentation:

- `indexerGrpcSpotApi` - Gets market data and order books (see <https://docs.injective.network/developers-native/query-indexer/spot#fetch-markets>).

- `indexerGrpcSpotStream` - Streams real-time updates (see <https://docs.injective.network/developers-native/query-indexer-stream/spot#stream-the-spot-orderbook> ).

- `chainRestBankApi` - Checks wallet balances (see <https://docs.injective.network/developers-native/query-chain/bank#fetching-injective-addresss-balances>).

- `chainGrpcExchangeApi` - Handles trading operations (see <https://docs.injective.network/developers-native/query-chain/exchange#fetch-the-pending-trading-rewards-points-for-injective-addresses>).

Think of these as different phone lines to different departments at Injective HQ.

### Try It Out

Let's make sure everything is working. In your terminal, run:

```bash
npm run dev
```

You should see a basic React app with "Injective Trading Interface" at the top. If you see any errors, double-check that you installed the packages correctly.

Perfect! We now have a working React app with Injective's SDK ready to go. In the next step, we'll add wallet connection so users can actually connect their MetaMask wallet.

---

## Step 2: Connecting MetaMask Wallet

Now that we have our project set up, let's add the ability for users to connect their MetaMask wallet. This is where things get interesting - we need to handle the connection, convert addresses to work with Injective, and provide nice user feedback.

### Adding State for Wallet Connection

First, let's add some [React state variables](https://react.dev/reference/react/useState) to track our wallet connection. Add these to your `App` component, right after the API client setup:

```tsx
// filepath: src/App.tsx
import { useState } from "react";

function App() {
  // ... existing API client setup ...

  // State management using React's useState hook
  const [isConnected, setIsConnected] = useState(false);
  const [addresses, setAddresses] = useState<string[]>([]);
  const [injectiveAddresses, setInjectiveAddresses] = useState<string[]>([]);
  const [error, setError] = useState<string>("");
```

Here's what each of these [state variables](https://react.dev/learn/state-a-components-memory) does:

- `isConnected` - Simple boolean to track if a wallet is connected
- `addresses` - The actual Ethereum addresses from MetaMask
- `injectiveAddresses` - Those same addresses converted to Injective format (they're different!)
- `error` - Any error messages we want to show the user

### Building the Connection Logic

Now let's create the functions that handle connecting to MetaMask. Add these functions inside your `App` component:

**Important Note**: MetaMask and other EVM wallets inject a `window.ethereum` object into your browser when they're installed. This object is a standard interface that any EVM-compatible DApp can use to interact with the wallet. Even though we're building on Injective (not Ethereum - they're separate blockchains), Injective supports EVM wallets because it's EVM-compatible. This means we can use MetaMask to connect and sign transactions, then convert the addresses to work with Injective's blockchain.

```tsx
// filepath: src/App.tsx
// Helper function to safely access MetaMask
const getEthereum = () => {
  if (!window.ethereum) {
    throw new Error("MetaMask extension not installed");
  }
  return window.ethereum;
};

const connectWallet = async () => {
  try {
    setError(""); // Clear any previous errors
    const ethereum = getEthereum();
    
    // Ask MetaMask for permission to access accounts
    const evmAddresses = await ethereum.request({
      method: "eth_requestAccounts",
    });

    // Convert Ethereum addresses to Injective format
    const injAddresses = evmAddresses.map(getInjectiveAddress);

    // Update our state
    setAddresses(evmAddresses);
    setInjectiveAddresses(injAddresses);
    setIsConnected(true);

    console.log("Ethereum Address:", evmAddresses[0]);
    console.log("Injective Address:", injAddresses[0]);
  } catch (err) {
    setError(err instanceof Error ? err.message : "Failed to connect wallet");
    setIsConnected(false);
  }
};

const disconnectWallet = () => {
  setIsConnected(false);
  setAddresses([]);
  setInjectiveAddresses([]);
  setError("");
};
```

The most important part here is the address conversion. Ethereum addresses (like `0x1234...`) need to be converted to Injective's format (like `inj1234...`) - that's what `getInjectiveAddress()` does for us.

For example, the Ethereum address `0xbcef77fa01d59138a494ff468ba9f6a8b2ee12f4` converts to the Injective address `inj1hnhh07sp6kgn3fy5largh20k4zewuyh5qsca2p`. They represent the same wallet, just in different formats for their respective blockchains.

### Creating the UI

Now let's create a nice header with a connect/disconnect button. First, add this helper function:

```typescript
// Helper to make long addresses more readable
const formatAddress = (address: string) => {
  if (!address) return "";
  return `${address.slice(0, 8)}...${address.slice(-6)}`;
};
```

Then update your JSX return to include this header:

```typescript
return (
  <div className="App">
    <header className="header">
      <div className="header-container">
        <div className="logo">
          <h2>Injective Trading</h2>
        </div>
        {!isConnected ? (
          <button className="connect-button" onClick={connectWallet}>
            Connect Wallet
          </button>
        ) : (
          <div className="wallet-info-header">
            <div className="connected-address">
              <span className="address-label">Connected:</span>
              <span className="address-text" title={injectiveAddresses[0] || ""}>
                {formatAddress(injectiveAddresses[0] || "")}
              </span>
            </div>
            <button className="disconnect-button" onClick={disconnectWallet}>
              Disconnect
            </button>
          </div>
        )}
      </div>
    </header>
    
    {error && (
      <div className="error-message">
        {error}
      </div>
    )}
    
    <main>
      <h1>We're building something awesome!</h1>
      {isConnected && (
        <p>✅ Wallet connected! Address: {formatAddress(injectiveAddresses[0] || "")}</p>
      )}
    </main>
  </div>
);
```

### Test the Connection

Save your file and check your browser. You should now see a "Connect Wallet" button. When you click it:

1. MetaMask should pop up asking for permission
2. After approving, you'll see your connected address in the header
3. Check the browser console to see both your Ethereum and Injective addresses

Pretty cool! The key thing to understand here is that we're taking your regular Ethereum address from MetaMask and converting it to work with Injective's blockchain. Same wallet, different format.

Next up, we'll start fetching real market data from Injective so users can see what's available to trade!

- **Error Handling**: Always wrap wallet interactions in try-catch blocks
- **State Management**: Keep wallet connection state synchronized

---

## Step 3: Fetching Available Markets

Great! Now we have wallet connection working. Let's make our trading interface actually useful by fetching real market data from Injective. This is where we'll see what trading pairs are available - like INJ/USDT, ATOM/USDT, etc.

### Setting Up Market Data Types

First, let's define what a trading pair looks like. This helps TypeScript understand our data structure and catches errors early:

```tsx
// filepath: src/App.tsx
interface TradingPair {
  marketId: string;
  ticker: string;
  baseDenom: string;
  quoteDenom: string;
  marketStatus: string;
  baseToken?: {
    name: string;
    symbol: string;
    decimals: number;
  };
  quoteToken?: {
    name: string;
    symbol: string;
    decimals: number;
  };
}
```

The important fields here are `ticker` (like "INJ/USDT"), `marketId` (unique identifier), and `marketStatus` (whether it's active).

### Adding State for Markets

Now let's add some [React state variables](https://react.dev/reference/react/useState) to track our market data:

```tsx
// filepath: src/App.tsx
// Add these with your other state variables
const [selectedPair, setSelectedPair] = useState<string>("");
const [tradingPairs, setTradingPairs] = useState<TradingPair[]>([]);
const [marketsLoading, setMarketsLoading] = useState(true);
const [marketsError, setMarketsError] = useState<string>("");
```

Here's what each does:

- `selectedPair` - Which trading pair the user has chosen (like "INJ/USDT")
- `tradingPairs` - Array of all available markets
- `marketsLoading` - Show loading state while fetching
- `marketsError` - Any errors that happen while fetching

### Fetching Markets from Injective

Now for the exciting part - let's actually fetch real market data! We'll use [React's useEffect hook](https://react.dev/reference/react/useEffect) to load this data when the component first renders:

```tsx
// filepath: src/App.tsx
import { useEffect } from "react";

// Add this useEffect inside your App component
useEffect(() => {
  const fetchMarkets = async () => {
    try {
      setMarketsLoading(true);
      setMarketsError("");

      console.log("Fetching markets from Injective testnet...");
      const markets = await indexerGrpcSpotApi.fetchMarkets();
      
      // Clean up the data for easier use
      const formattedMarkets: TradingPair[] = markets.map((market) => ({
        marketId: market.marketId,
        ticker: market.ticker,
        baseDenom: market.baseDenom,
        quoteDenom: market.quoteDenom,
        marketStatus: market.marketStatus,
        baseToken: market.baseToken,
        quoteToken: market.quoteToken,
      }));

      setTradingPairs(formattedMarkets);

      // Pick the first market as default
      if (formattedMarkets.length > 0 && !selectedPair) {
        setSelectedPair(formattedMarkets[0].ticker);
      }
    } catch (err) {
      console.error("Error fetching markets:", err);
      setMarketsError(
        err instanceof Error ? err.message : "Failed to fetch markets"
      );
    } finally {
      setMarketsLoading(false);
    }
  };

  fetchMarkets();
}, []); // Empty dependency array means this runs once when component mounts
```

This function:

1. **Calls Injective's API** to get all available spot markets
2. **Formats the data** into our TypeScript interface  
3. **Handles errors** gracefully if something goes wrong
4. **Sets a default selection** so users don't see an empty dropdown

### Creating the Market Selector UI

Now let's create a dropdown where users can choose which market to trade. Add this helper function first:

```typescript
// filepath: src/App.tsx
const getCurrentPairData = () => {
  return (
    tradingPairs.find((pair) => pair.ticker === selectedPair) ||
    tradingPairs[0]
  );
};
```

Then update your JSX to include the market selector. Add this after your header:

```typescript
// filepath: src/App.tsx
<div className="market-selector">
  <h3>Choose Market</h3>
  {marketsLoading ? (
    <div className="loading-message">Loading markets...</div>
  ) : marketsError ? (
    <div className="error-message">
      Error loading markets: {marketsError}
    </div>
  ) : (
    <div className="market-controls">
      <select
        value={selectedPair}
        onChange={(e) => setSelectedPair(e.target.value)}
        className="pair-select"
        disabled={tradingPairs.length === 0}
      >
        {tradingPairs.map((pair) => (
          <option key={pair.marketId} value={pair.ticker}>
            {pair.ticker} ({pair.marketStatus})
          </option>
        ))}
      </select>
    </div>
  )}
</div>
```

### Testing Your Market Data

Save your file and check your browser. You should now see:

1. A "Choose Market" section with a dropdown
2. Real trading pairs from Injective testnet (like INJ/USDT, ATOM/USDT)
3. Each option shows the market status (like "active")

![Market Selector Interface](./public/tutotial-images/market.png)
*Your market selector should look like this - a clean dropdown showing available trading pairs*

Check your browser console too - you'll see the full market data being logged. To open the console: press **F12** (or **Ctrl+Shift+I** on Windows/Linux, **Cmd+Option+I** on Mac), then click the "Console" tab.

![Console Output](./public/tutotial-images/console.png)
*The browser console shows all the market data being fetched from Injective's blockchain*

Pretty cool that we're pulling real data from a live blockchain!

The key thing to understand here is that we're not just showing dummy data - these are actual trading pairs that people are using to trade on Injective right now. In the next step, we'll start showing real-time order book data for whichever market the user selects.

- **Error Handling**: Always handle loading and error states for better UX
- **Default Selection**: Set a default market to prevent undefined state

---

## Step 4: Building a Live Order Book

This is where things get really exciting! We're going to create a live order book that shows real buy and sell orders updating in real-time. Imagine watching the actual heartbeat of trading on Injective - that's what we're building.

### Important Performance Pattern: Fetch First, Then Stream

A critical best practice when working with order books is to **fetch the initial snapshot first**, then connect to the real-time stream. This pattern:

- Provides immediate data to users (better UX)
- Prevents race conditions between initial load and stream setup
- Ensures you always have a complete orderbook state

Here's the recommended pattern from Injective's documentation:

```tsx
// First, fetch the initial orderbook snapshot
const initialOrderbook = await indexerGrpcSpotApi.fetchOrderbookV2(marketId);
processOrderbookData(initialOrderbook);

// Then setup the stream for real-time updates
const stream = indexerGrpcSpotStream.streamSpotOrderbookV2({
  marketIds: [marketId],
  callback: processOrderbookData,
});
```

### Understanding Order Books

Before we code, let me quickly explain what an order book is. It's basically two lists:

- **Buy orders (bids)** - People wanting to buy, sorted from highest price to lowest
- **Sell orders (asks)** - People wanting to sell, sorted from lowest price to highest

The gap between the highest buy price and lowest sell price is called the "spread."

**Important Note**: Most blockchain DEXs (decentralized exchanges) use AMM (Automated Market Maker) formulas for trading, where prices are determined algorithmically by liquidity pools. However, Injective uses a fully **on-chain order book** - the same mechanism used by traditional finance platforms like stock exchanges (NYSE, NASDAQ). This gives traders more control over their entry and exit prices, better price discovery, and the ability to place limit orders, just like professional trading platforms.

**learn more about orderbooks** Check out [Investopedia's guide to order books](https://www.investopedia.com/terms/o/order-book.asp) for a deeper dive into order book mechanics

### Setting Up Order Book Types

Let's define what an order book entry looks like:

```tsx
// filepath: src/App.tsx
interface OrderBookEntry {
  price: string;
  quantity: string;
  timestamp: number;
}
```

### Adding State for the Order Book

Now let's add [React state variables](https://react.dev/reference/react/useState) to track our order book data:

```tsx
// filepath: src/App.tsx
import { useRef } from "react";

// Add these state variables
const [buyOrders, setBuyOrders] = useState<OrderBookEntry[]>([]);
const [sellOrders, setSellOrders] = useState<OrderBookEntry[]>([]);
const [orderbookLoading, setOrderbookLoading] = useState(true);
const [currentPrice, setCurrentPrice] = useState<number>(0);

// Ref for managing WebSocket connections
const streamRef = useRef<any>(null);
```

Learn more about [useRef](https://react.dev/reference/react/useRef) for managing WebSocket connections.

### Creating the Real-Time Stream

Now for the cool part - let's connect to Injective's real-time order book stream! We'll use [React's useEffect](https://react.dev/reference/react/useEffect) to manage the WebSocket connection, following the best practice of fetching initial data first:

```tsx
// filepath: src/App.tsx
useEffect(() => {
  if (!selectedPair || tradingPairs.length === 0) return;

  const currentMarket = getCurrentPairData();
  if (!currentMarket) return;

  console.log("Fetching initial orderbook and starting stream for market:", currentMarket.marketId);
  setOrderbookLoading(true);

  // Shared function to process orderbook data from both initial fetch and stream
  const processOrderbookData = (orderbooks: any) => {
    try {
      const orderbook = orderbooks?.orderbook || orderbooks;
      
      if (orderbook && (orderbook.buys || orderbook.sells)) {
        // Process buy orders (top 10)
        const processedBuyOrders: OrderBookEntry[] = orderbook.buys
          ? orderbook.buys.slice(0, 10).map((order: any) => ({
              price: order.price,
              quantity: order.quantity,
              timestamp: order.timestamp || Date.now(),
            }))
          : [];

        // Process sell orders (top 10)
        const processedSellOrders: OrderBookEntry[] = orderbook.sells
          ? orderbook.sells.slice(0, 10).map((order: any) => ({
              price: order.price,
              quantity: order.quantity,
              timestamp: order.timestamp || Date.now(),
            }))
          : [];

        setBuyOrders(processedBuyOrders);
        setSellOrders(processedSellOrders);
        
        // Calculate current market price (mid-price between best bid and ask)
        if (processedBuyOrders.length > 0 && processedSellOrders.length > 0) {
          const bestBid = parseFloat(processedBuyOrders[0].price);
          const bestAsk = parseFloat(processedSellOrders[0].price);
          const midPrice = (bestBid + bestAsk) / 2;
          setCurrentPrice(midPrice);
          console.log(`Current price updated: ${midPrice} (Bid: ${bestBid}, Ask: ${bestAsk})`);
        } else if (processedBuyOrders.length > 0) {
          setCurrentPrice(parseFloat(processedBuyOrders[0].price));
        } else if (processedSellOrders.length > 0) {
          setCurrentPrice(parseFloat(processedSellOrders[0].price));
        }
        
        setOrderbookLoading(false);
      }
    } catch (err) {
      console.error("Error processing orderbook data:", err);
      setOrderbookLoading(false);
    }
  };

  // First, fetch the initial orderbook snapshot
  const fetchInitialOrderbook = async () => {
    try {
      console.log("Fetching initial orderbook snapshot...");
      const initialOrderbook = await indexerGrpcSpotApi.fetchOrderbookV2(
        currentMarket.marketId
      );
      console.log("Initial orderbook fetched:", initialOrderbook);
      processOrderbookData(initialOrderbook);
    } catch (err) {
      console.error("Error fetching initial orderbook:", err);
      setOrderbookLoading(false);
    }
  };

  // Fetch initial data
  fetchInitialOrderbook();

  // Then setup the stream for real-time updates
  const streamFn = indexerGrpcSpotStream.streamSpotOrderbookV2.bind(
    indexerGrpcSpotStream
  );

  const streamCallback = (orderbooks: any) => {
    console.log("Received orderbook stream update");
    processOrderbookData(orderbooks);
  };

  try {
    streamRef.current = streamFn({
      marketIds: [currentMarket.marketId],
      callback: streamCallback,
    });
  } catch (err) {
    console.error("Error starting orderbook stream:", err);
  }

  // Cleanup function - important for WebSockets!
  return () => {
    if (streamRef.current) {
      console.log("Cleaning up orderbook stream");
      streamRef.current = null;
    }
  };
}, [selectedPair, tradingPairs]);
```

This implementation:

1. **Fetches initial data** immediately for instant display
2. **Processes data** using a shared function for consistency
3. **Sets up streaming** for real-time updates after initial data loads
4. **Calculates** the current market price dynamically
5. **Cleans up** properly when the component unmounts or market changes

### Why This Pattern Matters

The fetch-first-then-stream pattern prevents common issues:

- **No race conditions**: Initial data loads before stream starts
- **Better UX**: Users see data immediately instead of waiting for first stream update
- **Consistent state**: Shared processing function ensures data format consistency

### Smart Price Formatting for Tiny Numbers

Before we build the UI, let's add a smart price formatting function. Many tokens (especially memecoins) have very small prices like `0.00005753`, which would normally display as `0.000058` when rounded. Professional DEX platforms like DexScreener use **subscript notation** to compress leading zeros:

- `0.00005753` → `0.0₄5753` (the ₄ means "4 zeros after the decimal")
- `0.00000002001` → `0.0₇2001` (the ₇ means "7 zeros")

This keeps the display clean while preserving significant digits. Add this function at the top level of your App.tsx:

```typescript
// filepath: src/App.tsx
// Format small prices with leading zero compression (like DexScreener)
const formatSmallPrice = (price: number, quoteSymbol: string = "$"): string => {
  if (price === 0) return `${quoteSymbol}0.00`;
  if (price >= 0.0001) {
    return `${price.toFixed(4)}`;
  }
  
  // Convert to string to count leading zeros
  const priceStr = price.toFixed(20).replace(/\.?0+$/, ''); // Remove trailing zeros
  const match = priceStr.match(/^0\.0+/);
  
  if (!match) return `${price}`;
  
  const leadingZeros = match[0].length - 2; // Subtract "0."
  
  // Limit to max 9 leading zeros (prevents subscript going beyond single digits)
  if (leadingZeros > 9) {
    const adjustedZeros = Math.min(leadingZeros, 9);
    const startPos = match[0].length + (leadingZeros - adjustedZeros);
    const significantDigits = priceStr.slice(startPos).slice(0, 4);
    
    const subscripts = ['₀', '₁', '₂', '₃', '₄', '₅', '₆', '₇', '₈', '₉'];
    const zeroCount = adjustedZeros.toString().split('').map(d => subscripts[parseInt(d)]).join('');
    
    return `0.0${zeroCount}${significantDigits}`;
  }
  
  const significantDigits = priceStr.slice(match[0].length).slice(0, 4);
  
  // Use subscript numbers: ₀₁₂₃₄₅₆₇₈₉
  const subscripts = ['₀', '₁', '₂', '₃', '₄', '₅', '₆', '₇', '₈', '₉'];
  const zeroCount = leadingZeros.toString().split('').map(d => subscripts[parseInt(d)]).join('');
  
  return `0.0${zeroCount}${significantDigits}`;
};
```

We'll also need a helper to get the quote token symbol (INJ, USDT, etc.) dynamically:

```typescript
// filepath: src/App.tsx
const getQuoteSymbol = () => {
  const currentMarket = getCurrentPairData();
  return currentMarket?.quoteToken?.symbol || "$";
};
```

**Why This Matters**: Without this formatting, a price like `0.00000000007` would display as `0.00000` (misleading) or take up massive screen space. With subscript compression, it displays as `0.0₁₀7` - clean and accurate!

### Building the Order Book UI

Now let's create the visual order book with our smart price formatting. First, add a helper function for when users click on prices:

```typescript
// filepath: src/App.tsx
const handlePriceClick = (clickedPrice: string) => {
  setPrice(clickedPrice); // We'll add this state variable in the next step
};
```

Then add this to your JSX after the market selector:

```typescript
// filepath: src/App.tsx
<div className="order-book">
  <h3>Live Order Book</h3>

  {orderbookLoading ? (
    <div className="loading-message">Loading order book...</div>
  ) : (
    <>
      {/* Sell Orders (Asks) - shown at top in red */}
      <div className="sell-orders">
        <div className="order-header">
          <span>Price</span>
          <span>Quantity</span>
        </div>
        {sellOrders
          .slice()
          .reverse() // Reverse so lowest price is closest to center
          .map((order, index) => (
            <div
              key={`sell-${index}`}
              className="order-row sell-order"
              onClick={() => handlePriceClick(order.price)}
              style={{ color: '#f44336', cursor: 'pointer' }}
            >
              <span className="price">
                {/* Use smart formatting - shows 0.0₄5753 instead of 0.00005753 */}
                {formatSmallPrice(parseFloat(order.price), getQuoteSymbol())}
              </span>
              <span className="quantity">
                {parseFloat(order.quantity).toFixed(2)}
              </span>
            </div>
          ))}
        {sellOrders.length === 0 && (
          <div className="no-orders">No sell orders</div>
        )}
      </div>

      {/* Current Price Display - dynamically shows correct quote token */}
      <div className="current-price" style={{ 
        textAlign: 'center', 
        padding: '10px',
        background: '#f0f0f0',
        margin: '10px 0',
        fontWeight: 'bold'
      }}>
        {currentPrice > 0 
          ? `${formatSmallPrice(currentPrice, getQuoteSymbol())} ${getQuoteSymbol()}` 
          : 'CURRENT: --'}
      </div>

      {/* Buy Orders (Bids) - shown at bottom in green */}
      <div className="buy-orders">
        {buyOrders.map((order, index) => (
          <div
            key={`buy-${index}`}
            className="order-row buy-order"
            onClick={() => handlePriceClick(order.price)}
            style={{ color: '#4CAF50', cursor: 'pointer' }}
          >
            <span className="price">
              {/* Smart formatting for tiny prices */}
              {formatSmallPrice(parseFloat(order.price), getQuoteSymbol())}
            </span>
            <span className="quantity">
              {parseFloat(order.quantity).toFixed(2)}
            </span>
          </div>
        ))}
        {buyOrders.length === 0 && (
          <div className="no-orders">No buy orders</div>
        )}
      </div>
    </>
  )}
</div>
```

### Test Your Live Order Book

Save your file and check your browser. You should now see:

1. A "Live Order Book" section
2. Real sell orders (in red) at the top
3. Current market price in the middle
4. Real buy orders (in green) at the bottom
5. Orders updating in real-time as the market moves!

![Live Order Book](./public/tutotial-images/orderbook.png)
*Your live order book should display real-time buy and sell orders with prices updating as the market moves*

Try clicking on different prices - we'll use this feature in the next step to auto-fill trade forms.

This is incredibly cool - you're now showing real-time order book data from actual traders on Injective! The numbers you see are real people's buy and sell orders, formatted intelligently to handle everything from regular prices to tiny memecoin prices. Notice how:

- **Tiny prices** like `0.00005753` display as `0.0₄5753` (clean and readable)
- **Quote tokens** show dynamically (INJ, USDT, etc.) based on the selected market
- **Current price** updates in real-time as the market moves

In the next step, we'll build a trading form so users can place their own orders into this same order book.

---

## Step 5: Creating Our Trading Form

Excellent! Now that we have live market data and a real-time order book, it's time to build the heart of our trading interface - the form where users can actually place orders. This is where the magic happens!

### Understanding Trading Forms

Before we dive into the code, let me explain the key concepts we'll be working with:

- **Order Side**: Whether you're buying (going long) or selling (going short)
- **Order Type**: Market orders (execute immediately) vs Limit orders (wait for specific price)
- **Price**: For limit orders, what price you want to buy/sell at
- **Quantity**: How much of the asset you want to trade

Think of it like placing an order at a restaurant - you specify what you want (quantity), how you want it prepared (order type), and what you're willing to pay (price).

### Setting Up Our Form State

First, let's define the types and state variables we'll need. Add these type definitions at the top of your file with the other interfaces:

```typescript
// filepath: src/App.tsx
type OrderSide = "buy" | "sell";
type OrderType = "market" | "limit";
```

Now add these state variables with your other state declarations:

```typescript
// filepath: src/App.tsx
const [orderSide, setOrderSide] = useState<OrderSide>("buy");
const [orderType, setOrderType] = useState<OrderType>("limit");
const [price, setPrice] = useState<string>("");
const [quantity, setQuantity] = useState<string>("");
```

Here's what each state variable does:

- `orderSide` - Whether user wants to buy or sell (starts with "buy" as default)
- `orderType` - Market order (executes immediately) or limit order (waits for price)
- `price` - The price user wants to buy/sell at (for limit orders)
- `quantity` - How much they want to trade

### Adding Smart Calculations

Let's create a helper function that automatically calculates the total cost of the order. This gives users instant feedback about how much they're about to spend:

```typescript
// filepath: src/App.tsx
const calculateTotal = () => {
  const priceNum = parseFloat(price || "0");
  const quantityNum = parseFloat(quantity || "0");
  return (priceNum * quantityNum).toFixed(2);
};
```

We'll also add some smart form behavior - when users change any form values, we'll clear any previous success/error messages. Add this `useEffect`:

```typescript
// filepath: src/App.tsx
// Clear order messages when form values change
useEffect(() => {
  if (orderError || orderSuccess) {
    clearOrderMessages(); // We'll define this function soon
  }
}, [price, quantity, orderSide, orderType, selectedPair]);
```

This creates a smooth user experience - old messages don't stick around when users are making new orders.

### Building the Trading Form UI

Now for the fun part - let's build the actual form! This will be a comprehensive trading interface with tabs, inputs, and smart validation. Add this to your JSX after the order book:

```typescript
// filepath: src/App.tsx
<div className="trading-form">
  <h3>Place Your Trade</h3>

  {/* Buy/Sell Selection - These tabs let users choose their order direction */}
  <div className="order-side-tabs">
    <button
      className={`tab ${orderSide === "buy" ? "active" : ""}`}
      onClick={() => setOrderSide("buy")}
    >
      BUY
    </button>
    <button
      className={`tab ${orderSide === "sell" ? "active" : ""}`}
      onClick={() => setOrderSide("sell")}
    >
      SELL
    </button>
  </div>

  {/* Order Type Selection - Market vs Limit orders */}
  <div className="order-type-toggle">
    <label>Order Type:</label>
    <div className="radio-group">
      <label>
        <input
          type="radio"
          value="market"
          checked={orderType === "market"}
          onChange={() => setOrderType("market")}
        />
        Market (Execute Now)
      </label>
      <label>
        <input
          type="radio"
          value="limit"
          checked={orderType === "limit"}
          onChange={() => setOrderType("limit")}
        />
        Limit (Wait for Price)
      </label>
    </div>
  </div>

  {/* Price Input - Only shown for limit orders */}
  <div className="input-group">
    <label>
      {orderType === "limit" ? "Limit Price" : "Market Price (Auto)"}
    </label>
    <input
      type="number"
      value={price}
      onChange={(e) => setPrice(e.target.value)}
      disabled={orderType === "market"}
      placeholder="0.000000"
      step="0.000001"
    />
    {orderType === "market" && (
      <small style={{ color: "#666", fontSize: "0.8rem" }}>
        Market orders execute at the best available price
      </small>
    )}
  </div>

  {/* Quantity Input - How much to buy/sell */}
  <div className="input-group">
    <label>Quantity</label>
    <input
      type="number"
      value={quantity}
      onChange={(e) => setQuantity(e.target.value)}
      placeholder="0.000000"
      step="0.000001"
    />
  </div>

  {/* Total Cost Display - Real-time calculation */}
  <div className="total-display">
    <span>
      Estimated Total: ${calculateTotal()}
      {orderType === "market" && " (approximate)"}
    </span>
  </div>

  {/* Place Order Button - The main action button */}
  <button
    className={`place-order-btn ${orderSide}`}
    onClick={handlePlaceOrder} // We'll define this in the next step!
    disabled={
      isPlacingOrder ||
      !quantity ||
      (orderType === "limit" && !price) ||
      !isConnected
    }
  >
    {isPlacingOrder ? (
      <span>Placing Order...</span>
    ) : (
      `Place ${orderSide.toUpperCase()} Order`
    )}
  </button>

  {/* Helpful message when wallet isn't connected */}
  {!isConnected && (
    <div style={{ 
      padding: "0.5rem 1rem", 
      textAlign: "center", 
      fontSize: "0.8rem", 
      color: "#666", 
      fontStyle: "italic" 
    }}>
      Connect your wallet to start trading
    </div>
  )}
</div>
```

### Understanding the Form Logic

Let me break down what makes this form special:

**Smart Input Validation**: The place order button is automatically disabled when:

- Required fields are empty (quantity always, price for limit orders)
- User hasn't connected their wallet
- An order is already being placed (prevents double-clicking)

**Dynamic Labels**: Notice how the price input label changes based on order type - "Limit Price" for limit orders, "Market Price (Auto)" for market orders.

**Real-time Feedback**: The total cost updates instantly as users type, and we show approximation warnings for market orders since their final price isn't guaranteed.

**User-Friendly Design**: We use clear labels like "Execute Now" vs "Wait for Price" instead of just "Market" vs "Limit" - this helps users who aren't familiar with trading terminology.

### Connecting to Order Book Prices

Remember that `handlePriceClick` function we added to the order book? Now it becomes really useful! When users click on any price in the order book, it automatically fills in the price field of our form. This creates a seamless trading experience - users can see a price they like and immediately start building an order.

You'll notice we also have an `orderError` and `orderSuccess` state mentioned in the code. Don't worry about those yet - we'll add those in the next step when we implement the actual order placement logic.

### Testing Your Form

Save your file and check your browser. You should now see a complete trading form with:

1. **Buy/Sell tabs** that change the button color
2. **Order type radio buttons** that disable/enable the price input
3. **Smart validation** that prevents invalid orders
4. **Real-time total calculation** showing the cost
5. **Connection status** that shows helpful messages

![Trading Form Interface](./public/tutotial-images/order.png)
*Your trading form should display with buy/sell tabs, order type options, price and quantity inputs, and a dynamic place order button*

Try switching between market and limit orders to see how the form adapts. The price input should disable for market orders since they execute at whatever the current market price is.

Next up, we'll implement the actual order placement logic - this is where we'll interact with Injective's blockchain to submit real trades!

---

## Step 6: Making Orders Actually Work - Transaction Signing

This is the exciting part where we connect our beautiful form to the actual blockchain! We'll build the complete order placement system step by step, so you understand every piece of the transaction flow.

> **📝 Note**: If you're following the completed repository code, this functionality is already implemented in your `App.tsx` file. This section explains how it works so you understand the transaction flow.

## Understanding the Transaction Flow

Before we dive into the code, let me explain the complete journey of a blockchain transaction:

1. **Validate Input** - Make sure all required fields are filled
2. **Create Order Message** - Build the specific order (limit or market)
3. **Fetch Account Info** - Get your account number and sequence (like a check number)
4. **Create EIP712 Typed Data** - Format the transaction for MetaMask
5. **Sign with MetaMask** - User approves the transaction
6. **Recover Public Key** - Extract the public key from the signature
7. **Build Transaction** - Package everything into a blockchain transaction
8. **Broadcast** - Send it to Injective's network
9. **Confirm** - Wait for blockchain confirmation

Think of it like mailing a certified letter - you write it, sign it, get it notarized, send it via certified mail, and track its delivery.

---

### Step 6a: Additional Imports Needed

First, let's make sure we have all the imports we need for transaction signing. Add these to your existing imports at the top of `App.tsx`:

```typescript
// filepath: src/App.tsx
import {
  getEip712TypedData,
  createWeb3Extension,
  createTxRawEIP712,
  SIGN_AMINO,
  hexToBase64,
  recoverTypedSignaturePubKey,
  ChainRestTendermintApi,
  ChainRestAuthApi,
} from "@injectivelabs/sdk-ts";
import { EvmChainId, ChainId } from '@injectivelabs/ts-types';
import {
  BigNumberInBase,
  DEFAULT_STD_FEE,
  DEFAULT_BLOCK_TIMEOUT_HEIGHT,
} from "@injectivelabs/utils";
import { TxGrpcApi } from "@injectivelabs/sdk-ts";
```

These imports give us:

- **EIP712 functions** - For creating MetaMask-compatible signatures
- **Chain IDs** - Network identifiers for testnet/mainnet
- **Transaction utilities** - Fee calculations and transaction building
- **gRPC API** - For broadcasting transactions

### Step 6b: Setting Up Order and Account State

Add these state variables with your other state declarations:

```typescript
// filepath: src/App.tsx
// Order placement state
const [isPlacingOrder, setIsPlacingOrder] = useState(false);
const [orderError, setOrderError] = useState<string>("");
const [orderSuccess, setOrderSuccess] = useState<string>("");

// Account state for transaction signing
const [accountNumber, setAccountNumber] = useState<number>(0);
const [sequence, setSequence] = useState<number>(0);
```

**Why do we need account number and sequence?**

- **Account Number**: A unique identifier for your account on Injective (never changes)
- **Sequence**: A counter that increments with each transaction (prevents replay attacks)

Think of sequence like a check number - each transaction needs a unique, sequential number.

### Step 6c: Fetching and Auto-Loading Account Details

Create a function to fetch your account information and set it up to load automatically when the wallet connects:

```typescript
// filepath: src/App.tsx
const fetchAccountDetails = async (injectiveAddress: string) => {
  try {
    console.log("Fetching account details for address:", injectiveAddress);
    
    // Get REST endpoint for the network
    const rest = getNetworkEndpoints(Network.Testnet).rest;
    const chainRestAuthApi = new ChainRestAuthApi(rest);
    
    // Fetch account information from the blockchain
    const accountDetailsResponse = await chainRestAuthApi.fetchAccount(
      injectiveAddress
    );
    
    // Extract base account info (handles different account types)
    const baseAccount =
      accountDetailsResponse.account.base_account ||
      accountDetailsResponse.account;

    // Parse and store account details
    const fetchedAccountNumber = parseInt(baseAccount.account_number);
    const fetchedSequence = parseInt(baseAccount.sequence);

    setAccountNumber(fetchedAccountNumber);
    setSequence(fetchedSequence);

    console.log("Account details fetched:", {
      accountNumber: fetchedAccountNumber,
      sequence: fetchedSequence,
    });
  } catch (err) {
    console.error("Failed to fetch account details:", err);
    setUserPanelError(
      `Failed to fetch account details: ${
        err instanceof Error ? err.message : "Unknown error"
      }`
    );
  }
};

// Auto-fetch account details when wallet connects
useEffect(() => {
  if (isConnected && injectiveAddresses.length > 0) {
    const primaryAddress = injectiveAddresses[0];
    console.log("Fetching user data for address:", primaryAddress);

    // Fetch account details (account number and sequence)
    fetchAccountDetails(primaryAddress);
    // Fetch balances and positions
    fetchUserBalances(primaryAddress);
    fetchUserPositions(primaryAddress);
  }
}, [isConnected, injectiveAddresses, tradingPairs]);
```

**Key Points:**

- We fetch account info from Injective's REST API whenever the wallet connects
- Account number stays constant, but sequence increments with each transaction
- We need fresh sequence data before every transaction
- The useEffect ensures this happens automatically on connection

### Step 6d: Building the Order Placement Function - Setup & Validation

Now let's start building the complete `handlePlaceOrder` function. First, the setup and validation:

```typescript
// filepath: src/App.tsx
const handlePlaceOrder = async () => {
  // Validate quantity
  if (!quantity) return;
  
  // Validate wallet connection
  if (!isConnected || injectiveAddresses.length === 0) {
    setOrderError("Please connect your wallet first");
    return;
  }

  // Validate market selection
  const currentMarket = getCurrentPairData();
  if (!currentMarket) {
    setOrderError("Please select a market first");
    return;
  }

  // Validate price based on order type
  if (orderType === "limit") {
    if (!price || parseFloat(price) <= 0) {
      setOrderError("Please enter a valid price for limit order");
      return;
    }
  } else {
    // For market orders, check if we have current price from orderbook
    if (currentPrice <= 0) {
      setOrderError(
        "Market price not available yet. Please wait for orderbook to load."
      );
      return;
    }
  }

  try {
    setIsPlacingOrder(true);
    setOrderError("");
    setOrderSuccess("");

    console.log("Starting order placement process...");
    
    const injectiveAddress = injectiveAddresses[0];
    const ethereumAddress = getEthereumAddress(injectiveAddress);
```

**What's happening here?**

- We validate all required inputs before proceeding
- We clear any previous error/success messages
- We get both the Injective and Ethereum addresses (we need both!)

### Step 6e: Market Configuration and Price Conversion

Next, we need to configure the market and convert prices to blockchain format:

```typescript
    // Market configuration
    const market = {
      marketId: currentMarket.marketId,
      baseDecimals: currentMarket.baseToken?.decimals || 18,
      quoteDecimals: currentMarket.quoteToken?.decimals || 6,
      minPriceTickSize: currentMarket.minPriceTickSize,
      minQuantityTickSize: currentMarket.minQuantityTickSize,
    };

    // Get mathematical multipliers for price/quantity conversion
    // These convert human-readable numbers to blockchain format
    const tensMultipliers = getSpotMarketTensMultiplier({
      baseDecimals: market.baseDecimals,
      quoteDecimals: market.quoteDecimals,
      minPriceTickSize: market.minPriceTickSize,
      minQuantityTickSize: market.minQuantityTickSize,
    });

    const marketWithMultipliers = {
      ...market,
      priceTensMultiplier: tensMultipliers.priceTensMultiplier,
      quantityTensMultiplier: tensMultipliers.quantityTensMultiplier,
    };

    console.log("Market configuration:", marketWithMultipliers);
```

**Why the complex conversion?**
Blockchains store numbers as integers, but users think in decimals. For example:

- User sees: `1.5 INJ`
- Blockchain stores: `1500000000000000000` (1.5 × 10^18)

The multipliers handle this conversion automatically.

### Step 6f: Creating the Subaccount ID

Now we create the subaccount ID (your trading account identifier):

```typescript
    // Subaccount setup
    // Most users use subaccount index 0 (their default trading account)
    const subaccountIndex = 0;
    
    // Create subaccount ID: ethereum address + 23 zeros + index
    // Example: 0x1234...5678 + 00000000000000000000000 + 0
    const suffix = "0".repeat(23) + subaccountIndex;
    const subaccountId = ethereumAddress + suffix;
    
    console.log("Using subaccount ID:", subaccountId);
```

**What's a subaccount?**
Think of subaccounts like having multiple trading accounts under one wallet. Most people just use subaccount 0, but advanced traders might use multiple subaccounts to separate different strategies.

### Step 6g: Creating the Order Message - Limit Orders

Now we create the actual order message. Let's handle limit orders first:

```typescript
    // Determine order type: 1 = Buy, 2 = Sell
    const orderTypeValue = orderSide === "buy" ? 1 : 2;
    const feeRecipient = injectiveAddress;

    let msg; // This will hold our order message

    if (orderType === "limit") {
      console.log("Creating limit order message...");
      
const limitPrice = parseFloat(price);
      
      // Convert human-readable price to blockchain format
      const chainPrice = spotPriceToChainPriceToFixed({
        value: limitPrice,
        tensMultiplier: marketWithMultipliers.priceTensMultiplier,
        baseDecimals: marketWithMultipliers.baseDecimals,
        quoteDecimals: marketWithMultipliers.quoteDecimals,
      });

      // Convert human-readable quantity to blockchain format
      const chainQuantity = spotQuantityToChainQuantityToFixed({
        value: parseFloat(quantity),
        tensMultiplier: marketWithMultipliers.quantityTensMultiplier,
        baseDecimals: marketWithMultipliers.baseDecimals,
      });

      console.log("Limit order details:", {
        userPrice: limitPrice,
        chainPrice,
        userQuantity: quantity,
        chainQuantity,
        orderSide: orderSide,
      });

      // Create the limit order message
      msg = MsgCreateSpotLimitOrder.fromJSON({
        subaccountId,
        injectiveAddress,
        orderType: orderTypeValue, // 1 for buy, 2 for sell
        price: chainPrice,
        quantity: chainQuantity,
        marketId: marketWithMultipliers.marketId,
        feeRecipient, // Where trading fees go
      });
      
      console.log("Limit order message created:", msg);
```

**What's in this message?**

- **subaccountId**: Which trading account is placing the order
- **orderType**: Buy (1) or Sell (2)
- **price**: The exact price you want (in blockchain format)
- **quantity**: How much you want to trade (in blockchain format)
- **marketId**: Which trading pair (like INJ/USDT)
- **feeRecipient**: Usually your own address (you can optionally earn fee rebates)

### Step 6h: Creating the Order Message - Market Orders

Now let's handle market orders:

```typescript
    } else {
      console.log("Creating market order message...");
      
      // For market orders, use the current market price from orderbook
      const marketPrice = currentPrice;
      
      // Convert current market price to blockchain format
      const chainPrice = spotPriceToChainPriceToFixed({
        value: marketPrice,
        tensMultiplier: marketWithMultipliers.priceTensMultiplier,
        baseDecimals: marketWithMultipliers.baseDecimals,
        quoteDecimals: marketWithMultipliers.quoteDecimals,
      });

      // Convert quantity to blockchain format
      const chainQuantity = spotQuantityToChainQuantityToFixed({
        value: parseFloat(quantity),
        tensMultiplier: marketWithMultipliers.quantityTensMultiplier,
        baseDecimals: marketWithMultipliers.baseDecimals,
      });

      console.log("Market order details:", {
        currentMarketPrice: marketPrice,
        chainPrice,
        userQuantity: quantity,
        chainQuantity,
        orderSide: orderSide,
      });

      // Create the market order message
      msg = MsgCreateSpotMarketOrder.fromJSON({
        subaccountId,
        injectiveAddress,
        orderType: orderTypeValue, // 1 for buy, 2 for sell
        price: chainPrice, // Uses current market price
        quantity: chainQuantity,
        marketId: marketWithMultipliers.marketId,
        feeRecipient,
      });
      
      console.log("Market order message created:", msg);
    }
```

**Key Difference:**

- **Limit orders**: Execute at the exact price you specify (or better)
- **Market orders**: Execute immediately at the best available price

Market orders use the current orderbook price as the starting point, but may execute at slightly different prices depending on available liquidity.

### Step 6i: EIP712 Signing, Transaction Creation, and Broadcasting

Now we complete the transaction flow - from EIP712 signing through MetaMask to broadcasting on Injective:

```typescript
    /** --- EIP712 SIGNING FLOW START --- */
    console.log("Starting EIP712 signing flow...");
    
    // Network configuration
    const chainId = ChainId.Testnet;
    const evmChainID = EvmChainId.TestnetEvm;
    const rest = getNetworkEndpoints(Network.Testnet).rest;

    // Use account details for transaction
    const currentAccountNumber = accountNumber;
    const currentSequence = sequence;

    // Fetch latest block height for timeout calculation
    const chainRestTendermintApi = new ChainRestTendermintApi(rest);
    const latestBlock = await chainRestTendermintApi.fetchLatestBlock();
    const latestHeight = latestBlock.header.height;
    const timeoutHeight = new BigNumberInBase(latestHeight).plus(
      DEFAULT_BLOCK_TIMEOUT_HEIGHT
    );

    // Create EIP712 typed data (what MetaMask shows users)
    const eip712TypedData = getEip712TypedData({
      msgs: [msg],
      tx: {
        accountNumber: currentAccountNumber.toString(),
        sequence: currentSequence.toString(),
        timeoutHeight: timeoutHeight.toFixed(),
        chainId,
      },
      evmChainId: evmChainID,
    });

    // Request signature from MetaMask
    console.log("Requesting signature from MetaMask...");
    const ethereum = getEthereum();
    const signature = await ethereum.request({
      method: "eth_signTypedData_v4",
      params: [addresses[0], JSON.stringify(eip712TypedData)],
    });

    // Recover public key from signature
    const publicKeyHex = recoverTypedSignaturePubKey(eip712TypedData, signature);
    const publicKeyBase64 = hexToBase64(publicKeyHex);
    const signatureBuff = Buffer.from(signature.replace("0x", ""), "hex");

    // Create the transaction
    const { txRaw } = createTransaction({
      message: [msg],
      memo: `${orderType.toUpperCase()} ${orderSide.toUpperCase()} order via Injective Trading App`,
      signMode: SIGN_AMINO,
      fee: DEFAULT_STD_FEE,
      pubKey: publicKeyBase64,
      sequence: currentSequence,
      timeoutHeight: timeoutHeight.toNumber(),
      accountNumber: currentAccountNumber,
      chainId,
    });

    // Create EIP712 transaction and attach signature
    const web3Extension = createWeb3Extension({ evmChainId: evmChainID });
    const txRawEip712 = createTxRawEIP712(txRaw, web3Extension);
    txRawEip712.signatures = [signatureBuff];

    // Broadcast to Injective network
    console.log("Broadcasting transaction to Injective...");
    const txRestApi = new TxGrpcApi(rest);
    const txResponse = await txRestApi.broadcast(txRawEip712);
    
    // Wait for confirmation
    console.log("Waiting for transaction confirmation...");
    const response = await txRestApi.fetchTxPoll(txResponse.txHash);
    console.log("✅ Transaction confirmed:", response);
```

**What's happening in this complete flow:**

1. **EIP712 Setup**: Configure network and transaction timeout (90 blocks ≈ 90 seconds)
2. **Typed Data**: Create human-readable transaction data for MetaMask to display
3. **User Approval**: MetaMask pops up, user reviews and signs
4. **Public Key Recovery**: Extract public key from signature to prove authenticity
5. **Transaction Building**: Package everything into a blockchain transaction
6. **Broadcasting**: Send to Injective's network
7. **Confirmation**: Wait for validators to include it in a block (~2-3 seconds)

**Key concepts:**

- **EIP712**: Standard for showing structured data in MetaMask (not just random hashes)
- **Timeout Height**: Transaction expires after 90 blocks if not processed
- **Public Key Recovery**: Proves the signature is valid without exposing private key
- **Broadcasting vs Confirmation**: Broadcasting is instant, confirmation takes ~2-3 seconds

### Step 6j: Post-Transaction Cleanup and Success

Finally, handle success and update the UI:

```typescript
    // CRITICAL: Increment sequence after successful broadcast
    // This allows the next transaction to have the correct sequence number
    setSequence((prev) => prev + 1);
    console.log("Sequence incremented to:", sequence + 1);

    // Show success message with transaction hash
    setOrderSuccess(
      `✅ Order placed successfully!\nTransaction Hash: ${txResponse.txHash}\n\nYour order has been broadcast to the Injective blockchain.`
    );

    // Reset form inputs
    setPrice("");
    setQuantity("");

    // Refresh user data after a short delay (let blockchain update)
    setTimeout(() => {
      console.log("Refreshing user data...");
      refreshUserData();
    }, 2000);

  } catch (err) {
    console.error("❌ Error placing order:", err);
    setOrderError(
      err instanceof Error ? err.message : "Failed to place order"
    );
  } finally {
    setIsPlacingOrder(false);
  }
};
```

**Critical detail: Sequence incrementing**
After a successful transaction, we **must** increment the sequence number. This ensures the next transaction uses the correct sequence. Think of it like writing check numbers - each check needs a unique, sequential number.

**Why setTimeout for refresh?**
The blockchain needs a moment to update your balances after the transaction. We wait 2 seconds to ensure the data is fresh when we refresh.

### Step 6k: Adding Helper Functions and Auto-Clear Logic

Add these helper functions for UI management and the useEffect to auto-clear messages:

```typescript
// filepath: src/App.tsx
const clearOrderMessages = () => {
  setOrderError("");
  setOrderSuccess("");
};

const refreshUserData = () => {
  if (isConnected && injectiveAddresses.length > 0) {
    const primaryAddress = injectiveAddresses[0];
    console.log("Manually refreshing account data...");
    fetchUserBalances(primaryAddress);
    fetchUserPositions(primaryAddress);
    // Also refresh account details to get updated sequence
    fetchAccountDetails(primaryAddress);
  }
};

// Clear order messages when form values change
useEffect(() => {
  if (orderError || orderSuccess) {
    clearOrderMessages();
  }
}, [price, quantity, orderSide, orderType, selectedPair]);
```

**What these do:**

- `clearOrderMessages()` - Clears error and success messages on demand
- `refreshUserData()` - Refreshes balances, positions, and account details
- The useEffect creates smooth UX by clearing old messages when users start a new order

---

## Testing Your Transaction System

Now that you understand how the transaction signing works, let's test it! Here's what should happen:

1. **Fill out the form** with quantity (and price for limit orders)
2. **Click "Place Order"** button
3. **MetaMask pops up** showing transaction details
4. **Approve in MetaMask**
5. **See success message** with transaction hash
6. **Check Injective Explorer** to see your transaction on the blockchain!

**Injective Testnet Explorer**: `https://testnet.explorer.injective.network/`

You can paste your transaction hash there to see all the details!

### 🎯 What You've Accomplished

You now have a complete blockchain transaction system that includes:

✅ **Input validation** to prevent invalid orders  
✅ **Price conversion** from human-readable to blockchain format  
✅ **EIP712 signing** for MetaMask compatibility  
✅ **Public key recovery** for transaction verification  
✅ **Transaction broadcasting** to Injective's network  
✅ **Confirmation polling** to ensure success  
✅ **Sequence management** for multiple transactions  
✅ **Error handling** for a smooth user experience  

---

## Step 7: Showing User Account Information

Now that users can place orders, they'll want to see their account information - what tokens do they have? What are their current balances? Any active positions? Let's build a user panel that shows all this information in real-time.

> **📝 Note**: If you're following the completed repository code, the user account functionality is already implemented in your `App.tsx` file. This section explains how it works so you understand account data fetching and display.

### Understanding Injective Accounts

Here's how accounts work on Injective:

- **Token Balances** - All the different tokens in your wallet (INJ, USDT, ATOM, etc.)
- **Subaccounts** - Like different trading accounts, most people use subaccount 0
- **Positions** - Your active trades and their current profit/loss
- **Orders** - Your pending buy/sell orders that haven't executed yet

Think of it like your bank account dashboard - you want to see your balance, recent transactions, and any pending activities.

### Setting Up Data Types

First, let's define what account information looks like. Add these interfaces with your other type definitions:

```typescript
// filepath: src/App.tsx
interface TokenBalance {
  symbol: string;
  amount: number;
  usdValue?: number;
  denom?: string; // The internal blockchain name for the token
}

interface Position {
  subaccountId: string;
  marketId: string;
  direction: string; // "long" or "short"
  quantity: string;
  entryPrice: string;
  margin: string;
  liquidationPrice: string;
  markPrice: string;
  ticker?: string; // Human-readable market name like "INJ/USDT"
}
```

### Adding Account State

Now let's add state variables to track user account data. Add these with your other state variables:

```typescript
// filepath: src/App.tsx
const [tokenBalances, setTokenBalances] = useState<TokenBalance[]>([]);
const [positions, setPositions] = useState<Position[]>([]);
const [balancesLoading, setBalancesLoading] = useState(false);
const [positionsLoading, setPositionsLoading] = useState(false);
const [userPanelError, setUserPanelError] = useState<string>("");
```

These track:

- `tokenBalances` - All the tokens in the user's wallet
- `positions` - Any active trading positions
- Loading states for both data types
- Error messages specific to the user panel

### Fetching Token Balances

Let's create a function to fetch all the tokens in a user's wallet. This is trickier than it sounds because token names on blockchains can be quite complex:

```typescript
// filepath: src/App.tsx
const fetchUserBalances = async (injectiveAddress: string) => {
  try {
    setBalancesLoading(true);
    setUserPanelError("");

    console.log("Fetching balances for address:", injectiveAddress);
    const balancesResponse = await chainRestBankApi.fetchBalances(injectiveAddress);
    console.log("Raw balances received:", balancesResponse);

    const balancesArray = balancesResponse.balances || [];
    const formattedBalances: TokenBalance[] = balancesArray
      .map((balance: any) => {
        // Convert amount from blockchain's base units to human-readable numbers
        // Most tokens use 18 decimal places, so we divide by 10^18
        const amount = parseFloat(balance.amount) / Math.pow(10, 18);

        // Extract user-friendly symbol from the technical denomination
        let symbol = balance.denom;

        // Handle different token naming conventions
        if (balance.denom.startsWith("factory/")) {
          // Factory tokens: extract the last part of the name
          const parts = balance.denom.split("/");
          symbol = parts[parts.length - 1] || balance.denom;
        } else if (balance.denom.startsWith("ibc/")) {
          // IBC tokens: show abbreviated hash
          symbol = "IBC/" + balance.denom.substring(4, 12) + "...";
        } else if (balance.denom.startsWith("peggy0x")) {
          // Peggy bridge tokens: show abbreviated address
          symbol = "PEGGY/" + balance.denom.substring(7, 15) + "...";
        } else if (balance.denom === "inj") {
          // Native INJ token
          symbol = "INJ";
        }

        // Keep symbols readable (max 12 characters)
        if (symbol.length > 12) {
          symbol = symbol.substring(0, 9) + "...";
        }

        return {
          symbol,
          amount,
          denom: balance.denom, // Keep original name for reference
          usdValue: 0, // We could add price fetching later
        };
      })
      .filter((balance) => balance.amount > 0); // Only show tokens with positive balance

    setTokenBalances(formattedBalances);
    console.log("Formatted balances:", formattedBalances);
  } catch (err) {
    console.error("Error fetching balances:", err);
    setUserPanelError(
      `Failed to fetch balances: ${err instanceof Error ? err.message : "Unknown error"}`
    );
  } finally {
    setBalancesLoading(false);
  }
};
```

This function does a lot of smart formatting to make blockchain token names readable for users.

### Fetching User Positions

Now let's fetch any active trading positions. This is more complex because we need to match position data with market information:

```typescript
// filepath: src/App.tsx
const fetchUserPositions = async (injectiveAddress: string) => {
  try {
    setPositionsLoading(true);
    setUserPanelError("");

    // Create the subaccount ID (this is your trading account identifier)
    const subaccountId = getDefaultSubaccountId(injectiveAddress);
    console.log("Fetching positions for subaccount:", subaccountId);

    // For spot trading, we mainly track orders rather than positions
    // But let's try to fetch both for completeness
    try {
      const orders = await indexerGrpcSpotApi.fetchSubaccountOrdersList({
        subaccountId,
      });
      console.log("Subaccount orders:", orders);
    } catch (orderError) {
      console.log("Order fetching failed (this might be normal):", orderError);
    }

    // Try to fetch positions (more relevant for derivatives trading)
    try {
      const positionsResponse = await chainGrpcExchangeApi.fetchPositions();
      console.log("Raw positions data:", positionsResponse);

      const positionsArray = Array.isArray(positionsResponse) ? positionsResponse : [];
      
      // Filter positions that belong to this user
      const userPositions = positionsArray.filter((position: any) => {
        // Check if this position belongs to our user's subaccount
        if (position.subaccountId && position.subaccountId.includes(injectiveAddress.replace("inj", ""))) {
          return true;
        }
        return true; // For demo purposes, we'll include all positions
      });

      // Format positions for display
      const formattedPositions: Position[] = userPositions
        .filter((position: any) => position && position.marketId) // Only valid positions
        .map((position: any) => {
          // Try to find the market name for this position
          const market = tradingPairs.find((pair) => pair.marketId === position.marketId);

          return {
            subaccountId: position.subaccountId || "",
            marketId: position.marketId,
            direction: position.direction || "unknown",
            quantity: position.quantity || "0",
            entryPrice: position.entryPrice || "0",
            margin: position.margin || "0",
            liquidationPrice: position.liquidationPrice || "0",
            markPrice: position.markPrice || "0",
            // Use market ticker if available, otherwise show abbreviated market ID
            ticker: market?.ticker || (position.marketId ? position.marketId.substring(0, 8) + "..." : "Unknown"),
          };
        })
        .filter((position: Position) => position.quantity !== "0"); // Only show positions with size

      setPositions(formattedPositions);
      console.log("Formatted positions:", formattedPositions);
    } catch (positionError) {
      console.log("Position fetching failed (this might be normal if no positions exist):", positionError);
      setPositions([]); // Set empty array if no positions
    }
  } catch (err) {
    console.error("Error in fetchUserPositions:", err);
    setUserPanelError(
      `Failed to fetch positions: ${err instanceof Error ? err.message : "Unknown error"}`
    );
    setPositions([]);
  } finally {
    setPositionsLoading(false);
  }
};
```

### Auto-Loading Account Data

Let's automatically fetch account data when users connect their wallet. Add this `useEffect`:

```typescript
// filepath: src/App.tsx
// Automatically fetch user data when wallet connects
useEffect(() => {
  if (isConnected && injectiveAddresses.length > 0) {
    const primaryAddress = injectiveAddresses[0];
    console.log("Wallet connected - fetching account data for:", primaryAddress);
    
    fetchUserBalances(primaryAddress);
    fetchUserPositions(primaryAddress);
  }
}, [isConnected, injectiveAddresses, tradingPairs]); // Re-run when wallet status or markets change
```

### Building the User Panel

Now let's create the UI to display all this information. First, add a refresh function:

```typescript
// filepath: src/App.tsx
const refreshUserData = () => {
  if (isConnected && injectiveAddresses.length > 0) {
    const primaryAddress = injectiveAddresses[0];
    console.log("Manually refreshing account data...");
    fetchUserBalances(primaryAddress);
    fetchUserPositions(primaryAddress);
  }
};
```

Now add the user panel JSX after your trading form:

```typescript
// filepath: src/App.tsx
<div className="user-panel">
  <div className="user-panel-header" style={{
    display: "flex",
    justifyContent: "space-between",
    alignItems: "center",
    marginBottom: "10px",
  }}>
    <h3>Your Account</h3>
    {isConnected && (
      <button
        onClick={refreshUserData}
        style={{
          fontSize: "12px",
          padding: "4px 8px",
          background: "#0066cc",
          color: "white",
          border: "none",
          borderRadius: "4px",
          cursor: "pointer",
        }}
        disabled={balancesLoading || positionsLoading}
      >
        {balancesLoading || positionsLoading ? "Refreshing..." : "Refresh"}
      </button>
    )}
  </div>

  {userPanelError && (
    <div className="error-message" style={{ color: "red", fontSize: "12px", margin: "5px 0" }}>
      {userPanelError}
    </div>
  )}

  {!isConnected && (
    <div className="connect-prompt" style={{
      textAlign: "center",
      padding: "20px",
      color: "#666",
      fontStyle: "italic"
    }}>
      Connect your wallet to view your account information
    </div>
  )}

  {isConnected && (
    <>
      {/* Token Balances Section */}
      <div className="balances-section">
        <h4>
          Wallet Balances{" "}
          {balancesLoading && (
            <span style={{ fontSize: "12px", color: "#666" }}>
              (Loading...)
            </span>
          )}
        </h4>
        <div className="balance-header">
          <span>Token</span>
          <span>Amount</span>
        </div>
        {tokenBalances.length > 0 ? (
          tokenBalances.map((balance, index) => (
            <div key={index} className="balance-row">
              <span
                className="token"
                title={`Full denomination: ${balance.denom}`} // Tooltip shows full name
                style={{
                  cursor: "help",
                  borderBottom: balance.symbol.includes("...") ? "1px dotted #666" : "none",
                }}
              >
                {balance.symbol}
              </span>
              <span className="amount">{balance.amount.toFixed(6)}</span>
            </div>
          ))
        ) : (
          <div className="no-balances" style={{ color: "#666", fontSize: "12px" }}>
            {balancesLoading ? "Loading your balances..." : "No token balances found"}
          </div>
        )}
      </div>

      {/* Active Positions Section */}
      <div className="active-positions-section">
        <h4>
          Active Positions{" "}
          {positionsLoading && (
            <span style={{ fontSize: "12px", color: "#666" }}>
              (Loading...)
            </span>
          )}
        </h4>
        <div className="orders-header">
          <span>Market</span>
          <span>Side</span>
          <span>Size</span>
        </div>
        {positions.length > 0 ? (
          positions.map((position, index) => (
            <div key={index} className="active-order-row">
              <span className="market" title={position.marketId} style={{ fontSize: "11px" }}>
                {position.ticker}
              </span>
              <span className={`side ${position.direction?.toLowerCase() || "unknown"}`}>
                {position.direction === "long"
                  ? "LONG"
                  : position.direction === "short"
                  ? "SHORT" 
                  : position.direction?.toUpperCase() || "UNKNOWN"}
              </span>
              <span className="size" style={{ fontSize: "11px" }}>
                {parseFloat(position.quantity || "0").toFixed(2)}
              </span>
            </div>
          ))
        ) : (
          <div className="no-orders" style={{ color: "#666", fontSize: "12px" }}>
            {positionsLoading ? "Loading positions..." : "No active positions"}
          </div>
        )}
      </div>
    </>
  )}
</div>
```

### What We Built

This user panel shows:

1. **Real Balances** - All tokens in the user's wallet with proper formatting
2. **Position Tracking** - Any active trading positions (more relevant for derivatives)
3. **Loading States** - Clear feedback while data is being fetched  
4. **Error Handling** - Graceful handling when API calls fail
5. **Manual Refresh** - Users can update their data anytime

The trickiest part here is handling all the different token naming conventions on Injective. Blockchain tokens can have complex internal names, but we format them to be user-friendly.

### Test Your Account Panel

Save your file and connect your wallet. You should see:

1. Your INJ balance (if you have testnet INJ)
2. Any other tokens in your wallet
3. A refresh button that updates the data
4. Loading states while data is being fetched

In the next step, we'll add CSS styling to make everything look professional and polished!

---

## Step 8: Making It Look Professional - Styling & Polish

Great! We now have a fully functional trading interface, but it probably looks pretty basic right now. Let's add some professional CSS styling to make it look like a real trading platform. Don't worry if you're not a CSS expert - I'll explain everything as we go!

### Understanding CSS Organization

Before we dive into the styling, let me explain how we'll organize our CSS:

- **Layout Styles** - Grid systems, spacing, and overall page structure
- **Component Styles** - Specific styling for buttons, forms, and panels
- **Interactive States** - Hover effects, active states, and transitions
- **Responsive Design** - Making it work on mobile devices

Think of CSS like decorating a house - we first set up the rooms (layout), then choose furniture (components), add lighting (interactive states), and make sure it works for all visitors (responsive).

### Creating Our Stylesheet

The styling for a professional trading interface requires quite a bit of CSS. Rather than overwhelming this tutorial with hundreds of lines of CSS, let's show you the key concepts and then link to the complete stylesheet.

Here's a sample of the main styling approach:

```css
/* filepath: src/App.css */
/* Reset and Base Styles */
.App {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #f5f5f5;
  min-height: 100vh;
}

/* Header with gradient background */
.header {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 1rem 2rem;
  border-radius: 8px;
  margin-bottom: 2rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

/* Interactive buttons with hover effects */
.connect-button {
  background: #4CAF50;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

.connect-button:hover {
  background: #45a049;
  transform: translateY(-1px);
}

/* Three-column responsive layout */
.main-container {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 2rem;
  margin-bottom: 2rem;
}

/* ... and much more styling for order books, forms, etc. */
```

### 📁 Get the Complete CSS File

**[Copy the complete CSS from the repository](https://github.com/Intellihackz/injective-trading-app-tutorial/blob/main/src/App.css)**

The complete stylesheet includes:

- **Professional color scheme** - Blue gradients for headers, green/red for trading actions
- **Card-style components** - Clean panels with shadows and hover effects
- **Responsive grid layout** - Three-column design that adapts to mobile
- **Interactive animations** - Smooth transitions and hover states
- **Trading-specific styling** - Order book styling, form validation states
- **Mobile responsiveness** - Works perfectly on all device sizes

Simply copy the entire CSS file from the repository link above and paste it into your `src/App.css` file.

### What These Styles Achieve

The complete CSS provides:

- **Professional Trading Look** - Matches the visual style of commercial trading platforms
- **Interactive Feedback** - Hover animations and transitions make the interface feel responsive
- **Mobile-First Design** - Grid layout automatically adjusts for smaller screens
- **Accessibility** - Proper contrast ratios and focus states for keyboard navigation
- **Performance Optimized** - GPU-accelerated animations using `transform` instead of position changes

---

**💡 Important Note:** The complete CSS file is quite large (500+ lines) and would make this tutorial too long.

**[📁 Copy the complete CSS from here](https://github.com/Intellihackz/injective-trading-app-tutorial/blob/main/src/App.css)** and paste it into your `src/App.css` file.

---

### Importing the Styles

Make sure to import your CSS file in `App.tsx`. At the top of your file, you should have:

```typescript
import './App.css';
```

If you already have this import, great! If not, add it.

### What These Styles Do

Let me explain the key styling choices we made:

**Color Scheme**: We used a professional blue-purple gradient for headers and green/red for buy/sell actions (standard trading colors).

**Interactive Effects**: Hover animations and transitions make the interface feel responsive and modern.

**Layout System**: CSS Grid creates a clean three-column layout that automatically adjusts on smaller screens.

**Typography**: We chose clean, readable fonts and proper spacing for financial data.

**Feedback Systems**: Loading animations, error styling, and success messages keep users informed.

### Testing Your New Look

Save your files and refresh your browser. Your trading interface should now look completely transformed! You'll notice:

1. **Professional Header** with gradient background and clean wallet connection
2. **Card-style Components** with subtle shadows and hover effects  
3. **Interactive Elements** that respond to your mouse with smooth transitions
4. **Mobile Responsiveness** - try resizing your browser window
5. **Proper Color Coding** - green for buys, red for sells, standard trading colors

### Customization Tips

Want to make it your own? Here are some easy customizations:

**Change the Color Scheme**:

```css
/* Replace the gradient colors in .header */
background: linear-gradient(135deg, #your-color-1 0%, #your-color-2 100%);
```

**Adjust Button Colors**:

```css
/* Modify the .place-order-btn.buy and .place-order-btn.sell colors */
background: linear-gradient(135deg, #your-buy-color 0%, #your-buy-hover-color 100%);
```

**Add Your Logo**:
Replace the text in `.logo h2` with an image or customize the text styling.

### Performance Notes

The CSS we added includes:

- **GPU-accelerated animations** using `transform` instead of position changes
- **Efficient selectors** that don't slow down rendering
- **Minimal reflows** by using flexbox and grid properly
- **Optimized transitions** that don't impact functionality

And that's it! You now have a professional-looking trading interface that rivals commercial trading platforms. The styling is responsive, accessible, and optimized for performance.

In our final sections, we'll cover troubleshooting tips and how to take this project even further!

## Conclusion

You now have a complete spot trading interface for Injective Protocol! This tutorial covered:

- ✅ Wallet integration with MetaMask
- ✅ Real-time market data fetching
- ✅ Live order book streaming
- ✅ Order placement with proper validation
- ✅ Transaction signing and broadcasting
- ✅ User account management

The foundation you've built can be extended with additional features like order history, advanced chart integration, and portfolio analytics. Remember to test thoroughly on testnet before deploying to mainnet, and always prioritize user experience and security in your implementation.

Happy trading on Injective!
