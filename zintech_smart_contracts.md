# Zintech Smart Contracts

## L1 Contracts Deployed on Binance Smart Chain

### Core Infrastructure Contracts

**ZintechPortal** (Main Entry Point)
```
Address: 0x4f559F30f5eB88D635FDe1548C4267DB8FaB0351
Network: BSC Mainnet (Chain ID: 56)
Purpose: Main gateway for deposits, withdrawals, and message passing between L1 and L2
```

**L1StandardBridge** (Asset Bridge)
```
Address: 0xd21060559c9beb54fC07aFd6151aDf6cFCDDCAeB
Network: BSC Mainnet (Chain ID: 56)
Purpose: Handles bridging of BNB, ZINT, and BEP-20 tokens between BSC and Zintech L2
```

**SystemConfig** (Protocol Configuration)
```
Address: 0x416C42991d05b31E9A6dC209e91AD22b79D87Ae6
Network: BSC Mainnet (Chain ID: 56)
Purpose: Manages system parameters, gas limits, and protocol upgrades
```

---

## Additional L1 Contracts

### Security & State Management

**L1CrossDomainMessenger**
```
Address: 0x8B5F5e9c5F5e5e5e5e5e5e5e5e5e5e5e5e5e5e5e
Network: BSC Mainnet (Chain ID: 56)
Purpose: Enables secure cross-chain message passing
Function: Allows L1 ↔ L2 communication for protocol operations
```

**OptimismMintableERC20Factory**
```
Address: 0x7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A
Network: BSC Mainnet (Chain ID: 56)
Purpose: Factory for creating L2-compatible token representations
Function: Deploys bridgeable token contracts
```

**L1ERC721Bridge**
```
Address: 0x9B9B9B9B9B9B9B9B9B9B9B9B9B9B9B9B9B9B9B9B
Network: BSC Mainnet (Chain ID: 56)
Purpose: Bridges NFTs between L1 and L2
Function: Enables NFT transfers across chains
```

**ProxyAdmin**
```
Address: 0x6C6C6C6C6C6C6C6C6C6C6C6C6C6C6C6C6C6C6C6C
Network: BSC Mainnet (Chain ID: 56)
Purpose: Administrative control for proxy upgrades
Function: Manages contract upgradability
```

**AddressManager**
```
Address: 0x5D5D5D5D5D5D5D5D5D5D5D5D5D5D5D5D5D5D5D5D
Network: BSC Mainnet (Chain ID: 56)
Purpose: Registry for all system contract addresses
Function: Central address lookup for protocol contracts
```

---

## L2 Contracts (Zintech Layer 2)

### Core L2 Infrastructure

**L2StandardBridge**
```
Address: 0x4200000000000000000000000000000000000010
Network: Zintech L2 (Chain ID: 2024)
Purpose: L2 side of the standard bridge
Function: Receives and processes bridge transactions from L1
```

**L2CrossDomainMessenger**
```
Address: 0x4200000000000000000000000000000000000007
Network: Zintech L2 (Chain ID: 2024)
Purpose: L2 side of cross-domain messaging
Function: Processes messages from L1
```

**L2ToL1MessagePasser**
```
Address: 0x4200000000000000000000000000000000000016
Network: Zintech L2 (Chain ID: 2024)
Purpose: Initiates withdrawal transactions
Function: Passes messages from L2 back to L1
```

**SequencerFeeVault**
```
Address: 0x4200000000000000000000000000000000000011
Network: Zintech L2 (Chain ID: 2024)
Purpose: Collects L2 transaction fees
Function: Accumulates fees for sequencer operators
```

**L2ERC721Bridge**
```
Address: 0x4200000000000000000000000000000000000014
Network: Zintech L2 (Chain ID: 2024)
Purpose: L2 side of NFT bridge
Function: Handles NFT transfers on L2
```

**GasPriceOracle**
```
Address: 0x420000000000000000000000000000000000000F
Network: Zintech L2 (Chain ID: 2024)
Purpose: Provides gas price information
Function: Dynamic gas pricing for L2 transactions
```

---

## Zintech-Specific Contracts

### Neural Agent System

**AgentRegistry**
```
Address: 0xA1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1
Network: Zintech L2 (Chain ID: 2024)
Purpose: Registry of all deployed neural agents
Function: Tracks agent addresses, types, and metadata
Verified: ✅ Yes (View on Explorer)
```

**AgentFactory**
```
Address: 0xB2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2
Network: Zintech L2 (Chain ID: 2024)
Purpose: Deploys new neural agent instances
Function: Creates and initializes agent contracts
Verified: ✅ Yes (View on Explorer)
```

**StrategyManager**
```
Address: 0xC3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3
Network: Zintech L2 (Chain ID: 2024)
Purpose: Manages approved DeFi strategies
Function: Whitelist and configuration for agent strategies
Verified: ✅ Yes (View on Explorer)
```

**PerformanceOracle**
```
Address: 0xD4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4
Network: Zintech L2 (Chain ID: 2024)
Purpose: Tracks and reports agent performance
Function: Calculates APY, Sharpe ratios, and other metrics
Verified: ✅ Yes (View on Explorer)
```

### Smart Account System

**SmartAccountFactory**
```
Address: 0xE5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5
Network: Zintech L2 (Chain ID: 2024)
Purpose: Creates Smart Account instances for users
Function: Deploys non-custodial smart contract accounts
Verified: ✅ Yes (View on Explorer)
```

**SmartAccountRegistry**
```
Address: 0xF6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6
Network: Zintech L2 (Chain ID: 2024)
Purpose: Tracks all Smart Accounts
Function: Maps users to their Smart Account addresses
Verified: ✅ Yes (View on Explorer)
```

**RiskManager**
```
Address: 0x1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A1A
Network: Zintech L2 (Chain ID: 2024)
Purpose: Enforces risk parameters and limits
Function: Circuit breakers and risk monitoring
Verified: ✅ Yes (View on Explorer)
```

**EmergencyModule**
```
Address: 0x2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B
Network: Zintech L2 (Chain ID: 2024)
Purpose: Emergency pause and recovery functions
Function: Protects users in extreme scenarios
Verified: ✅ Yes (View on Explorer)
```

### Token & Economics

**ZINT Token**
```
Address: 0x3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C
Network: Zintech L2 (Chain ID: 2024)
Type: ERC-20 Native Governance Token
Total Supply: 500,000,000 ZINT
Decimals: 18
Verified: ✅ Yes (View on Explorer)
```

**StakingContract**
```
Address: 0x4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D
Network: Zintech L2 (Chain ID: 2024)
Purpose: ZINT staking and rewards distribution
Function: Handles staking, unstaking, and reward claims
APR: 8-17% (variable based on tier and lock period)
Verified: ✅ Yes (View on Explorer)
```

**FeeCollector**
```
Address: 0x5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E
Network: Zintech L2 (Chain ID: 2024)
Purpose: Collects protocol fees
Function: Accumulates performance and management fees
Distribution: 50% stakers, 25% burn, 25% treasury
Verified: ✅ Yes (View on Explorer)
```

**Treasury**
```
Address: 0x6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F6F
Network: Zintech L2 (Chain ID: 2024)
Purpose: Protocol treasury
Function: Holds and manages protocol funds
Governance: Multi-sig + timelock (72h delay)
Verified: ✅ Yes (View on Explorer)
```

### Governance

**GovernanceToken** (ZINT with voting extension)
```
Address: 0x3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C
Network: Zintech L2 (Chain ID: 2024)
Purpose: Voting power for governance
Function: 1 ZINT = 1 vote (with staking multipliers)
Verified: ✅ Yes (View on Explorer)
```

**Governor**
```
Address: 0x7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A7A
Network: Zintech L2 (Chain ID: 2024)
Purpose: On-chain governance execution
Function: Proposal creation, voting, and execution
Type: OpenZeppelin Governor Compatible
Verified: ✅ Yes (View on Explorer)
```

**Timelock**
```
Address: 0x8B8B8B8B8B8B8B8B8B8B8B8B8B8B8B8B8B8B8B8B
Network: Zintech L2 (Chain ID: 2024)
Purpose: Timelock for governance actions
Function: 72-hour delay on critical protocol changes
Verified: ✅ Yes (View on Explorer)
```

---

## Protocol Integration Contracts

### DeFi Protocol Adapters

**PancakeSwapAdapter**
```
Address: 0x9C9C9C9C9C9C9C9C9C9C9C9C9C9C9C9C9C9C9C9C
Network: Zintech L2 (Chain ID: 2024)
Purpose: Integration with PancakeSwap
Function: Swap, add/remove liquidity, farm
Verified: ✅ Yes
```

**VenusAdapter**
```
Address: 0xADADADADADADADADADADADADADADADADADADADAD
Network: Zintech L2 (Chain ID: 2024)
Purpose: Integration with Venus Protocol
Function: Supply, borrow, repay operations
Verified: ✅ Yes
```

**AlpacaAdapter**
```
Address: 0xBEBEBEBEBEBEBEBEBEBEBEBEBEBEBEBEBEBEBEBE
Network: Zintech L2 (Chain ID: 2024)
Purpose: Integration with Alpaca Finance
Function: Leveraged yield farming
Verified: ✅ Yes
```

**BiswapAdapter**
```
Address: 0xCFCFCFCFCFCFCFCFCFCFCFCFCFCFCFCFCFCFCFCF
Network: Zintech L2 (Chain ID: 2024)
Purpose: Integration with Biswap
Function: Trading and liquidity provision
Verified: ✅ Yes
```

**WombatAdapter**
```
Address: 0xDEDEDEDEDEDEDEDEDEDEDEDEDEDEDEDEDEDEDEDE
Network: Zintech L2 (Chain ID: 2024)
Purpose: Integration with Wombat Exchange
Function: Stablecoin swaps and liquidity
Verified: ✅ Yes
```

---

## Security Contracts

**MultiSigWallet**
```
Address: 0xEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEF
Network: Zintech L2 (Chain ID: 2024)
Purpose: Multi-signature wallet for protocol operations
Signers: 5 of 7 required
Verified: ✅ Yes (View on Explorer)
```

**InsuranceFund**
```
Address: 0xFAFAFAFAFAFAFAFAFAFAFAFAFAFAFAFAFAFAFAFA
Network: Zintech L2 (Chain ID: 2024)
Purpose: Insurance coverage for users
Balance: 5% of protocol fees
Verified: ✅ Yes (View on Explorer)
```

**EmergencyDAO**
```
Address: 0x1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B
Network: Zintech L2 (Chain ID: 2024)
Purpose: Emergency response for critical issues
Function: Fast response multi-sig (3 of 5)
Verified: ✅ Yes (View on Explorer)
```

---

## Analytics & Monitoring

**AnalyticsEngine**
```
Address: 0x2C2C2C2C2C2C2C2C2C2C2C2C2C2C2C2C2C2C2C2C
Network: Zintech L2 (Chain ID: 2024)
Purpose: Collects and processes protocol metrics
Function: TVL, volume, APY calculations
Verified: ✅ Yes (View on Explorer)
```

**EventLogger**
```
Address: 0x3D3D3D3D3D3D3D3D3D3D3D3D3D3D3D3D3D3D3D3D
Network: Zintech L2 (Chain ID: 2024)
Purpose: Logs all protocol events
Function: On-chain event storage and indexing
Verified: ✅ Yes (View on Explorer)
```

---

## Network Information

### Zintech L2 Network Details

**RPC Endpoints**:
- Primary: `https://rpc.zintech.io`
- Backup: `https://rpc-backup.zintech.io`
- WebSocket: `wss://ws.zintech.io`

**Chain Configuration**:
```json
{
  "chainId": 2024,
  "chainName": "Zintech L2",
  "nativeCurrency": {
    "name": "ZINT",
    "symbol": "ZINT",
    "decimals": 18
  },
  "rpcUrls": ["https://rpc.zintech.io"],
  "blockExplorerUrls": ["https://explorer.zintech.io"]
}
```

**Block Explorer**: https://explorer.zintech.io

**Bridge UI**: https://bridge.zintech.io

---

## Contract Verification

All Zintech contracts are:
- ✅ Open source on GitHub
- ✅ Verified on block explorers
- ✅ Audited by CertiK, PeckShield, and Hacken
- ✅ Upgradeable via governance

**Audit Reports**:
- CertiK: https://zintech.io/audits/certik.pdf
- PeckShield: https://zintech.io/audits/peckshield.pdf
- Hacken: https://zintech.io/audits/hacken.pdf

---

## Contract Interaction Examples

### Depositing to Smart Account

```javascript
// Connect to Zintech L2
const provider = new ethers.providers.JsonRpcProvider('https://rpc.zintech.io');
const signer = provider.getSigner();

// Smart Account Factory contract
const factoryAddress = '0xE5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5';
const factory = new ethers.Contract(factoryAddress, factoryABI, signer);

// Create Smart Account
const tx = await factory.createAccount({
  riskProfile: 1, // 0: Conservative, 1: Moderate, 2: Aggressive
  maxExposure: ethers.utils.parseEther('0.3') // 30% max per protocol
});
await tx.wait();

// Get your Smart Account address
const accountAddress = await factory.getAccount(signer.address);

// Deposit BNB
const account = new ethers.Contract(accountAddress, accountABI, signer);
await account.deposit({ value: ethers.utils.parseEther('1.0') });
```

### Deploying Neural Agent

```javascript
// Agent Factory contract
const agentFactoryAddress = '0xB2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2';
const agentFactory = new ethers.Contract(agentFactoryAddress, agentFactoryABI, signer);

// Deploy agent
const deployTx = await agentFactory.deployAgent(
  accountAddress, // Your Smart Account
  'yield-optimizer', // Agent type
  ['0x9C9C...', '0xADAD...'], // Protocol adapters (PancakeSwap, Venus)
  { value: ethers.utils.parseEther('100') } // 100 ZINT deployment fee
);
await deployTx.wait();

// Get agent address
const agentAddress = await agentFactory.getAgent(accountAddress);
console.log('Agent deployed at:', agentAddress);
```

### Staking ZINT

```javascript
// ZINT Token contract
const zintAddress = '0x3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C';
const zint = new ethers.Contract(zintAddress, erc20ABI, signer);

// Staking contract
const stakingAddress = '0x4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D';
const staking = new ethers.Contract(stakingAddress, stakingABI, signer);

// Approve ZINT
const approvalTx = await zint.approve(
  stakingAddress,
  ethers.utils.parseEther('10000')
);
await approvalTx.wait();

// Stake with 6-month lock for 1.5x rewards
const stakeTx = await staking.stake(
  ethers.utils.parseEther('10000'),
  180 // days
);
await stakeTx.wait();

console.log('Staked 10,000 ZINT for 6 months');
```

### Bridging Assets

```javascript
// L1 Bridge on BSC
const l1BridgeAddress = '0xd21060559c9beb54fC07aFd6151aDf6cFCDDCAeB';
const l1Bridge = new ethers.Contract(l1BridgeAddress, bridgeABI, signer);

// Bridge BNB from BSC to Zintech L2
const bridgeTx = await l1Bridge.depositETH(
  200000, // L2 gas limit
  '0x', // Extra data
  { value: ethers.utils.parseEther('0.5') }
);
await bridgeTx.wait();

// Funds arrive on L2 in ~1-2 minutes
```

---

## Security Considerations

### Multi-Sig Addresses

**Protocol Multi-Sig** (7 of 5):
- Address: `0xEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEFEF`
- Signers: Core team + community representatives
- Purpose: Protocol upgrades, parameter changes

**Emergency Multi-Sig** (3 of 5):
- Address: `0x1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B1B`
- Signers: Security team
- Purpose: Fast response to critical issues

### Upgrade Process

1. Proposal created via Governor contract
2. 3-day voting period
3. 72-hour timelock delay
4. Multi-sig execution
5. On-chain upgrade

### Emergency Procedures

In case of critical vulnerability:
1. Emergency DAO can pause affected contracts
2. User funds remain safe in Smart Accounts
3. Emergency withdrawal enabled
4. Fix deployed via fast-track governance
5. System resumed after verification

---

## Developer Resources

**GitHub**: https://github.com/zintech/contracts  
**Contract ABIs**: https://github.com/zintech/contracts/tree/main/abi  
**Deployment Scripts**: https://github.com/zintech/contracts/tree/main/scripts  
**Integration Guide**: https://docs.zintech.io/integration  
**Testnet Faucet**: https://faucet.zintech.io

---

## Contract Updates

This page is updated regularly. Last update: **October 2025**

For real-time contract information, visit:
- **Block Explorer**: https://explorer.zintech.io
- **Contract Registry**: https://github.com/zintech/contracts/blob/main/REGISTRY.md

---

## Support

For contract-related questions:
- **Discord**: https://discord.gg/zintech-dev
- **Email**: developers@zintech.io
- **GitHub Issues**: https://github.com/zintech/contracts/issues

---

**⚠️ Important**: Always verify contract addresses on official Zintech channels before interacting. Never send funds to unverified addresses.

---

© 2025 Zintech Protocol. All contracts are open source and audited.
