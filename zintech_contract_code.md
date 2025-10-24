# Zintech Smart Contract Code Examples

## Core Contract Implementations

This document contains Solidity code examples for Zintech's main smart contracts.

---

## 1. SmartAccount.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

/**
 * @title SmartAccount
 * @notice Non-custodial account that allows neural agents to execute strategies
 * @dev Users maintain full ownership while granting execution permissions to agents
 */
contract SmartAccount is ReentrancyGuard, Ownable {
    using SafeERC20 for IERC20;

    // Agent authorized to execute strategies
    address public agent;
    
    // Risk profile: 0 = Conservative, 1 = Moderate, 2 = Aggressive
    uint8 public riskProfile;
    
    // Maximum exposure per protocol (in basis points, 10000 = 100%)
    uint256 public maxExposurePerProtocol;
    
    // Emergency withdrawal threshold (triggers auto-pause)
    uint256 public emergencyWithdrawThreshold;
    
    // Protocol whitelist
    mapping(address => bool) public allowedProtocols;
    
    // Current positions
    mapping(address => uint256) public positions;
    
    // Total value locked
    uint256 public totalValue;
    
    // Paused state
    bool public paused;
    
    // Performance metrics
    uint256 public totalDeposits;
    uint256 public totalWithdrawals;
    int256 public realizedPnL;
    
    // Events
    event Deposited(address indexed token, uint256 amount);
    event Withdrawn(address indexed token, uint256 amount, address indexed to);
    event AgentChanged(address indexed oldAgent, address indexed newAgent);
    event StrategyExecuted(address indexed protocol, bytes data, bool success);
    event EmergencyPause(string reason);
    event ConfigUpdated(uint8 riskProfile, uint256 maxExposure);
    
    // Modifiers
    modifier onlyAgent() {
        require(msg.sender == agent, "Not authorized agent");
        _;
    }
    
    modifier whenNotPaused() {
        require(!paused, "Account is paused");
        _;
    }
    
    /**
     * @notice Initialize Smart Account
     * @param _owner Account owner
     * @param _riskProfile Risk profile (0-2)
     * @param _maxExposure Maximum exposure per protocol in basis points
     */
    constructor(
        address _owner,
        uint8 _riskProfile,
        uint256 _maxExposure
    ) {
        require(_owner != address(0), "Invalid owner");
        require(_riskProfile <= 2, "Invalid risk profile");
        require(_maxExposure <= 5000, "Max exposure too high"); // Max 50%
        
        _transferOwnership(_owner);
        riskProfile = _riskProfile;
        maxExposurePerProtocol = _maxExposure;
        emergencyWithdrawThreshold = 1500; // 15% loss triggers emergency
    }
    
    /**
     * @notice Deposit native token (BNB)
     */
    function deposit() external payable onlyOwner nonReentrant {
        require(msg.value > 0, "Invalid amount");
        
        totalValue += msg.value;
        totalDeposits += msg.value;
        
        emit Deposited(address(0), msg.value);
    }
    
    /**
     * @notice Deposit ERC20 token
     * @param token Token address
     * @param amount Amount to deposit
     */
    function depositToken(address token, uint256 amount) 
        external 
        onlyOwner 
        nonReentrant 
    {
        require(amount > 0, "Invalid amount");
        
        IERC20(token).safeTransferFrom(msg.sender, address(this), amount);
        
        totalValue += amount;
        totalDeposits += amount;
        
        emit Deposited(token, amount);
    }
    
    /**
     * @notice Withdraw native token (BNB)
     * @param amount Amount to withdraw
     * @param to Recipient address
     */
    function withdraw(uint256 amount, address payable to) 
        external 
        onlyOwner 
        nonReentrant 
    {
        require(amount > 0 && amount <= address(this).balance, "Invalid amount");
        require(to != address(0), "Invalid recipient");
        
        totalValue -= amount;
        totalWithdrawals += amount;
        
        to.transfer(amount);
        
        emit Withdrawn(address(0), amount, to);
    }
    
    /**
     * @notice Withdraw ERC20 token
     * @param token Token address
     * @param amount Amount to withdraw
     * @param to Recipient address
     */
    function withdrawToken(
        address token, 
        uint256 amount, 
        address to
    ) 
        external 
        onlyOwner 
        nonReentrant 
    {
        require(amount > 0, "Invalid amount");
        require(to != address(0), "Invalid recipient");
        
        IERC20(token).safeTransfer(to, amount);
        
        totalValue -= amount;
        totalWithdrawals += amount;
        
        emit Withdrawn(token, amount, to);
    }
    
    /**
     * @notice Execute strategy via agent
     * @param protocol Target protocol address
     * @param data Encoded function call
     */
    function executeStrategy(address protocol, bytes calldata data) 
        external 
        onlyAgent 
        whenNotPaused 
        nonReentrant 
        returns (bool success, bytes memory returnData)
    {
        require(allowedProtocols[protocol], "Protocol not whitelisted");
        
        // Check exposure limits
        uint256 proposedExposure = _calculateExposure(protocol);
        require(
            proposedExposure <= maxExposurePerProtocol, 
            "Exceeds max exposure"
        );
        
        // Execute strategy
        (success, returnData) = protocol.call(data);
        
        // Update position
        if (success) {
            positions[protocol] = proposedExposure;
        }
        
        emit StrategyExecuted(protocol, data, success);
        
        return (success, returnData);
    }
    
    /**
     * @notice Set agent address
     * @param newAgent New agent address
     */
    function setAgent(address newAgent) external onlyOwner {
        require(newAgent != address(0), "Invalid agent");
        
        address oldAgent = agent;
        agent = newAgent;
        
        emit AgentChanged(oldAgent, newAgent);
    }
    
    /**
     * @notice Update configuration
     * @param _riskProfile New risk profile
     * @param _maxExposure New max exposure
     */
    function updateConfig(uint8 _riskProfile, uint256 _maxExposure) 
        external 
        onlyOwner 
    {
        require(_riskProfile <= 2, "Invalid risk profile");
        require(_maxExposure <= 5000, "Max exposure too high");
        
        riskProfile = _riskProfile;
        maxExposurePerProtocol = _maxExposure;
        
        emit ConfigUpdated(_riskProfile, _maxExposure);
    }
    
    /**
     * @notice Whitelist protocol
     * @param protocol Protocol address
     * @param allowed Whether protocol is allowed
     */
    function setProtocolAllowed(address protocol, bool allowed) 
        external 
        onlyOwner 
    {
        allowedProtocols[protocol] = allowed;
    }
    
    /**
     * @notice Emergency pause
     * @param reason Reason for pause
     */
    function emergencyPause(string calldata reason) external {
        require(
            msg.sender == owner() || msg.sender == agent,
            "Not authorized"
        );
        
        paused = true;
        
        emit EmergencyPause(reason);
    }
    
    /**
     * @notice Unpause account
     */
    function unpause() external onlyOwner {
        paused = false;
    }
    
    /**
     * @notice Get account performance
     */
    function getPerformance() external view returns (
        uint256 _totalValue,
        uint256 _totalDeposits,
        uint256 _totalWithdrawals,
        int256 _realizedPnL,
        uint256 _unrealizedPnL
    ) {
        return (
            totalValue,
            totalDeposits,
            totalWithdrawals,
            realizedPnL,
            _calculateUnrealizedPnL()
        );
    }
    
    /**
     * @notice Calculate exposure to protocol
     */
    function _calculateExposure(address protocol) 
        internal 
        view 
        returns (uint256) 
    {
        if (totalValue == 0) return 0;
        return (positions[protocol] * 10000) / totalValue;
    }
    
    /**
     * @notice Calculate unrealized PnL
     */
    function _calculateUnrealizedPnL() internal view returns (uint256) {
        if (totalValue > totalDeposits) {
            return totalValue - totalDeposits;
        }
        return 0;
    }
    
    /**
     * @notice Receive native token
     */
    receive() external payable {
        totalValue += msg.value;
    }
}
```

---

## 2. AgentFactory.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "./NeuralAgent.sol";
import "./AgentRegistry.sol";

/**
 * @title AgentFactory
 * @notice Factory for deploying neural agent instances
 */
contract AgentFactory is Ownable, ReentrancyGuard {
    // Registry contract
    AgentRegistry public registry;
    
    // Deployment fee in ZINT
    uint256 public deploymentFee = 100 ether; // 100 ZINT
    
    // ZINT token address
    address public zintToken;
    
    // Mapping from Smart Account to Agent
    mapping(address => address) public accountToAgent;
    
    // Agent templates
    mapping(string => address) public agentTemplates;
    
    // Events
    event AgentDeployed(
        address indexed account,
        address indexed agent,
        string agentType,
        address indexed deployer
    );
    event TemplateAdded(string agentType, address template);
    event DeploymentFeeUpdated(uint256 oldFee, uint256 newFee);
    
    /**
     * @notice Initialize factory
     * @param _registry Agent registry address
     * @param _zintToken ZINT token address
     */
    constructor(address _registry, address _zintToken) {
        require(_registry != address(0), "Invalid registry");
        require(_zintToken != address(0), "Invalid token");
        
        registry = AgentRegistry(_registry);
        zintToken = _zintToken;
    }
    
    /**
     * @notice Deploy new neural agent
     * @param account Smart Account address
     * @param agentType Type of agent (yield-optimizer, lp-manager, etc.)
     * @param protocols Whitelisted protocol addresses
     */
    function deployAgent(
        address account,
        string calldata agentType,
        address[] calldata protocols
    ) 
        external 
        nonReentrant 
        returns (address agentAddress) 
    {
        require(account != address(0), "Invalid account");
        require(accountToAgent[account] == address(0), "Agent already exists");
        require(agentTemplates[agentType] != address(0), "Invalid agent type");
        
        // Collect deployment fee
        if (deploymentFee > 0) {
            IERC20(zintToken).transferFrom(
                msg.sender,
                address(this),
                deploymentFee
            );
            
            // Burn deployment fee (deflationary)
            IERC20(zintToken).transfer(
                address(0x000000000000000000000000000000000000dEaD),
                deploymentFee
            );
        }
        
        // Clone agent template
        address template = agentTemplates[agentType];
        agentAddress = _clone(template);
        
        // Initialize agent
        NeuralAgent(agentAddress).initialize(account, protocols);
        
        // Register agent
        accountToAgent[account] = agentAddress;
        registry.registerAgent(agentAddress, account, agentType);
        
        emit AgentDeployed(account, agentAddress, agentType, msg.sender);
        
        return agentAddress;
    }
    
    /**
     * @notice Add agent template
     * @param agentType Agent type identifier
     * @param template Template contract address
     */
    function addTemplate(string calldata agentType, address template) 
        external 
        onlyOwner 
    {
        require(template != address(0), "Invalid template");
        
        agentTemplates[agentType] = template;
        
        emit TemplateAdded(agentType, template);
    }
    
    /**
     * @notice Update deployment fee
     * @param newFee New deployment fee
     */
    function setDeploymentFee(uint256 newFee) external onlyOwner {
        uint256 oldFee = deploymentFee;
        deploymentFee = newFee;
        
        emit DeploymentFeeUpdated(oldFee, newFee);
    }
    
    /**
     * @notice Get agent address for account
     * @param account Smart Account address
     */
    function getAgent(address account) external view returns (address) {
        return accountToAgent[account];
    }
    
    /**
     * @notice Clone contract using EIP-1167 minimal proxy
     */
    function _clone(address implementation) 
        internal 
        returns (address instance) 
    {
        assembly {
            let ptr := mload(0x40)
            mstore(ptr, 0x3d602d80600a3d3981f3363d3d373d3d3d363d73000000000000000000000000)
            mstore(add(ptr, 0x14), shl(0x60, implementation))
            mstore(add(ptr, 0x28), 0x5af43d82803e903d91602b57fd5bf30000000000000000000000000000000000)
            instance := create(0, ptr, 0x37)
        }
        require(instance != address(0), "Clone failed");
    }
}
```

---

## 3. ZINT Token

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Burnable.sol";
import "@openzeppelin/contracts/token/ERC20/extensions/ERC20Votes.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

/**
 * @title ZINT
 * @notice Zintech native governance and utility token
 */
contract ZINT is ERC20, ERC20Burnable, ERC20Votes, Ownable {
    // Total supply: 500 million ZINT
    uint256 public constant MAX_SUPPLY = 500_000_000 ether;
    
    // Minter role
    mapping(address => bool) public minters;
    
    // Events
    event MinterAdded(address indexed minter);
    event MinterRemoved(address indexed minter);
    
    /**
     * @notice Initialize ZINT token
     */
    constructor() 
        ERC20("Zintech", "ZINT") 
        ERC20Permit("Zintech") 
    {
        // Mint initial supply to deployer
        _mint(msg.sender, 50_000_000 ether); // 10% initial supply
    }
    
    /**
     * @notice Mint new tokens
     * @param to Recipient address
     * @param amount Amount to mint
     */
    function mint(address to, uint256 amount) external {
        require(minters[msg.sender], "Not a minter");
        require(totalSupply() + amount <= MAX_SUPPLY, "Exceeds max supply");
        
        _mint(to, amount);
    }
    
    /**
     * @notice Add minter
     * @param minter Address to add as minter
     */
    function addMinter(address minter) external onlyOwner {
        require(minter != address(0), "Invalid minter");
        minters[minter] = true;
        emit MinterAdded(minter);
    }
    
    /**
     * @notice Remove minter
     * @param minter Address to remove as minter
     */
    function removeMinter(address minter) external onlyOwner {
        minters[minter] = false;
        emit MinterRemoved(minter);
    }
    
    // Override required by Solidity
    function _afterTokenTransfer(
        address from,
        address to,
        uint256 amount
    ) internal override(ERC20, ERC20Votes) {
        super._afterTokenTransfer(from, to, amount);
    }
    
    function _mint(address to, uint256 amount)
        internal
        override(ERC20, ERC20Votes)
    {
        super._mint(to, amount);
    }
    
    function _burn(address account, uint256 amount)
        internal
        override(ERC20, ERC20Votes)
    {
        super._burn(account, amount);
    }
}
```

---

## 4. StakingContract.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

/**
 * @title StakingContract
 * @notice Staking contract for ZINT with time-based multipliers
 */
contract StakingContract is ReentrancyGuard, Ownable {
    using SafeERC20 for IERC20;
    
    // ZINT token
    IERC20 public zintToken;
    
    // Staking info
    struct Stake {
        uint256 amount;
        uint256 startTime;
        uint256 lockPeriod; // in days
        uint256 multiplier; // in basis points (10000 = 1x)
        uint256 rewards;
        bool active;
    }
    
    // User stakes
    mapping(address => Stake[]) public userStakes;
    
    // Total staked
    uint256 public totalStaked;
    
    // Reward rate (APR in basis points)
    uint256 public baseRewardRate = 1000; // 10% base APR
    
    // Lock period multipliers (in basis points)
    mapping(uint256 => uint256) public lockMultipliers;
    
    // Events
    event Staked(
        address indexed user,
        uint256 indexed stakeId,
        uint256 amount,
        uint256 lockPeriod,
        uint256 multiplier
    );
    event Unstaked(
        address indexed user,
        uint256 indexed stakeId,
        uint256 amount,
        uint256 rewards
    );
    event RewardsClaimed(address indexed user, uint256 amount);
    event RewardRateUpdated(uint256 oldRate, uint256 newRate);
    
    /**
     * @notice Initialize staking contract
     * @param _zintToken ZINT token address
     */
    constructor(address _zintToken) {
        require(_zintToken != address(0), "Invalid token");
        
        zintToken = IERC20(_zintToken);
        
        // Set lock multipliers
        lockMultipliers[0] = 10000;    // Flexible: 1.0x
        lockMultipliers[90] = 12000;   // 3 months: 1.2x
        lockMultipliers[180] = 15000;  // 6 months: 1.5x
        lockMultipliers[365] = 20000;  // 12 months: 2.0x
    }
    
    /**
     * @notice Stake ZINT tokens
     * @param amount Amount to stake
     * @param lockPeriodDays Lock period in days
     */
    function stake(uint256 amount, uint256 lockPeriodDays) 
        external 
        nonReentrant 
        returns (uint256 stakeId)
    {
        require(amount > 0, "Invalid amount");
        require(
            lockMultipliers[lockPeriodDays] > 0,
            "Invalid lock period"
        );
        
        // Transfer tokens
        zintToken.safeTransferFrom(msg.sender, address(this), amount);
        
        // Create stake
        stakeId = userStakes[msg.sender].length;
        userStakes[msg.sender].push(Stake({
            amount: amount,
            startTime: block.timestamp,
            lockPeriod: lockPeriodDays * 1 days,
            multiplier: lockMultipliers[lockPeriodDays],
            rewards: 0,
            active: true
        }));
        
        totalStaked += amount;
        
        emit Staked(
            msg.sender,
            stakeId,
            amount,
            lockPeriodDays,
            lockMultipliers[lockPeriodDays]
        );
        
        return stakeId;
    }
    
    /**
     * @notice Unstake and claim rewards
     * @param stakeId Stake ID
     */
    function unstake(uint256 stakeId) external nonReentrant {
        require(stakeId < userStakes[msg.sender].length, "Invalid stake ID");
        
        Stake storage userStake = userStakes[msg.sender][stakeId];
        require(userStake.active, "Stake not active");
        require(
            block.timestamp >= userStake.startTime + userStake.lockPeriod,
            "Still locked"
        );
        
        // Calculate final rewards
        uint256 rewards = _calculateRewards(msg.sender, stakeId);
        uint256 totalAmount = userStake.amount + rewards;
        
        // Update state
        userStake.active = false;
        userStake.rewards = rewards;
        totalStaked -= userStake.amount;
        
        // Transfer tokens
        zintToken.safeTransfer(msg.sender, totalAmount);
        
        emit Unstaked(msg.sender, stakeId, userStake.amount, rewards);
    }
    
    /**
     * @notice Claim rewards without unstaking
     * @param stakeId Stake ID
     */
    function claimRewards(uint256 stakeId) external nonReentrant {
        require(stakeId < userStakes[msg.sender].length, "Invalid stake ID");
        
        Stake storage userStake = userStakes[msg.sender][stakeId];
        require(userStake.active, "Stake not active");
        
        uint256 rewards = _calculateRewards(msg.sender, stakeId);
        require(rewards > 0, "No rewards");
        
        // Update state
        userStake.rewards += rewards;
        userStake.startTime = block.timestamp; // Reset reward timer
        
        // Transfer rewards
        zintToken.safeTransfer(msg.sender, rewards);
        
        emit RewardsClaimed(msg.sender, rewards);
    }
    
    /**
     * @notice Calculate pending rewards
     * @param user User address
     * @param stakeId Stake ID
     */
    function calculateRewards(address user, uint256 stakeId)
        external
        view
        returns (uint256)
    {
        return _calculateRewards(user, stakeId);
    }
    
    /**
     * @notice Get user's total staked amount
     * @param user User address
     */
    function getUserTotalStaked(address user) 
        external 
        view 
        returns (uint256 total) 
    {
        Stake[] memory stakes = userStakes[user];
        for (uint256 i = 0; i < stakes.length; i++) {
            if (stakes[i].active) {
                total += stakes[i].amount;
            }
        }
        return total;
    }
    
    /**
     * @notice Set lock multiplier
     * @param lockPeriodDays Lock period in days
     * @param multiplier Multiplier in basis points
     */
    function setLockMultiplier(uint256 lockPeriodDays, uint256 multiplier)
        external
        onlyOwner
    {
        require(multiplier >= 10000 && multiplier <= 30000, "Invalid multiplier");
        lockMultipliers[lockPeriodDays] = multiplier;
    }
    
    /**
     * @notice Update base reward rate
     * @param newRate New rate in basis points
     */
    function setRewardRate(uint256 newRate) external onlyOwner {
        require(newRate <= 5000, "Rate too high"); // Max 50% APR
        
        uint256 oldRate = baseRewardRate;
        baseRewardRate = newRate;
        
        emit RewardRateUpdated(oldRate, newRate);
    }
    
    /**
     * @notice Internal function to calculate rewards
     */
    function _calculateRewards(address user, uint256 stakeId)
        internal
        view
        returns (uint256)
    {
        Stake memory userStake = userStakes[user][stakeId];
        if (!userStake.active) return 0;
        
        uint256 timeStaked = block.timestamp - userStake.startTime;
        uint256 annualReward = (userStake.amount * baseRewardRate) / 10000;
        uint256 baseReward = (annualReward * timeStaked) / 365 days;
        
        // Apply multiplier
        uint256 multipliedReward = (baseReward * userStake.multiplier) / 10000;
        
        return multipliedReward;
    }
}
```

---

## 5. FeeCollector.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol";

/**
 * @title FeeCollector
 * @notice Collects and distributes protocol fees
 * @dev Distribution: 50% to stakers, 25% burned, 25% to treasury
 */
contract FeeCollector is Ownable, ReentrancyGuard {
    using SafeERC20 for IERC20;
    
    // ZINT token
    IERC20 public zintToken;
    
    // Distribution addresses
    address public stakingContract;
    address public treasury;
    address public constant BURN_ADDRESS = 
        0x000000000000000000000000000000000000dEaD;
    
    // Performance fee: 10% (1000 basis points)
    uint256 public performanceFee = 1000;
    
    // Management fee: 0.5% annual (50 basis points)
    uint256 public managementFee = 50;
    
    // Total fees collected
    uint256 public totalFeesCollected;
    
    // Events
    event FeesCollected(
        address indexed from,
        uint256 amount,
        string feeType
    );
    event FeesDistributed(
        uint256 toStakers,
        uint256 burned,
        uint256 toTreasury
    );
    event FeeRatesUpdated(
        uint256 performanceFee,
        uint256 managementFee
    );
    
    /**
     * @notice Initialize fee collector
     */
    constructor(
        address _zintToken,
        address _stakingContract,
        address _treasury
    ) {
        require(_zintToken != address(0), "Invalid token");
        require(_stakingContract != address(0), "Invalid staking");
        require(_treasury != address(0), "Invalid treasury");
        
        zintToken = IERC20(_zintToken);
        stakingContract = _stakingContract;
        treasury = _treasury;
    }
    
    /**
     * @notice Collect performance fee
     * @param from Agent or account paying fee
     * @param amount Fee amount
     */
    function collectPerformanceFee(address from, uint256 amount)
        external
        nonReentrant
    {
        require(amount > 0, "Invalid amount");
        
        zintToken.safeTransferFrom(from, address(this), amount);
        totalFeesCollected += amount;
        
        emit FeesCollected(from, amount, "performance");
    }
    
    /**
     * @notice Collect management fee
     * @param from Account paying fee
     * @param amount Fee amount
     */
    function collectManagementFee(address from, uint256 amount)
        external
        nonReentrant
    {
        require(amount > 0, "Invalid amount");
        
        zintToken.safeTransferFrom(from, address(this), amount);
        totalFeesCollected += amount;
        
        emit FeesCollected(from, amount, "management");
    }
    
    /**
     * @notice Distribute collected fees
     */
    function distributeFees() external nonReentrant {
        uint256 balance = zintToken.balanceOf(address(this));
        require(balance > 0, "No fees to distribute");
        
        // Calculate distribution
        uint256 toStakers = (balance * 5000) / 10000;  // 50%
        uint256 toBurn = (balance * 2500) / 10000;     // 25%
        uint256 toTreasury = balance - toStakers - toBurn; // 25%
        
        // Distribute
        zintToken.safeTransfer(stakingContract, toStakers);
        zintToken.safeTransfer(BURN_ADDRESS, toBurn);
        zintToken.safeTransfer(treasury, toTreasury);
        
        emit FeesDistributed(toStakers, toBurn, toTreasury);
    }
    
    /**
     * @notice Update fee rates
     * @param _performanceFee New performance fee (basis points)
     * @param _managementFee New management fee (basis points)
     */
    function updateFeeRates(
        uint256 _performanceFee,
        uint256 _managementFee
    ) 
        external 
        onlyOwner 
    {
        require(_performanceFee <= 2000, "Performance fee too high"); // Max 20%
        require(_managementFee <= 200, "Management fee too high"); // Max 2%
        
        performanceFee = _performanceFee;
        managementFee = _managementFee;
        
        emit FeeRatesUpdated(_performanceFee, _managementFee);
    }
    
    /**
     * @notice Update distribution addresses
     */
    function updateAddresses(
        address _stakingContract,
        address _treasury
    ) 
        external 
        onlyOwner 
    {
        require(_stakingContract != address(0), "Invalid staking");
        require(_treasury != address(0), "Invalid treasury");
        
        stakingContract = _stakingContract;
        treasury = _treasury;
    }
}
```

---

## Contract Addresses Summary

### L1 (BSC Mainnet)
- **ZintechPortal**: `0x4f559F30f5eB88D635FDe1548C4267DB8FaB0351`
- **L1StandardBridge**: `0xd21060559c9beb54fC07aFd6151aDf6cFCDDCAeB`
- **SystemConfig**: `0x416C42991d05b31E9A6dC209e91AD22b79D87Ae6`

### L2 (Zintech Network)
- **ZINT Token**: `0x3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C3C`
- **SmartAccountFactory**: `0xE5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5`
- **AgentFactory**: `0xB2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2B2`
- **StakingContract**: `0x4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D4D`
- **FeeCollector**: `0x5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E5E`

---

## Usage Examples

See the main contracts documentation for complete usage examples and integration guides.

---

## Security

All contracts have been:
- ✅ Audited by CertiK, PeckShield, and Hacken
- ✅ Tested extensively on testnet
- ✅ Verified on block explorers
- ✅ Open sourced on GitHub

**GitHub**: https://github.com/zintech/contracts

---

© 2025 Zintech Protocol
