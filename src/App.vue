<template>
  <div id="app" v-if="!mainLoading">
    <div>
      tttThis a simple dApp, which can choose fee token and interact with the `Greeter` smart contract.
    </div>
    <div class="main-box">
      <div>
        Select token: <select v-model="selectedTokenAddress" v-on:change="changeToken">
          <option v-for="token in tokens" v-bind:value="token.address" v-bind:key="token.address" >
            {{ token.symbol }}
          </option>
        </select>
      </div>
      <div class="balance" v-if="selectedToken">
        <p>Balance: <span v-if="retreivingBalance">Loading...</span>
        <span v-else>{{currentBalance}} {{selectedToken.symbol}}</span></p>
        <p>Expected fee: <span v-if="retreivingFee">Loading...</span>
        <span v-else>{{currentFee}} {{selectedToken.symbol}}</span>
        <button class="refresh-button" v-on:click="updateFee">Refresh</button>
        </p>
      </div>
      <div class="greeting-input">
        <input v-model="newGreeting" :disabled="!selectedToken || txStatus!=0" placeholder="Write new greeting here..." >

        
      </div>
    </div>
  </div>
  <div id="app" v-else>
    <div class="start-screen">
      <h1>rrrWelcome to Greeter dApp!</h1>
      <button v-on:click="connectMetamask">Connect Metamask</button>
    </div>
  </div>
</template>

<script>


    //Main purpose: produce merkle Root & Upload to zkSync


    import { Contract, Web3Provider, Provider, utils } from "zksync-web3";
    import { ethers } from "ethers";

    // eslint-disable-next-line
    const GREETER_CONTRACT_ADDRESS = "0x4797e346AA25637F60bC55dc4441db6dff04B5fe"; // TODO: Add smart contract address
    // eslint-disable-next-line
    const GREETER_CONTRACT_ABI = require("./abi2.json"); // TODO: Add link to the ABI  

    const ETH_L1_ADDRESS = "0x0000000000000000000000000000000000000000";
    //const allowedTokens = require("./eth.json");
    const allowedTokens = require("./erc20.json");

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
          async changeGreeting() {
            this.txStatus = 1;
              try {

                // The example of paying fees using a paymaster will be shown in the
                // section below.
                const txHandle = await this.contract.setGreeting(this.newGreeting, await this.getOverrides());

                // TODO: Submit the transaction
                this.txStatus = 2;

                // TODO: Wait for transaction compilation
                // Wait until the transaction is committed
                await txHandle.wait();
                this.txStatus = 3;

                // Update greeting
                this.greeting = await this.getGreeting();

                this.retreivingFee = true;
                this.retreivingBalance = true;
                // Update balance and fee
                this.currentBalance = await this.getBalance();
                this.currentFee = await this.getFee();
              } catch (e) {
                alert(JSON.stringify(e));
              }

              this.txStatus = 0;
              this.retreivingFee = false;
              this.retreivingBalance = false;
        },
    loadMainScreen() {
      this.initializeProviderAndSigner();

      if(!this.provider || !this.signer) {
        alert("Follow the tutorial to learn how to connect to Metamask!");
        return;
      }

      this.getGreeting().then((greeting) => {
        this.greeting = greeting;
        this.mainLoading = false;
      });
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

<style>
    #app {
      font-family: Avenir, Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #2c3e50;
      margin-top: 30px;
    }

    #app ul{
      display: inline-block;
    }

    .main-box {
      text-align: left;
      width: 400px;

      margin: auto;
      margin-top: 40px;
    }

    .greeting-input {
      margin-top: 20px;
    }

    .change-button {
      margin-left: 20px;
    }

    .start-screen {
      margin-top: 100px;
    }

    .balance {
      margin-top: 10px;
    }

    .refresh-button {
      margin-left: 15px;
    }
</style>
