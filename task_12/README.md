# Document Porting An Existing Ethereum DApp To Polyjuice

## Document Porting An Existing Ethereum DApp To Polyjuice
### Video tutorial: https://www.youtube.com/watch?v=uc2pyQc3ZwI&t=169s

### There are 9 main step:
- Have an existing ETH DApp.
- Run ETH DApp with ganache.
- Setup Godwoken Testnet in MetaMask.
- Install dependencies for Polyjuice.
- Add config file.
- Modify some Frontend code.
- Dislay Polyjuice Address on UI
- Set Higher Gas Limit.
- Run and test DApp.

### **Step 1 Ñ  Clone an existing ETH DApp**
```
mkdir -p ~/project
cd ~/projects
git clone https://github.com/TTNguyenDev/Dapps-Support-ForceBridge -b starter
````

### **Step 2 Ñ  Run ETH DApp with ganache**
Make sure you already install docker before.

```
cd ~/projects/Dapps-Support-ForceBridge
yarn && yarn build && yarn start:ganache
```
Open another command output window under the current path,enter the command:
```
yarn ui
```

Now open a browser tab http://localhost:3000 to view ETH DApp.

### **Step 3 Ñ  Setup Godwoken Testnet in MetaMask**
Make sure your browser have a metamask extension.
Setup a custom RPC following information below:
```
Network Name: Godwoken Test Network
New RPC URL: https://godwoken-testnet-web3-rpc.ckbapp.dev/
Chain ID: 71393
Currency Symbol (optional): N/A
Block Explorer URL (optional): N/A
```
We will use this RPC to interact with Polyjuice DApp.

### **Step 4 Ñ  Install dependencies for Polyjuice**
There are two dependencies you need to add to your existing web3 Dapp to get it working with the Layer 2 solution Polyjuice.

The first, @polyjuice-provider/web3, is a custom Polyjuice web3 provider to use instead of the web3 library. It is required for interaction with Nervos Layer 2 smart contracts.

The second, nervos-godwoken-integration, is a tool that can generate a Polyjuice address based on your Ethereum address. You might be required to use it if you store values mapped to addresses in any of your smart contracts.

Install both using the follow commands:
```
cd ~/projects/Dapps-Support-ForceBridge
yarn add @polyjuice-provider/web3@0.0.1-rc7 nervos-godwoken-integration@0.0.6
```

### **Step 5 Ñ  Add config file**
Create a new configuration file with config.ts name:

```
cd ~/projects/Dapps-Support-ForceBridge/src
touch config.ts
```
Fill the below values to config.ts file.

```
export const CONFIG = {
WEB3_PROVIDER_URL: 'https://godwoken-testnet-web3-rpc.ckbapp.dev'
ROLLUP_TYPE_HASH: '0x4cc2e6526204ae6a2e8fcf12f7ad472f41a1606d5b9624beebd215d780809f6a'
ETH_ACCOUNT_LOCK_CODE_HASH: '0xdeec13a7b8e100579541384ccaf4b5223733e4a5483c3aec95ddc4c1d5ea5b22'
}
```

### **Step 6 Ñ  Modify some Frontend code**
Firstly, We will upadte the main UI file. Open the app.tsx file from this url 
```
~/projects/Dapps-Support-ForceBridge/src/ui
```

Import 3 line of code at the begining of our files:
```
import { PolyjuiceHttpProvider } from '@polyjuice-provider/web3';

import { AddressTranslator } from 'nervos-godwoken-integration';

import { CONFIG } from '../config';
```

Next find and replace line ``` const web3 = new Web3((window as any).ethereum);``` to:
```
const godwokenRpcUrl = CONFIG.WEB3_PROVIDER_URL;
const providerConfig = {
rollupTypeHash: CONFIG.ROLLUP_TYPE_HASH,
ethAccountLockCodeHash: CONFIG.ETH_ACCOUNT_LOCK_CODE_HASH,
web3Url: godwokenRpcUrl
};
const provider = new PolyjuiceHttpProvider(godwokenRpcUrl, providerConfig);
const web3 = new Web3(provider);
```

We replace the old Ethereum web3 instance to the Polyjuice web3 instance.
At this step, our DApp is capable of communicating with Polyjuice.

### **Step 7 Ñ  Dislay Polyjuice Address on UI**

Next we will display polyjuice Address to user. Add new constant and useEffect hook for this constant.
```
const [polyjuiceAddress, setPolyjuiceAddress] = useState<string | undefined>();

useEffect(() => {
    if (accounts?.[0]) {
        const addressTranslator = new AddressTranslator();
        setPolyjuiceAddress(addressTranslator.ethAddressToGodwokenShortAddress(accounts?.[0]));
    } else {
        setPolyjuiceAddress(undefined);
    }
}, [accounts?.[0]]);
```

The useEffect hook will be execute when the accounts?[0] changed.

Finally, we will display polyjuiceAddress to user. To do this, we will add one new line to html code.
```
<br />
Your Polyjuice address: <b>{polyjuiceAddress || ' - '}</b>
<br />
```

### **Step 8 Ñ  Set Higher Gas Limit**

Godwoken requires a higher gas limit to be set for transactions on the test network.

Navigate to this url:
```
 cd ~/projects/Dapps-Support-ForceBridge/src/lib/contracts
```
And open TTNguyenToken.ts files.

Add new constant at the begining of this file:
```
const DEFAULT_SEND_OPTIONS = {
    gas: 6000000
};
```

We will modify 2 function below:
```
  async setTransferToken(fromAddress: string, toAddress: string, amount: number) {
        const tx = await this.contract.methods
            .transfer(toAddress, this.web3.utils.toWei(this.web3.utils.toBN(amount)))
            .send({
                from: fromAddress
            });

        return tx;
    }
    
      async deploy(fromAddress: string) {
        const deployTx = await (this.contract
            .deploy({
                data: TTNguyenTokenJSON.bytecode,
                arguments: []
            })
            .send({
                from: fromAddress,
                to: '0x0000000000000000000000000000000000000000'
            } as any) as any);

        this.useDeployed(deployTx.contractAddress);

        return deployTx.transactionHash;
    }
```
to the new version
```
  async setTransferToken(fromAddress: string, toAddress: string, amount: number) {
        const tx = await this.contract.methods
            .transfer(toAddress, this.web3.utils.toWei(this.web3.utils.toBN(amount)))
            .send({
                ...DEFAULT_SEND_OPTIONS,
                from: fromAddress
            });

        return tx;
    }
    
      async deploy(fromAddress: string) {
        const deployTx = await (this.contract
            .deploy({
                data: TTNguyenTokenJSON.bytecode,
                arguments: []
            })
            .send({
                ...DEFAULT_SEND_OPTIONS,
                from: fromAddress,
                to: '0x0000000000000000000000000000000000000000'
            } as any) as any);

        this.useDeployed(deployTx.contractAddress);

        return deployTx.transactionHash;
    }
```

### **Step 9 Ñ The final step - Run and test DApp**
We already done all config for Polyjuice.
The final thing is run our app with this command:
```
yarn ui
```

After this, we can open our browser to view  http://localhost:3000/
