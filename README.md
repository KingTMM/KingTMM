const fs = require('fs/promises'); const axios = require('axios'); const Web3 = require('web3'); require('dotenv').config(); // Load environment variables from .env file const ALCHEMY_RPC_URL = process.env.ALCHEMY_RPC_URL; const COIN_GECKO_URL = process.env.COIN_GECKO_URL; const privateKey = process.env.PRIVATE_KEY; const senderAddress = process.env.SENDER_ADDRESS; const receiverAddress = process.env.RECEIVER_ADDRESS; const ALCHEMY_API_KEY = process.env.ALCHEMY_API_KEY; const usdtContractAddress = process.env.USDT_CONTRACT_ADDRESS; const paymentAmount = process.env.PAYMENT_AMOUNT; if (!ALCHEMY_RPC_URL || !privateKey || !senderAddress) { console.error("Error: Missing environment variables. Please set ALCHEMY_RPC_URL, PRIVATE_KEY, and SENDER_ADDRESS."); process.exit(1); } const web3 = new Web3(new Web3.providers.HttpProvider(ALCHEMY_RPC_URL)); const usdtAbi = [ { "constant": false, "inputs": [ { "name": "_to", "type": "address" }, { "name": "_value", "type": "uint256" } ], "name": "transfer", "outputs": [ { "name": "", "type": "bool" } ], "type": "function" } ]; async function convertAndSendUSDT(data) { try { const currency = data.pmtInf.currency; if (currency !== 'EUR') { throw new Error('Transaction is not in EUR.'); } const response = await axios.get(COIN_GECKO_URL + '/simple/price?ids=tether&vs_currencies=eur'); if (!response.data || !response.data.tether || !response.data.tether.eur) { throw new Error('Invalid or missing exchange rate data from CoinGecko.'); } const exchangeRate = response.data.tether.eur; const usdtAmount = (parseFloat(paymentAmount.toString()) / exchangeRate).toFixed(6); console.log("Converted amount:", usdtAmount, "USDT"); const nonce = await web3.eth.getTransactionCount(senderAddress, 'latest'); const usdtContract = new web3.eth.Contract(usdtAbi, usdtContractAddress); const gasLimit = 60000; const gasPrice = await web3.eth.getGasPrice(); const balance = await web3.eth.getBalance(senderAddress); if (balance < gasPrice * gasLimit) { throw new Error('Insufficient balance for gas fees.'); } const tx = { from: senderAddress, to: usdtContractAddress, gas: gasLimit, gasPrice: gasPrice, data: usdtContract.methods.transfer(receiverAddress, web3.utils.toWei(usdtAmount, 'mwei')).encodeABI(), nonce: nonce, value: 0, chainId: 1 }; console.log("Transaction details:", tx); const signedTx = await web3.eth.accounts.signTransaction(tx, privateKey); const receipt = await web3.eth.sendSignedTransaction(signedTx.rawTransaction) .on('transactionHash', (hash) => console.log('Transaction hash:', hash)) .on('receipt', (receipt) => console.log('Transaction receipt:', receipt)) .on('confirmation', (confirmationNumber, receipt) => console.log('Confirmation Number:', confirmationNumber)) .on('error', (error) => { throw new Error('Transaction Error:' + error) }); console.log('Transaction successful:', receipt); } catch (error) { console.error('Error during conversion or transfer:', error.message); console.error('Error Stack:', error.stack); } } // Example usage const sampleData = { "msgId": "62953993124543DB", "cdtr": { "nm": "John Doe", "adr": { "str": "123 Main St", "cty": "New York", "ctry": "US" }, "id": { "BIC": "ABC123456" }, "crypAcct": { "addr": "0x742d35Cc6634C0532925a3b844Bc454e4438f44e" } }, "dbtr": { "nm": "HDH-NORD-BAU GMBH", "adr": { "str": "KORACHSTRASSE 33", "cty": "HAMBURG", "ctry": "DE"
