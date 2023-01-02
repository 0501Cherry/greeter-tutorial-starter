<template>
  <div>
    <div>
      <input type='button' value='Create Merkle Root & Upload Data' id='btnRegularUpload'>
    </div>
    <div>
      DataID:
      <input type='text' id='dataId'>
    </div>
    <div>
      Encrypted Data:
      <textarea id='encryptedData'></textarea>
    </div>
    <div>
      <input type='button' value='Verify Data' id='btnVerifyData'>
    </div>
  </div>
</template>

<script>
    //Main purpose: produce merkle Root & Upload to zkSync

    import { Contract, Web3Provider, Provider} from "zksync-web3"; //, utils 
    //import { ethers } from "ethers";

    // eslint-disable-next-line
    const GREETER_CONTRACT_ADDRESS = "0x4797e346AA25637F60bC55dc4441db6dff04B5fe"; // TODO: Add smart contract address
    // eslint-disable-next-line
    const GREETER_CONTRACT_ABI = require("./abi2.json"); // TODO: Add link to the ABI  

    //const ETH_L1_ADDRESS = "0x0000000000000000000000000000000000000000";
    //const allowedTokens = require("./eth.json");
    //const allowedTokens = require("./erc20.json");

    export default {
        name: 'App',
        methods: {
          initializeProviderAndSigner() {
            // TODO: initialize provider and signer based on `window.ethereum`

            this.provider = new Provider('https://zksync2-testnet.zksync.dev');

            // Note that we still need to get the Metamask signer
            this.signer = (new Web3Provider(window.ethereum)).getSigner();
            this.contract = new Contract(
                GREETER_CONTRACT_ADDRESS,
                GREETER_CONTRACT_ABI,
                this.signer
            );

          },
          async storeMerkleRoot(){
            var merkleRootId = "rrrr";
            var merkleRoot = "asdf";
            return await this.contract.storeMerkleRoot(merkleRootId, merkleRoot);
          },
          async verify(){
            var root = "asdf";
            var leaf = "ddd";
            return await this.contract.verify(root, leaf);
          },  
          loadMainScreen() {
            this.initializeProviderAndSigner();

            if(!this.provider || !this.signer) {
              alert("Follow the tutorial to learn how to connect to Metamask!");
              return;
            }
          },  
          connectMetamask() {
            window.ethereum.request({ method: 'eth_requestAccounts' })
              .then(() => {
                if (+window.ethereum.networkVersion == 280) {
                  this.loadMainScreen();
                } else {
                  alert("Please switch network to zkSync!");
                }
              })
              .catch((e) => console.log(e)); 
          }
        }
    }
</script>
