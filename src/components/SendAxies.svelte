<script lang="ts">
    import {onMount} from "svelte";
    import { ethers } from "ethers";
    import {ERC720_ABI} from "../data/ERC720_ABI.svelte";
    import {Axie_ABI} from "../data/Axie_ABI.svelte";


    // blockchain
    let provider;
    let wallet;


    // user
    let mnemonic:string = "";
    let signedTransactions:string = "";
    let from_ronin:string = "";
    let to_ronin:string = "";
    let axie_id:string = "";

    onMount(() => {
        //setupProvider();
    });


    const setupProvider = () => {
        // get provider
        provider = new ethers.providers.JsonRpcProvider("https://proxy.roninchain.com/free-gas-rpc", 2020);

        // get wallet from mnemonic
        let account_path = `m/44'/60'/0'/0/${1}`;
        wallet = ethers.Wallet.fromMnemonic(mnemonic, account_path)
        wallet = wallet.connect(provider);
    }


    // create send axie transaction
    //let tx = safeTransferAxie(ronin_address, receiver_ronin_address, parseInt(selected_axie_id+""));

    // send transaction
    //let receipt = await wallet.sendTransaction(tx);


    let createTransaction = async () => {
        setupProvider();

        // create axie (ERC-721) contract
        let Axie_Contract_Address = `0x32950db2a7164ae833121501c797d79e7b79d74c`;
        
        let AxieContract = new ethers.Contract(Axie_Contract_Address, Axie_ABI, provider);
        
        console.log("axie contract", AxieContract);

        let fromRoninAddress = from_ronin;
        let toRoninAddress = to_ronin;
        let tokenId = parseInt(axie_id);
        
        // create transaction
        let tx = await AxieContract.populateTransaction.safeTransferFrom(fromRoninAddress, toRoninAddress, tokenId);
        // sign transaction
        let signedTx = await wallet.signTransaction(tx);

        
        console.log("tx", tx);
        console.log("signed", signedTx);

        signedTransactions = signedTx;
    }


</script>


<div class="area">

    <div class="titlebar">
        <div class="sub">Ronin Transaction Generator</div>
        <h1 class="b">Send Axies</h1>
    </div>

    <div class="box">

        <div class="input">
            <div class="line">
                <div class="label">Mnemonic:</div>
                <input type="password" bind:value={mnemonic} class="seed_input"/>
            </div>
            <div class="line">
                <div class="label">From Ronin:</div>
                <input type="text" bind:value={from_ronin} />
            </div>
            <div class="line">
                <div class="label">To Ronin:</div>
                <input type="text" bind:value={to_ronin} />
            </div>
            <div class="line">
                <div class="label">Axie ID:</div>
                <input type="number" min="1" max="9999999999" step="1" bind:value={axie_id} />
            </div>
            <div class="center">
                <button on:click={createTransaction}>
                    Generate TX
                </button>
            </div>
        </div>

        <div class="output">

            <div class="label">Signed Transaction</div>
            <textarea bind:value={signedTransactions} />
        </div>
    </div>
</div>


<style>

    .titlebar {
        display:flex;
        flex-flow:column;
    }

    .titlebar .sub {
        font-size: 16px;
        margin-bottom: 0;
        color: #b7b7b7;
        font-family: monospace;
    }

    .center {
        display:flex;
        align-items:center;
        justify-content: center;
    }

    .label {
        margin-bottom: 8px;
        font-weight: 400;
        font-size: 14px;
        color: #007de7;
    }

    .output {
        margin-top:40px;
    }

    button {
        background: #3192f5;
        color: white;
        border: none;
        border-radius: 50px;
        padding: 0 35px;
        height: 35px;
        cursor:pointer;
        margin-top:10px;
    }
    button:hover {
        background:#55a9ff;
    }

    h1 {
        margin:0;
        margin-bottom: 40px;
    }

    h1 .sub {
        font-weight:normal;
    }

    .line {
        margin-bottom:10px;
    }

    .area {
        padding:20px 25px;
        margin:0 auto;
        max-width:600px;
        background: #ffffff;
        border-radius: 14px;
        box-shadow: 0 5px 18px #00000021;
    }

    input {
        width:100%;
        font-size:14px;
    }
    .seed_input {
        width: 100%;
    }

    textarea {
        font-size: 11px;
        font-family: monospace;
        font-weight: 400;
        line-height: 130%;

        width:100%;
        height:150px;
        border-color: #7c7c7c;
        background: #fafafa;
        resize: vertical;
    }
</style>