# Metacrafters Avalanche Advanced Subnet

Welcome to the exciting world of blockchain-based gaming! This project explores the development of a **DeFi Kingdom clone** on the Avalanche network. Players can collect, build, and battle with digital assets, all while enjoying engaging gameplay mechanics and earning rewards.

While a deep understanding of DeFi is essential, this guide focuses on the technical foundation: setting up an EVM Subnet on Avalanche and deploying the core smart contracts required for the game. Our journey will involve the following key steps:

---

## Project Roadmap

### 1. Set Up an EVM Subnet
Use the **Avalanche CLI** and the Avalanche documentation to create a custom EVM subnet on the Avalanche network.
- Define your native currency to serve as the in-game currency.

### 2. Connect to MetaMask
Seamlessly connect your custom EVM Subnet to MetaMask for ease of use.

### 3. Deploy Core Smart Contracts
Using **Solidity** and **Remix**, deploy the foundational contracts for battling, exploring, and trading. These smart contracts will:
- Define game rules.
- Set up liquidity pools.
- Manage tokens and game mechanics.

---

## Avalanche Subnet Overview
An **Avalanche Subnet** is an independent, customizable blockchain network within the Avalanche ecosystem. Subnets allow developers to:
- Tailor consensus mechanisms and tokenomics.
- Create scalable, specialized environments for their applications.

### Setting Up Your Subnet
1. Install the latest Avalanche CLI binary:
   ```bash
   curl -sSfL https://raw.githubusercontent.com/ava-labs/avalanche-cli/main/scripts/install.sh | sh -s
   ```

2. Create an EVM-based Subnet using **Subnet-EVM**, Avalanche's fork of the Ethereum Virtual Machine:
   - Supports features like airdrops, custom fee tokens, and configurable gas parameters.
   - Command to create a new Subnet:
     ```bash
     avalanche subnet create mySubnet
     ```
   - Configure the following:
     - **Chain ID:** Any positive integer (e.g., `12345567`)
     - **Native Token Symbol:** Define your in-game token (e.g., `MYSUBNET`)

3. Deploy the Subnet:
   ```bash
   avalanche subnet deploy mySubnet
   ```
   - Deploy your Subnet locally and retrieve the details for interaction.

### Subnet-EVM Documentation
Learn more about Subnet-EVM on the [official repository](https://github.com/ava-labs/subnet-evm).

---

## Smart Contract Highlights

### ERC20 Contract
This contract implements the ERC20 token standard to create fungible tokens for your game. Below are its key components:

#### Variables
- **totalSupply:** Tracks the total supply of tokens.
- **balanceOf:** Maps each address to its balance.
- **allowance:** Manages approved token spending.
- **name, symbol, decimals:** Token metadata.

#### Events
- **Transfer:** Emitted during token transfers.
- **Approval:** Emitted when an address approves another to spend tokens.

#### Functions
- **transfer:** Transfers tokens between addresses.
- **approve:** Grants spending rights to another address.
- **transferFrom:** Executes transfers using approved allowances.
- **mint:** Generates new tokens.
- **burn:** Destroys tokens from the sender’s balance.

---

### Vault Contract
The **Vault Contract** manages deposits and withdrawals of ERC20 tokens, providing a simple system for users to exchange tokens for shares.

#### Components
- **IERC20 Interface:** Standard interface for interacting with ERC20 tokens.
- **token:** Immutable reference to the ERC20 token.
- **totalSupply, balanceOf:** Tracks shares within the vault.

#### Functions
- **deposit:**
  - Allows users to deposit ERC20 tokens in exchange for shares.
  - Calculates shares based on the proportion of tokens deposited relative to the vault’s balance.
- **withdraw:**
  - Burns shares and returns the equivalent amount of ERC20 tokens.

---

## Example Code Snippets

### ERC20 Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "GameToken";
    string public symbol = "GTKN";
    uint8 public decimals = 18;

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address recipient, uint amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(msg.sender, recipient, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(address sender, address recipient, uint amount) external returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[recipient] += amount;
        emit Transfer(sender, recipient, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```

### Vault Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function transfer(address recipient, uint amount) external returns (bool);
    function transferFrom(address sender, address recipient, uint amount) external returns (bool);
    function balanceOf(address account) external view returns (uint);
}

contract Vault {
    IERC20 public immutable token;
    uint public totalSupply;
    mapping(address => uint) public balanceOf;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function deposit(uint _amount) external {
        uint shares = (totalSupply == 0)
            ? _amount
            : (_amount * totalSupply) / token.balanceOf(address(this));

        balanceOf[msg.sender] += shares;
        totalSupply += shares;
        token.transferFrom(msg.sender, address(this), _amount);
    }

    function withdraw(uint _shares) external {
        uint amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        balanceOf[msg.sender] -= _shares;
        totalSupply -= _shares;
        token.transfer(msg.sender, amount);
    }
}
```

---

## Resources
- [Avalanche Documentation](https://docs.avax.network/)
- [Subnet-EVM Repository](https://github.com/ava-labs/subnet-evm)

Embark on this journey to create your very own blockchain-based DeFi Kingdom clone and explore the endless possibilities of decentralized gaming!
