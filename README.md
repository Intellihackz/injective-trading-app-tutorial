# Injective Trading App

A decentralized trading interface built on the Injective Protocol testnet. This application allows users to connect their MetaMask wallet, view real-time market data, and place spot trading orders on Injective's decentralized exchange.

## Features

- **Wallet Integration**: Connect with MetaMask to access Injective testnet
- **Real-time Market Data**: Live orderbook streaming and market information
- **Spot Trading**: Place limit and market orders on available trading pairs
- **Portfolio Management**: View token balances and trading positions
- **Multi-Market Support**: Trade across various spot markets available on Injective

## Tech Stack

- **Frontend**: React 19 + TypeScript + Vite
- **Blockchain**: Injective Protocol (Testnet)
- **Wallet**: MetaMask integration
- **SDK**: Injective Labs SDK (@injectivelabs/sdk-ts)
- **Styling**: CSS with responsive design

## Quick Start

1. **Install Dependencies**

   ```bash
   npm install
   ```

2. **Start Development Server**

   ```bash
   npm run dev
   ```

3. **Connect MetaMask**

   - Ensure MetaMask is installed and configured for Injective testnet
   - Click "Connect Wallet" in the application
   - Approve the connection request

4. **Start Trading**
   - Select a trading pair from the market selector
   - View real-time orderbook data
   - Place limit or market orders
   - Monitor your balances and positions

## Prerequisites

- **MetaMask**: Browser extension installed and configured
- **Testnet Tokens**: INJ testnet tokens for trading (get from Injective faucet)
- **Node.js**: Version 18+ recommended

## Available Scripts

- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run lint` - Run ESLint
- `npm run preview` - Preview production build

## Project Structure

```bash
src/
├── App.tsx          # Main application component
├── App.css          # Application styles
└── main.tsx         # Application entry point
```

## Key Dependencies

- `@injectivelabs/sdk-ts` - Core Injective Protocol SDK
- `@injectivelabs/networks` - Network configuration utilities
- `react` & `react-dom` - React framework
- `typescript` - Type safety
- `vite` - Build tool and dev server

## Documentation

For detailed usage instructions and trading guide, see [Tutorial](tutorial/0-setup.md).

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly on testnet
5. Submit a pull request

## License

MIT License - see LICENSE file for details.
