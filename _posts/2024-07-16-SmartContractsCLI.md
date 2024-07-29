---
layout: article
title: Getting Started with Smart Contracts CLI 🚀
Author: Gopal Nambiar
tags: ResilientDB SmartContractsCLI CLI
aside:
   toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/resdb-gettingstarted/code_close_up.jpeg
---

**This document provides a comprehensive guide to building a command-line interface (CLI) for Smart Contracts using Node.js. It covers setting up the project, implementing various commands, handling configurations, and adding logging functionality.**

[https://github.com/ResilientEcosystem/ResContract](https://github.com/ResilientEcosystem/ResContract)

## 🌐 Smart Contracts on ResilientDB Today

- Initiated the process by adhering to Junchao's comprehensive guide.
- Encountered a variety of issues during the installation process.
- It's crucial to build tools post the execution of `~./install.sh` and refrain from initiating the `kv_service`.
- Ensure that the smart contract service is run after the tools' construction.
- The guide effectively simplified the event flow for better understanding.
- The journey commenced with the creation of a smart account, which in turn generated a new account address.
- Leveraging `solc`, a utility offered by Solidity, enabled the compilation of our contract from a `.sol` file to a `.json` file.
- Post-deployment of our contract, it was duly executed.
- The guide provided a straightforward example of token transfer.
- Below is a run of the steps from the guide:
    
    ```bash
    # Here are some saved outputs from running the commands mentioned above.
    
    # Account creation
    gopuman@node0:~/incubator-resilientdb$ bazel-bin/service/tools/contract/api_tools/contract_tools create -c service/tools/config/interface/service.config
    cmd = create config path = service/tools/config/interface/service.config
    WARNING: Logging before InitGoogleLogging() is written to STDERR
    E20240621 13:43:30.429148 23040 contract_tools.cpp:45] create account:
    address: "0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8"
    
    # Converting the .sol contract into .json
    gopuman@node0:~/incubator-resilientdb$ solc --evm-version homestead --combined-json bin,hashes --pretty-json --optimize token.sol > token.json
    gopuman@node0:~/incubator-resilientdb$ cat token.json
    {
      "contracts": {
        "token.sol:Token": {
          "bin": "6080604052348015600f57600080fd5b506040516102c93803806102c9833981016040819052602c916040565b336000908152602081905260409020556058565b600060
    208284031215605157600080fd5b5051919050565b610262806100676000396000f3fe608060405234801561001057600080fd5b5060043610610052577c01000000000000000000000000
    00000000000000000000000000000000600035046370a082318114610057578063a9059cbb14610093575b600080fd5b61008061006536600461018b565b600160a060020a031660009081
    526020819052604090205490565b6040519081526020015b60405180910390f35b6100a66100a13660046101ad565b6100b6565b604051901515815260200161008a565b33600090815260
    20819052604081205482116101655733600090815260208190526040812080548492906100eb908490610206565b9091555050600160a060020a0383166000908152602081905260408120
    8054849290610118908490610219565b9091555050604051828152600160a060020a0384169033907fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef9060
    200160405180910390a3506001610169565b5060005b92915050565b8035600160a060020a038116811461018657600080fd5b919050565b60006020828403121561019d57600080fd5b61
    01a68261016f565b9392505050565b600080604083850312156101c057600080fd5b6101c98361016f565b946020939093013593505050565b7f4e487b7100000000000000000000000000
    000000000000000000000000000000600052601160045260246000fd5b81810381811115610169576101696101d7565b80820180821115610169576101696101d756fea264697066735822
    1220a29b2643f62235089657c12be4b680a5a56a8432ecdf0d68cd664c7a9cd3c88c64736f6c634300081a0033",
          "hashes": {
            "balanceOf(address)": "70a08231",
            "transfer(address,uint256)": "a9059cbb"
          }
        }
      },
      "version": "0.8.26+commit.8a97fa7a.Linux.g++"
    }
    
    # Deploying the smart contract
    gopuman@node0:~/incubator-resilientdb$ bazel-bin/service/tools/contract/api_tools/contract_tools deploy -c service/tools/config/interface/service.conf
    ig -p token.json -n token.sol:Token -a 1000 -m 0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8
    cmd = deploy config path = service/tools/config/interface/service.config
    WARNING: Logging before InitGoogleLogging() is written to STDERR
    E20240621 14:27:22.535681 25512 contract_client.cpp:90] send request:cmd: DEPLOY
    caller_address: "0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8"
    deploy_info {
      init_param: "1000"
      func_info {
        func_name: "balanceOf(address)"
        hash: "70a08231"
      }
      func_info {
        func_name: "transfer(address,uint256)"
        hash: "a9059cbb"
      }
      contract_bin: "6080604052348015600f57600080fd5b506040516102c93803806102c9833981016040819052602c916040565b336000908152602081905260409020556058565b600
    060208284031215605157600080fd5b5051919050565b610262806100676000396000f3fe608060405234801561001057600080fd5b5060043610610052577c01000000000000000000000
    00000000000000000000000000000000000600035046370a082318114610057578063a9059cbb14610093575b600080fd5b61008061006536600461018b565b600160a060020a031660009
    081526020819052604090205490565b6040519081526020015b60405180910390f35b6100a66100a13660046101ad565b6100b6565b604051901515815260200161008a565b33600090815
    26020819052604081205482116101655733600090815260208190526040812080548492906100eb908490610206565b9091555050600160a060020a0383166000908152602081905260408
    1208054849290610118908490610219565b9091555050604051828152600160a060020a0384169033907fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef9
    060200160405180910390a3506001610169565b5060005b92915050565b8035600160a060020a038116811461018657600080fd5b919050565b60006020828403121561019d57600080fd5
    b6101a68261016f565b9392505050565b600080604083850312156101c057600080fd5b6101c98361016f565b946020939093013593505050565b7f4e487b7100000000000000000000000
    000000000000000000000000000000000600052601160045260246000fd5b81810381811115610169576101696101d7565b80820180821115610169576101696101d756fea264697066735
    8221220a29b2643f62235089657c12be4b680a5a56a8432ecdf0d68cd664c7a9cd3c88c64736f6c634300081a0033"
      contract_name: "token.sol:Token"
    }
    E20240621 14:27:22.661372 25512 contract_tools.cpp:58] deploy contract:
    owner_address: "0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8"
    contract_address: "0xfc08e5bfebdcf7bb4cf5aafc29be03c1ae101b3b"
    deploy_hash: "0x4248e81c2e9e162e29f7bb502053a774eae3ccf2e22d9b67a5678ea508a8de10"
    
    # Executing the smart contract
    gopuman@node0:~/incubator-resilientdb$ bazel-bin/service/tools/contract/api_tools/contract_tools execute -c service/tools/config/interface/service.config -s 0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8 -d 0xfc08e5bfebdcf7bb4cf5aafc29be03c1ae101b3b -n transfer(address,uint256) -a 0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8,100 -m 0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8
    cmd = execute config path = service/tools/config/interface/service.config
    WARNING: Logging before InitGoogleLogging() is written to STDERR
    E20240621 14:31:29.780866 26219 contract_client.cpp:90] send request:cmd: EXECUTE
    caller_address: "0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8"
    exec_info {
      func_name: "transfer(address,uint256)"
      func_hash: "a9059cbb"
      arg: "0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8,100"
      is_tx: false
    }
    contract_address: "0xfc08e5bfebdcf7bb4cf5aafc29be03c1ae101b3b"
    E20240621 14:31:29.880665 26219 contract_tools.cpp:82] execute contract:
    func_params {
      param: "0x1be8e78d765a2e63339fc99a66320db73158a35a"
      param: "100"
      func_name: "transfer(address,uint256)"
    }
    E20240621 14:34:51.034179 25711 contract_tools.cpp:71] execute result:
    0x0000000000000000000000000000000000000000000000000000000000000001
    ```
    

## 🔍 Exploring Leading Technologies

To fully comprehend the capabilities that the ResilientDB Smart Contracts CLI must eventually offer, I delved into a few leading technologies, exploring their features and functionalities. Here are the key technologies I worked with:

- **Truffle Framework**: Standing tall as the most favored development framework for Ethereum, Truffle casts a wide net of tools for crafting smart contracts with Solidity, facilitating their testing, and ensuring their successful deployment to the blockchain.
- **Ganache**: Acting as your personal Ethereum development blockchain, Ganache empowers you to deploy contracts, create applications, and conduct tests. It mirrors the Ethereum network on your local setup.
- **Metamask**: As a browser extension, Metamask enables you to engage with Ethereum Dapps directly from your browser. Doubling as an Ethereum wallet, it provides an interface for account management.

---

## 📈 Getting Started with ResDB Smart Contracts CLI

**NOTE:** Before starting to write code for the Command Line Interface (CLI), we had to decide on the programming language to use. We could choose from a variety of languages to write the CLI. Considering the CLI will closely work with another project within the ResilientDB ecosystem, ResVault, which is written in Node.js, we thought it best to write the CLI in Node.js, the same runtime used by ResVault.

## 👨🏻‍💻 Developing the Smart Contracts CLI

### **Step 1: Setting Up the Project**

To start, we set up a Node.js project for our CLI. Here are the initial steps and configurations:

1. **Create the Project Directory:**
    
    ```bash
    mkdir smart-contracts-cli
    cd smart-contracts-cli
    ```
    
2. Initialize the Project
3. Create the main script
4. Update `package.json`

### Step 2: Implementing the First Command

Our first command will be for account creation, which we will call `create`. This command should take one argument, the path to a config file (`--config` or `--c`).

- **index.js**
    
    ```jsx
    #!/usr/bin/env node
    
    const { Command } = require('commander');
    const { exec } = require('child_process');
    const path = require('path');
    const inquirer = require('inquirer');
    const fs = require('fs-extra');
    const os = require('os');
    
    const program = new Command();
    const CONFIG_FILE_PATH = path.join(os.homedir(), '.smart-contracts-cli-config.json');
    
    async function getResDBHome() {
    if (process.env.ResDB_Home) {
    return process.env.ResDB_Home;
    }
    
    if (await fs.pathExists(CONFIG_FILE_PATH)) {
    const config = await fs.readJson(CONFIG_FILE_PATH);
    if (config.resDBHome) {
    process.env.ResDB_Home = config.resDBHome;
    return config.resDBHome;
    }
    }
    
    return null;
    }
    
    async function setResDBHome(resDBHome) {
    process.env.ResDB_Home = resDBHome;
    await fs.writeJson(CONFIG_FILE_PATH, { resDBHome });
    }
    
    async function promptForResDBHome() {
    const answers = await inquirer.prompt([
    {
    type: 'input',
    name: 'resDBHome',
    message: 'Please enter the ResDB_Home path:',
    },
    ]);
    
    const resDBHome = answers.resDBHome;
    await setResDBHome(resDBHome);
    
    return resDBHome;
    }
    
    program
    .version('1.0.0')
    .description('Smart Contracts CLI');
    
    program
    .command('create')
    .description('Create a new account')
    .option('-c, --config <path>', 'Path to the config file')
    .action(async (options) => {
    const configPath = options.config;
    if (!configPath) {
    console.error('Error: Config file path is required');
    process.exit(1);
    }
    let resDBHome = await getResDBHome();
    if (!resDBHome) {
      resDBHome = await promptForResDBHome();
    }
    
    const command = `${path.join(resDBHome, 'bazel-bin/service/tools/contract/api_tools/contract_tools')} create -c ${configPath}`;
    
    exec(command, (error, stdout, stderr) => {
      if (error) {
        console.error(`Error: ${error.message}`);
        return;
      }
      if (stderr) {
        console.error(`stderr: ${stderr}`);
        return;
      }
      console.log(`stdout: ${stdout}`);
    });
    });
    
    program.parse(process.argv);
    ```
    

### Step 3: Dealing with Configurations using `config.js`

One of the core components of our Smart Contracts CLI is the `config.js` file. This file manages the configuration settings necessary for our CLI to function correctly, particularly the `ResDB_Home` path. This path points to the directory where the ResilientDB installation resides.

- The Role of `ResDB_Home`
    
    The `ResDB_Home` path is crucial as it allows the CLI to locate and execute ResilientDB-related binaries and scripts. Without this path, the CLI would not know where to find the necessary tools to create, deploy, and execute smart contracts.
    

- `config.js` Implementation
    
    The `config.js` file contains logic to prompt the user for the `ResDB_Home` path the first time they use the CLI, and then stores this path for future use. Here’s how it works:
    
    1. **Environment Variable Check**: First, the CLI checks if the `ResDB_Home` environment variable is already set. If it is, this value is used.
    2. **Configuration File Check**: If the environment variable is not set, the CLI checks a configuration file (located at `~/.smart-contracts-cli-config.json`) to see if the `ResDB_Home` path has been saved previously.
    3. **User Prompt**: If neither the environment variable nor the configuration file provides the `ResDB_Home` path, the CLI prompts the user to enter it.
    4. **Saving the Path**: Once the user provides the `ResDB_Home` path, it is stored both in the environment variable and in the configuration file for future use, so the user is not prompted again.

- **config.js**
    
    ```jsx
    const path = require('path');
    const inquirer = require('inquirer');
    const fs = require('fs-extra');
    const os = require('os');
    
    const CONFIG_FILE_PATH = path.join(os.homedir(), '.smart-contracts-cli-config.json');
    
    async function getResDBHome() {
      if (process.env.ResDB_Home) {
        return process.env.ResDB_Home;
      }
    
      if (await fs.pathExists(CONFIG_FILE_PATH)) {
        const config = await fs.readJson(CONFIG_FILE_PATH);
        if (config.resDBHome) {
          process.env.ResDB_Home = config.resDBHome;
          return config.resDBHome;
        }
      }
    
      return null;
    }
    
    async function setResDBHome(resDBHome) {
      process.env.ResDB_Home = resDBHome;
      await fs.writeJson(CONFIG_FILE_PATH, { resDBHome });
    }
    
    async function promptForResDBHome() {
      const answers = await inquirer.prompt([
        {
          type: 'input',
          name: 'resDBHome',
          message: 'Please enter the ResDB_Home path:',
        },
      ]);
    
      const resDBHome = answers.resDBHome;
      await setResDBHome(resDBHome);
    
      return resDBHome;
    }
    
    module.exports = {
      getResDBHome,
      setResDBHome,
      promptForResDBHome,
    };
    
    ```
    

- How It Works
    1. **getResDBHome()**: This function checks for the `ResDB_Home` environment variable first. If not set, it checks the configuration file for a saved path. If found, it sets the environment variable to this path and returns it. If not found, it returns `null`.
    2. **setResDBHome(resDBHome)**: This function sets the `ResDB_Home` environment variable and writes the path to the configuration file.
    3. **promptForResDBHome()**: This function uses the `inquirer` library to prompt the user to enter the `ResDB_Home` path. After the user provides the path, it calls `setResDBHome()` to save it.

### Step 4: Testing the CLI

1. Ensure all dependencies are installed: 

```bash
npm install
```

2. Link the CLI for local development:

```bash
npm link
```

3. Run your CLI commands to test:

- First run:

```bash
smart-contracts-cli create --config /path/to/config/file
```

**Expected behavior**: It should prompt for the ResDB_Home path, save it, and then execute the command.

- Subsequent run:

```bash
smart-contracts-cli create --config /path/to/config/file
```

**Expected behavior**: It should not prompt for the ResDB_Home path again and should directly execute the command.

### Step 5: Implementing the `compile` command

The `compile` command is essential for converting Solidity smart contract files (`*.sol`) into JSON format (`*.json`) using the Solidity compiler (`solc`). Here’s how we can implement this command in our Smart Contracts CLI:

- **Command Definition**: Define the `compile` command in our CLI.

```jsx
program
  .command('compile <inputFile> <outputFile>')
  .description('Compile a Solidity contract')
  .action(async (inputFile, outputFile) => {
    // Implementation details will go here
  });
```

- **Using `solc`**: Utilize the Solidity compiler (`solc`) to compile the Solidity contract.

```jsx
const path = require('path');
const fs = require('fs-extra');
const solc = require('solc');

async function compileContract(inputFile, outputFile) {
  const contractPath = path.resolve(inputFile);
  const contractName = path.basename(contractPath, '.sol');
  const source = await fs.readFile(contractPath, 'utf8');

  const input = {
    language: 'Solidity',
    sources: {
      [contractName]: {
        content: source,
      },
    },
    settings: {
      outputSelection: {
        '*': {
          '*': ['*'],
        },
      },
    },
  };

  const output = JSON.parse(solc.compile(JSON.stringify(input)));
  const compiledContract = output.contracts[contractName][contractName];

  await fs.writeFile(outputFile, JSON.stringify(compiledContract, null, 2));
  console.log(`Contract compiled successfully. Output saved to ${outputFile}`);
}
```

- **Error Handling**: Implement error handling and logging for better user experience.

```jsx
program
  .command('compile <inputFile> <outputFile>')
  .description('Compile a Solidity contract')
  .action(async (inputFile, outputFile) => {
    try {
      await compileContract(inputFile, outputFile);
    } catch (error) {
      console.error('Error compiling contract:', error.message);
    }
  });
```

- `compile` Command
    - **Description**: Compile a Solidity contract (`.sol`) into a JSON format (`.json`) using the Solidity compiler (`solc`).
    - **Usage**: `smart-contracts-cli compile <inputFile> <outputFile>`
    - **Arguments**:
        - `<inputFile>`: Path to the Solidity contract file (`.sol`) that you want to compile.
        - `<outputFile>`: Path where the compiled JSON file (`.json`) will be saved.
    - **Functionality**: This command takes a Solidity contract file as input and compiles it into JSON format using `solc`. It generates a JSON file that describes the compiled contract's bytecode, ABI (Application Binary Interface), and other metadata.
    - **Implementation Details**: Utilizes the `solc` Solidity compiler library to compile the contract.
    - **Error Handling**: Handles errors that may occur during compilation and outputs appropriate error messages.
    
- **Example Usage**
    
    Assuming you have a Solidity contract file named `MyContract.sol` located in the current directory and you want to compile it into `MyContract.json`, here’s how you would use the `compile` command:
    

```bash
smart-contracts-cli compile MyContract.sol MyContract.json
```

This command compiles `MyContract.sol` using `solc` and saves the compiled output to `MyContract.json` in a readable JSON format.

### Step 6: Implementing the `deploy` command

The `deploy` command is essential to our Smart Contracts CLI. This command will allow us to deploy a smart contract using specific parameters. The internal command to be executed will be constructed using various options provided by the user. Added the following code changes to accommodate the `deploy` functionality.

- Code changes

```jsx
program
  .command('deploy')
  .description('Deploy a smart contract')
  .option('-c, --config <configPath>', 'Client configuration path')
  .option('-p, --contract <contractPath>', 'Contract path')
  .option('-n, --name <contractName>', 'Contract name')
  .option('-a, --arguments <parameters>', 'Parameters to construct the contract object')
  .option('-m, --owner <ownerAddress>', 'Contract owner’s address')
  .action(async (options) => {
    const { config: configPath, contract, name, arguments: args, owner } = options;
    if (!configPath || !contract || !name || !args || !owner) {
      console.error('Error: All options (-c, -p, -n, -a, -m) are required.');
      process.exit(1);
    }

    let resDBHome = await getResDBHome();
    if (!resDBHome) {
      resDBHome = await promptForResDBHome();
    }

    const command = `${path.join(resDBHome, 'bazel-bin/service/tools/contract/api_tools/contract_tools')} deploy -c ${configPath} -p ${contract} -n ${name} -a ${args} -m ${owner}`;

    exec(command, (error, stdout, stderr) => {
      if (error) {
        console.error(`Error: ${error.message}`);
        return;
      }
      if (stderr) {
        console.error(`stderr: ${stderr}`);
        return;
      }
      console.log(`stdout: ${stdout}`);
    });
  });
```

- **Command Description:** The `deploy` command allows for the deployment of a smart contract.

- **Options:**
    - `-c, --config <configPath>`: Specifies the client configuration path.
    - `-p, --contract <contractPath>`: Specifies the contract path.
    - `-n, --name <contractName>`: Specifies the contract name.
    - `-a, --arguments <parameters>`: Specifies the parameters to construct the contract object.
    - `-m, --owner <ownerAddress>`: Specifies the contract owner’s address.
    
- **Action:**
    - Validates that all necessary options are provided.
    - Retrieves the `ResDB_Home` path, prompting the user if not already set.
    - Constructs and executes the deployment command using the provided options and `ResDB_Home` path.
    
- **Example Usage:**
    
    To deploy a smart contract, you can use the following command:
    
    ```bash
    smart-contracts-cli deploy -c /path/to/config -p /path/to/contract.json -n contract.sol:ContractName -a "1000" -m 0xYourAddress
    ```
    

Replace `/path/to/config`, `/path/to/contract.json`, `contract.sol:ContractName`, `1000`, and `0xYourAddress` with your specific configuration path, contract path, contract name, contract parameters, and contract owner’s address, respectively.

### Step 7: Implementing the `execute` command

The final step in building the Smart Contracts CLI is to implement the `execute` command. This command allows you to execute a function within a deployed smart contract.

- **Usage:**

The `execute` command takes several parameters, including the client configuration path, the contract owner’s address, the contract address, the function signature, and the function arguments.

```bash
smart-contracts-cli execute --config <service.config> --owner <ownerAddress> 
--contract <contractAddress> --function <functionSignature> --arguments <functionArguments>
```

- **Options:**
    - `service.config`: Path to the client configuration file.
    - `ownerAddress`: The address of the contract owner.
    - `contractAddress`: The address of the deployed contract.
    - `functionSignature`: The function signature (e.g., "transfer(address,uint256)").
    - `functionArguments`: The arguments to pass to the function.

- **Code changes:**
    - To add the `execute` command, the following code changes were made:
    - **index.js**
        
        ```jsx
        program
          .command('execute')
          .description('Execute a function within a deployed smart contract')
          .option('-c, --config <configPath>', 'Client configuration path')
          .option('-m, --owner <ownerAddress>', 'Contract owner’s address')
          .option('-s, --contract <contractAddress>', 'Contract address')
          .option('-f, --function <functionSignature>', 'Function signature')
          .option('-a, --arguments <functionArguments>', 'Function arguments')
          .action(async (options) => {
            const { config: configPath, owner, contract, function: func, arguments: args } = options;
            if (!configPath || !owner || !contract || !func || !args) {
              console.error('Error: All options (-c, -m, -s, -f, -a) are required.');
              process.exit(1);
            }
        
            let resDBHome = await getResDBHome();
            if (!resDBHome) {
              resDBHome = await promptForResDBHome();
            }
        
            const command = `${path.join(resDBHome, 'bazel-bin/service/tools/contract/api_tools/contract_tools')} execute -c ${configPath} -m ${owner} -s ${contract} -f "${func}" -a ${args}`;
        
            exec(command, (error, stdout, stderr) => {
              if (error) {
                console.error(`Error: ${error.message}`);
                return;
              }
              if (stderr) {
                console.error(`stderr: ${stderr}`);
                return;
              }
              console.log(`stdout: ${stdout}`);
            });
          });
        
        ```
        
        This code defines the `execute` command with the necessary options and constructs the command to be executed based on the user’s input. It then runs the command using Node.js's `exec` function, handling any errors and logging the output.
        

### **Step 8: Implementing Logging Functionality**

To keep track of the commands executed and their outputs, we need to implement a logging mechanism. This will help with debugging and auditing.

- **Logging Requirements**
    
    The logging functionality should:
    
    - Record each executed command with a timestamp.
    - Capture both standard output (stdout) and standard error (stderr).
    - Save all log entries to a file named `cli-logs.log`.
        
        
- **Adding Logging Functionality**
    1. First, install the `winston` logging library:
    
    ```bash
    npm install winston
    ```
    
    2. **Import and Configure Winston:**
        
        Added the following code to set up the logger:
        
        - **index.js**
            
            ```jsx
            const winston = require('winston');
            
            const logger = winston.createLogger({
              level: 'info',
              format: winston.format.combine(
                winston.format.timestamp(),
                winston.format.printf(({ timestamp, level, message }) => {
                  return `${timestamp} ${level}: ${message}`;
                })
              ),
              transports: [
                new winston.transports.File({ filename: 'cli-logs.log' })
              ],
            });
            ```
            
        
    3. **Log Commands and Outputs:**
        
        Updated each command to log the executed command and its output. Here’s an example for the `create` command
        
    4. **Test the Logging Functionality**:
        
        Run various commands and check the `cli-logs.log` file to ensure all logs are recorded as expected.
        
        **Example log entry:**
        
    
    ```bash
    2024-07-10T00:39:17.902Z info: Command: create -c /users/gopuman/incubator-resilientdb/service/tools/config/interface/service.config
    stdout: Account created with address: 0x213ddc8770e93ea141e1fc673e017e97eadc6b96
    ```