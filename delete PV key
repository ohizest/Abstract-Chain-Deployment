#!/bin/bash

curl -s https://raw.githubusercontent.com/zunxbt/logo/main/logo.sh | bash
sleep 3

BOLD=$(tput bold)
NORMAL=$(tput sgr0)
PINK='\033[1;35m'
YELLOW='\033[1;33m'

show() {
    case $2 in
        "error")
            echo -e "${PINK}${BOLD}❌ $1${NORMAL}"
            ;;
        "progress")
            echo -e "${PINK}${BOLD}⏳ $1${NORMAL}"
            ;;
        *)
            echo -e "${PINK}${BOLD}✅ $1${NORMAL}"
            ;;
    esac
}

install_dependencies() {
    show "Installing Node.js..." "progress"
    source <(wget -O - https://raw.githubusercontent.com/zunxbt/installation/main/node.sh)
    clear
    show "Initializing Hardhat project..." "progress"
    npx hardhat init --yes
    clear
    show "Installing required dependencies..." "progress"
    npm install -D @matterlabs/hardhat-zksync @matterlabs/zksync-contracts zksync-ethers@6 ethers@6
    
    show "All dependencies installation completed."
}

compilation() {
    show "Modifying Hardhat configuration..." "progress"
    cat <<EOL > hardhat.config.ts
import { HardhatUserConfig } from "hardhat/config";
import "@matterlabs/hardhat-zksync";

const config: HardhatUserConfig = {
  zksolc: {
    version: "latest",
    settings: {
      enableEraVMExtensions: false,
    },
  },
  defaultNetwork: "abstractTestnet",
  networks: {
    abstractTestnet: {
      url: "https://api.testnet.abs.xyz",
      ethNetwork: "sepolia",
      zksync: true,
      verifyURL: "https://api-explorer-verify.testnet.abs.xyz/contract_verification",
    },
  },
  solidity: {
    version: "0.8.24",
  },
};

export default config;
EOL
    mv contracts/Lock.sol contracts/HelloAbstract.sol
    cat <<EOL > contracts/HelloAbstract.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract HelloAbstract {
    function sayHello() public pure virtual returns (string memory) {
        return "Hey there, This smart contract is deployed with the help of @ZunXBT!";
    }
}
EOL

    npx hardhat clean
    npx hardhat compile --network abstractTestnet
}

deployment() {
    read -p "Enter your wallet private key (without 0x): " DEPLOYER_PRIVATE_KEY
    mkdir -p deploy
    cat <<EOL > deploy/deploy.ts
import { Wallet } from "zksync-ethers";
import { HardhatRuntimeEnvironment } from "hardhat/types";
import { Deployer } from "@matterlabs/hardhat-zksync";

export default async function (hre: HardhatRuntimeEnvironment) {
  const wallet = new Wallet("$DEPLOYER_PRIVATE_KEY");
  const deployer = new Deployer(hre, wallet);
  const artifact = await deployer.loadArtifact("HelloAbstract");

  const tokenContract = await deployer.deploy(artifact);
  console.log(\`Your deployed contract address : \${await tokenContract.getAddress()}\`);
}
EOL
}

deploy_contracts() {
    read -p "How many contracts do you want to deploy? " CONTRACT_COUNT
    > contracts.txt

    for ((i = 1; i <= CONTRACT_COUNT; i++)); do
        show "Deploying contract #$i..." "progress"
        npx hardhat deploy-zksync --script deploy/deploy.ts
        show "Contract #$i deployed successfully"
        echo "------------------------------------"
        read -p "Please enter the deployed contract address for contract #$i : " CONTRACT_ADDRESS
        echo "$CONTRACT_ADDRESS" >> contracts.txt
    done

    # Securely delete deploy.ts after deployment
    if [ -f deploy/deploy.ts ]; then
        rm -f deploy/deploy.ts
        show "Private key file deploy.ts deleted securely." "progress"
    fi
}

verify_contracts() {
    while IFS= read -r CONTRACT_ADDRESS; do
        show "Verifying smart contract at address: $CONTRACT_ADDRESS..." "progress"
        npx hardhat verify --network abstractTestnet "$CONTRACT_ADDRESS"
    done < contracts.txt
}

menu() {
    echo -e "\n${YELLOW}┌─────────────────────────────────────────────────────┐${NORMAL}"
    echo -e "${YELLOW}│              Script Menu Options                    │${NORMAL}"
    echo -e "${YELLOW}├─────────────────────────────────────────────────────┤${NORMAL}"
    echo -e "${YELLOW}│              1) Install Dependencies                │${NORMAL}"
    echo -e "${YELLOW}│              2) Modify configuration & compile      │${NORMAL}"
    echo -e "${YELLOW}│              3) Wallet Set Up                       │${NORMAL}"
    echo -e "${YELLOW}│              4) Deploy contract(s)                  │${NORMAL}"
    echo -e "${YELLOW}│              5) Verify contracts                    │${NORMAL}"
    echo -e "${YELLOW}│              6) Exit                                │${NORMAL}"
    echo -e "${YELLOW}└─────────────────────────────────────────────────────┘${NORMAL}"

    read -p "Enter your choice: " CHOICE

    case $CHOICE in
        1) install_dependencies ;;
        2) compilation ;;
        3) deployment ;;
        4) deploy_contracts ;;
        5) verify_contracts ;;
        6) show "Exiting..." "progress"; exit 0 ;;
        *) show "Invalid choice. Please try again." "error" ;;
    esac
}

while true; do
    menu
done
