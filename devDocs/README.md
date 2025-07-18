# Encrypted Documentation Site with Web3 Authentication

A secure documentation platform built with Nextra and Web3Auth that requires EVM wallet signature authentication and whitelist verification for access.

## 🚀 Features

- **Web3 Authentication**: Login with EVM-compatible wallets (MetaMask, WalletConnect, etc.)
- **Whitelist Access Control**: Only pre-approved wallet addresses can access documentation
- **Encrypted Content**: Documents are encrypted and only accessible to authenticated users
- **Built on Nextra**: Leverages Vercel's powerful Next.js-based documentation framework
- **MDX Support**: Write documentation with enhanced Markdown capabilities
- **Modern UI**: Clean, responsive documentation interface

## 📋 Prerequisites

- Node.js 18.x or higher
- npm/yarn/pnpm package manager
- Basic understanding of React and Next.js
- EVM-compatible wallet for testing

## 🛠️ Technology Stack

- **[Nextra](https://nextra.site/)**: Next.js-based documentation framework
- **[Web3Auth](https://web3auth.io/)**: Web3 authentication infrastructure
- **[Next.js](https://nextjs.org/)**: React framework for production
- **[ethers.js](https://docs.ethers.org/)**: Ethereum library for wallet interactions
- **TypeScript**: For type-safe development

## 📦 Installation

1. **Clone the repository**
   ```bash
   git clone <your-repo-url>
   cd encrypted-docs-site
   ```

2. **Install dependencies**
   ```bash
   npm install
   # or
   yarn install
   # or
   pnpm install
   ```

3. **Set up environment variables**
   
   Create a `.env.local` file in the root directory:
   ```env
   # Web3Auth Configuration
   NEXT_PUBLIC_WEB3AUTH_CLIENT_ID=your_web3auth_client_id
   NEXT_PUBLIC_WEB3AUTH_NETWORK=mainnet # or testnet
   
   # Encryption Key (Keep this secret!)
   ENCRYPTION_KEY=your_32_character_encryption_key
   
   # Optional: Custom RPC URLs
   NEXT_PUBLIC_RPC_URL=https://eth-mainnet.g.alchemy.com/v2/your-api-key
   ```

## 🚀 Quick Start

### 1. Set up Web3Auth

1. Go to [Web3Auth Dashboard](https://dashboard.web3auth.io/)
2. Create a new project
3. Get your Client ID
4. Configure allowed origins (add `http://localhost:3000` for development)

### 2. Configure Whitelist

Create `config/whitelist.json`:
```json
{
  "addresses": [
    "0x1234567890123456789012345678901234567890",
    "0xabcdefabcdefabcdefabcdefabcdefabcdefabcd"
  ]
}
```

### 3. Run Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Visit `http://localhost:3000` to see your documentation site.

## 📁 Project Structure

```
encrypted-docs-site/
├── components/
│   ├── Auth/
│   │   ├── LoginButton.tsx      # Web3 login component
│   │   ├── AuthProvider.tsx     # Authentication context
│   │   └── ProtectedRoute.tsx   # Route protection wrapper
│   └── Layout/
│       └── theme.config.tsx      # Nextra theme configuration
├── pages/
│   ├── _app.tsx                 # Next.js app wrapper
│   ├── index.mdx                # Home page
│   └── docs/                    # Documentation pages
│       ├── getting-started.mdx
│       └── api-reference.mdx
├── lib/
│   ├── web3auth.ts              # Web3Auth configuration
│   ├── encryption.ts            # Content encryption utilities
│   └── whitelist.ts             # Whitelist verification
├── styles/
│   └── globals.css              # Global styles
├── public/                      # Static assets
├── config/
│   └── whitelist.json           # Whitelisted addresses
├── next.config.js               # Next.js configuration
├── package.json
└── tsconfig.json
```

## 🔧 Development Guide

### Setting up Authentication

1. **Initialize Web3Auth in `lib/web3auth.ts`:**
   ```typescript
   import { Web3Auth } from "@web3auth/modal";
   import { CHAIN_NAMESPACES } from "@web3auth/base";
   
   export const initWeb3Auth = async () => {
     const web3auth = new Web3Auth({
       clientId: process.env.NEXT_PUBLIC_WEB3AUTH_CLIENT_ID!,
       chainConfig: {
         chainNamespace: CHAIN_NAMESPACES.EIP155,
         chainId: "0x1", // Mainnet
       },
     });
     
     await web3auth.initModal();
     return web3auth;
   };
   ```

2. **Create Authentication Provider:**
   ```typescript
   // components/Auth/AuthProvider.tsx
   import { createContext, useContext, useState, useEffect } from 'react';
   import { initWeb3Auth } from '@/lib/web3auth';
   
   const AuthContext = createContext({});
   
   export const AuthProvider = ({ children }) => {
     const [user, setUser] = useState(null);
     const [loading, setLoading] = useState(true);
     
     // Authentication logic here
     
     return (
       <AuthContext.Provider value={{ user, loading }}>
         {children}
       </AuthContext.Provider>
     );
   };
   ```

### Implementing Whitelist Verification

```typescript
// lib/whitelist.ts
import whitelist from '@/config/whitelist.json';

export const isWhitelisted = (address: string): boolean => {
  return whitelist.addresses
    .map(addr => addr.toLowerCase())
    .includes(address.toLowerCase());
};
```

### Content Encryption

```typescript
// lib/encryption.ts
import crypto from 'crypto';

const algorithm = 'aes-256-gcm';
const key = Buffer.from(process.env.ENCRYPTION_KEY!, 'hex');

export const encryptContent = (text: string): string => {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv(algorithm, key, iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag();
  
  return iv.toString('hex') + ':' + authTag.toString('hex') + ':' + encrypted;
};

export const decryptContent = (encryptedData: string): string => {
  const parts = encryptedData.split(':');
  const iv = Buffer.from(parts[0], 'hex');
  const authTag = Buffer.from(parts[1], 'hex');
  const encrypted = parts[2];
  
  const decipher = crypto.createDecipheriv(algorithm, key, iv);
  decipher.setAuthTag(authTag);
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
};
```

### Protecting Routes

```typescript
// components/Auth/ProtectedRoute.tsx
import { useAuth } from '@/hooks/useAuth';
import { isWhitelisted } from '@/lib/whitelist';

export const ProtectedRoute = ({ children }) => {
  const { user, loading } = useAuth();
  
  if (loading) return <div>Loading...</div>;
  
  if (!user || !isWhitelisted(user.address)) {
    return <div>Access Denied. Please login with a whitelisted address.</div>;
  }
  
  return <>{children}</>;
};
```

### Nextra Configuration

```javascript
// theme.config.tsx
import { useAuth } from '@/hooks/useAuth';

export default {
  logo: <span>My Encrypted Docs</span>,
  project: {
    link: 'https://github.com/yourusername/encrypted-docs',
  },
  navbar: {
    extraContent: () => {
      const { user, login, logout } = useAuth();
      
      return user ? (
        <button onClick={logout}>Logout ({user.address.slice(0, 6)}...)</button>
      ) : (
        <button onClick={login}>Connect Wallet</button>
      );
    },
  },
};
```

## 🔒 Security Considerations

1. **Environment Variables**: Never commit `.env.local` to version control
2. **Encryption Key**: Generate a strong 32-character key for production
3. **Whitelist Management**: Store whitelist securely and implement admin panel for updates
4. **HTTPS**: Always use HTTPS in production
5. **CORS**: Configure Web3Auth dashboard with correct origins

## 🚀 Deployment

### Deploy to Vercel

1. Push your code to GitHub
2. Import project to [Vercel](https://vercel.com)
3. Configure environment variables in Vercel dashboard
4. Deploy!

### Environment Variables for Production

- `NEXT_PUBLIC_WEB3AUTH_CLIENT_ID`: Your Web3Auth client ID
- `NEXT_PUBLIC_WEB3AUTH_NETWORK`: Set to `mainnet` for production
- `ENCRYPTION_KEY`: Strong encryption key (keep secret!)
- `NEXT_PUBLIC_RPC_URL`: Production RPC endpoint

## 📝 Writing Documentation

Create MDX files in the `pages/docs` directory:

```mdx
---
title: Getting Started
description: Learn how to use our platform
---

# Getting Started

Welcome to our encrypted documentation platform!

## Prerequisites

- EVM-compatible wallet
- Whitelisted address

## First Steps

1. Connect your wallet
2. Verify your address is whitelisted
3. Access encrypted content
```

## 🧪 Testing

```bash
# Run tests
npm test

# Run tests in watch mode
npm test:watch

# Run e2e tests
npm run test:e2e
```

## 🤝 Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.


## 🙏 Acknowledgments

- [Nextra](https://nextra.site/) for the amazing documentation framework
- [Web3Auth](https://web3auth.io/) for seamless Web3 authentication
- [Vercel](https://vercel.com/) for hosting and deployment