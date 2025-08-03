
# **Nodes Garden - Drosera Trap Setup for Cadet Role**  
#### **Install WSL (Ubuntu)**
1. Open **PowerShell as Admin** and run:
   ```powershell
   wsl --install
   ```
2. Restart your PC.

#### **Update System**

run your wsl and update dependecies
```bash
sudo apt update && sudo apt upgrade -y
```

### **1. Install Drosera Tools**  
```sh
# Install Drosera CLI  
curl -L https://app.drosera.io/install | bash  
source ~/.bashrc  
droseraup  

# Install Foundry (for Solidity)  
curl -L https://foundry.paradigm.xyz | bash  
source ~/.bashrc  
foundryup  

# Install Bun (JS runtime)  
curl -fsSL https://bun.sh/install | bash  
source ~/.bashrc  
```  

### **2. Initialize Trap**  
```sh
mkdir ~/my-drosera-trap && cd ~/my-drosera-trap
forge init -t drosera-network/trap-foundry-template  
bun install  
```  

### **3. Configure Trap for Node Garden**  
Edit `nano src/Trap.sol` (keep original response contract!):  
```solidity
it:

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract Trap is ITrap {
    // Updated response contract address
    address public constant RESPONSE_CONTRACT = 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608;
    string constant discordName = "YOURDISCORD"; // Replace with your Discord username

    function collect() external view returns (bytes memory) {
        bool active = IMockResponse(RESPONSE_CONTRACT).isActive();
        return abi.encode(active, discordName);
    }

    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        (bool active, string memory name) = abi.decode(data[0], (bool, string));
        if (!active || bytes(name).length == 0) {
            return (false, bytes(""));
        }

        return (true, abi.encode(name));
    }
}
```
Replace `YOURDISCORD` with your Discord username.  
To save: **Ctrl+X**, then **Y**, and **Enter**


Edit `nano drosera.toml`:  
```toml
ethereum_rpc = "https://ethereum-hoodi-rpc.publicnode.com"  
drosera_rpc = "https://relay.hoodi.drosera.io"  
eth_chain_id = 560048  
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"  

[traps.mytrap]  
path = "out/Trap.sol/Trap.json"  
response_contract = "0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608"
response_function = "respondWithDiscordName(string)"  
private_trap = true  
whitelist = ["YOUR_NODE_GARDEN_WALLET_ADDRESS"]
address = "NODES_GARDEN_TRAP_ADDRESS" #Provided by Node Garden check dashboard
```  

### **4. Deploy (Using Node Garden’s Key)**  
```sh
forge build  
drosera dryrun  # Test first  
```
```sh
DROSERA_PRIVATE_KEY="NODES_GARDEN_WALLET_PRIVATE_KEY" drosera apply  
```

### **5. Verify Cadet Role**  
```sh
cast call 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608 "isResponder(address)(bool)" YOUR_NODE_GARDEN_WALLET_ADDRESS --rpc-url https://ethereum-hoodi-rpc.publicnode.com  
```
Replace `YOUR_NODES_GARDEN_WALLET_ADDRESS` with your Nodes Green Wallet Address

If `true`, your Discord role will update shortly.  

---

