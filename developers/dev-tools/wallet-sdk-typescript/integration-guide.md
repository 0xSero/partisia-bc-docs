# Integration Guide

## Partisia Wallet Integration

The Partisia Wallet enables secure interaction with the Partisia Blockchain. This guide explains how to integrate with the Partisia Wallet in your applications.

### Wallet Architecture

The Partisia Wallet integration follows a secure communication protocol:

1. **Initial Connection**: dApps request permissions which users approve through the wallet interface
2. **End-to-End Encryption**: All communication between dApp and wallet is encrypted
3. **Permission-Based Access**: Wallets grant specific permissions to dApps
4. **Transaction Signing**: Secure signing flow for transactions and messages

### Wallet Features

* Account management
* Transaction signing
* Contract interaction
* Private key security
* Cross-origin security model

### Integration Methods

There are two primary methods to integrate with the Partisia Wallet:

#### 1. Using the Partisia SDK (Recommended)

The Partisia SDK provides a high-level abstraction for wallet interactions, handling encryption, serialization, and wallet communication.

```javascript
import PartisiaSdk from 'partisia-sdk'

const sdk = new PartisiaSdk()
await sdk.connect({
  permissions: ['sign'],
  dappName: 'My dApp',
  chainId: 'Partisia Blockchain'
})

const signResult = await sdk.signMessage({
  payload: 'Hello Partisia!',
  payloadType: 'utf8'
})
```

#### 2. Direct Browser Extension Integration

Advanced users can integrate directly with the wallet's browser extension interface:

```javascript
// Check if extension is available
if (typeof window.__onPartisiaConfirmWin === 'function' && 
    typeof window.__onPartisiaWalletTabEvent === 'function') {
    
  // Extension is present
  const payload = {
    payload: {
      permissions: ['sign'],
      dappName: 'My dApp',
      chainId: 'Partisia Blockchain'
    },
    windowType: 'connection',
    xpub: publicKeyHex,
    msgId: messageIdentifier
  }
  
  // Open wallet popup
  window.__onPartisiaConfirmWin(payload, eventCallback)
}
```

### Security Model

#### Permission Types

The Partisia Wallet implements a permission-based security model:

| Permission    | Description                   | Use Case                                |
| ------------- | ----------------------------- | --------------------------------------- |
| `sign`        | Allows requesting signatures  | Transaction signing, message signatures |
| `private_key` | Allows requesting private key | Developer tools, advanced integrations  |

#### Connection Lifecycle

1. **Initialization**: dApp creates a secure communication channel
2. **Permission Request**: dApp requests specific permissions
3. **User Approval**: User reviews and approves permissions in wallet interface
4. **Secure Channel**: End-to-end encrypted channel established
5. **Interaction**: dApp can request signatures within approved permissions
6. **Termination**: Connection ends when user closes dApp or explicitly disconnects

### Data Format

#### Connection Request

```javascript
{
  "payload": {
    "permissions": ["sign"],
    "dappName": "Example dApp",
    "chainId": "Partisia Blockchain"
  },
  "windowType": "connection",
  "xpub": "xpub661MyMwAqRbcFtXgS5sYJABqqG9YLmC4Q1Rdap9gSE8NqtwybGhePY2gZ29ESFjqJoCu1Rupje8YtGqsefD265TMg7usUDFdp6W1EGMcet8",
  "msgId": "messageIdentifier"
}
```

#### Signing Request

```javascript
{
  "payload": "encryptedPayloadHex",
  "windowType": "sign",
  "connId": "connectionIdentifier"
}
```

#### Transaction Response

```javascript
{
  "signature": "304402...",
  "serializedTransaction": "0ab401...",
  "digest": "8773e068...",
  "trxHash": "11d09178b39c10520aec717200a4a5cd229e948bc15c4a87e65d682008f86db5",
  "isFinalOnChain": false
}
```

### Event Handling

The Partisia Wallet uses an event-based communication model:

```javascript
// Listen for wallet events
function handleWalletEvent(event) {
  if (event.type === 'connected') {
    // Handle successful connection
    const { address, publicKey } = event.data
    console.log(`Connected to wallet: ${address}`)
  } else if (event.type === 'signed') {
    // Handle signature result
    const { signature, trxHash } = event.data
    console.log(`Transaction signed: ${trxHash}`)
  } else if (event.type === 'error') {
    // Handle errors
    console.error(`Wallet error: ${event.message}`)
  }
}
```

### Browser Support

The Partisia Wallet browser extension is available for:

* Chrome / Chromium-based browsers
* Firefox
* Brave
* Edge

### User Experience Best Practices

#### Connection Flow

1. **Clear Indication**: Clearly indicate to users they need to connect their Partisia Wallet
2. **Explicit Button**: Provide a distinct "Connect Wallet" button
3. **Permission Explanation**: Explain why your dApp needs specific permissions
4. **Connection Status**: Display current connection status prominently

#### Transaction Signing

1. **Transaction Preview**: Show users what they're about to sign in a human-readable format
2. **Confirmation**: Require explicit confirmation before sending signing requests
3. **Loading States**: Indicate when waiting for wallet interaction
4. **Success/Error Handling**: Clearly communicate transaction outcomes

### Error Handling Guide

Common wallet integration errors and their solutions:

| Error                             | Possible Causes            | Solutions                             |
| --------------------------------- | -------------------------- | ------------------------------------- |
| "Extension not Found"             | Wallet not installed       | Guide user to install Partisia Wallet |
| "You must connect the Dapp first" | Signing without connection | Call `sdk.connect()` before signing   |
| "User rejected request"           | User denied permission     | Improve UX or permission explanation  |
| "Connection timeout"              | Network issues             | Implement retry logic                 |

### Developer Tools

The Partisia ecosystem provides tools to simplify wallet integration:

* **Test Network**: Use the Partisia Testnet for development
* **Faucet**: Get testnet tokens for testing
* **Block Explorer**: Verify transactions on-chain
* **SDK Documentation**: Comprehensive SDK documentation

### Example Integration

Here's a complete example of a basic wallet integration:

```javascript
import PartisiaSdk from 'partisia-sdk'
import { partisiaCrypto } from 'partisia-blockchain-applications-crypto'

// Initialize SDK
const sdk = new PartisiaSdk()

// Connect to wallet button
document.getElementById('connect-button').addEventListener('click', async () => {
  try {
    await sdk.connect({
      permissions: ['sign'],
      dappName: 'My dApp',
      chainId: 'Partisia Blockchain'
    })
    
    document.getElementById('status').textContent = 
      `Connected: ${sdk.connection.account.address}`
    document.getElementById('sign-section').style.display = 'block'
  } catch (error) {
    console.error('Connection error:', error)
    document.getElementById('status').textContent = 'Connection failed'
  }
})

// Sign message button
document.getElementById('sign-button').addEventListener('click', async () => {
  try {
    const message = document.getElementById('message-input').value
    const result = await sdk.signMessage({
      payload: message,
      payloadType: 'utf8'
    })
    
    document.getElementById('result').textContent = 
      `Transaction Hash: ${result.trxHash}`
  } catch (error) {
    console.error('Signing error:', error)
    document.getElementById('result').textContent = 'Signing failed'
  }
})
```

### Advanced Integration Patterns

#### State Management

For more complex applications, use a state management solution:

```javascript
import { createStore } from 'redux'

// Wallet state reducer
function walletReducer(state = { connected: false, address: null }, action) {
  switch (action.type) {
    case 'WALLET_CONNECTED':
      return { ...state, connected: true, address: action.address }
    case 'WALLET_DISCONNECTED':
      return { ...state, connected: false, address: null }
    default:
      return state
  }
}

const store = createStore(walletReducer)

// Connect to wallet and update state
async function connectWallet() {
  try {
    await sdk.connect({
      permissions: ['sign'],
      dappName: 'My dApp',
      chainId: 'Partisia Blockchain'
    })
    
    store.dispatch({ 
      type: 'WALLET_CONNECTED', 
      address: sdk.connection.account.address 
    })
  } catch (error) {
    console.error('Connection error:', error)
  }
}
```

#### React Integration

A custom React hook for Partisia Wallet integration:

```javascript
import { useState, useEffect, useCallback } from 'react'
import PartisiaSdk from 'partisia-sdk'

function usePartisiaWallet() {
  const [sdk] = useState(() => new PartisiaSdk())
  const [connected, setConnected] = useState(false)
  const [address, setAddress] = useState(null)
  const [error, setError] = useState(null)

  const connect = useCallback(async () => {
    try {
      await sdk.connect({
        permissions: ['sign'],
        dappName: 'My React dApp',
        chainId: 'Partisia Blockchain'
      })
      
      setConnected(true)
      setAddress(sdk.connection.account.address)
      setError(null)
    } catch (err) {
      setError(err.message)
    }
  }, [sdk])

  const signMessage = useCallback(async (message) => {
    if (!connected) throw new Error('Wallet not connected')
    
    try {
      return await sdk.signMessage({
        payload: message,
        payloadType: 'utf8'
      })
    } catch (err) {
      setError(err.message)
      throw err
    }
  }, [sdk, connected])

  return { sdk, connected, address, error, connect, signMessage }
}
```
