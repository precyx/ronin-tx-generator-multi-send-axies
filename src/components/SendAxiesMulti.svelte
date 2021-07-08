<script lang="ts">
    import {onMount} from "svelte";
    import { ethers, utils } from "ethers";
    import {Axie_ABI} from "../data/Axie_ABI.svelte";
    import {asyncForEach, sleep, debounce} from "../utils/Utils.svelte";

    // const
    const N = `
`;

    // loading
    let loading = false;

    // blockchain
    let CHAIN_ID = 2020;
    let provider:ethers.providers.JsonRpcProvider;
    let wallet:ethers.Wallet;
    let public_address:string;

    $: calculatePublicAddress(private_key);

    // user
    let private_key:string = "";
    let to_ronin:string = "";
    let axie_id:string = "";
    let signedTransactions:string = "";

    // counts
    let num_to_ronins = 0;
    let num_axie_ids = 0;
    let num_signed_transactions = 0;

    // transactions
    let completed_txs = 0;
    let total_txs = 0;

    // reactivity
    $: calculateNumToAddresses(to_ronin);
    $: calculateNumAxieIds(axie_id);
    $: calculateNumSignedTransactions(signedTransactions);


    /**
     * Reactivity
     */
    const calculatePublicAddress = (_private_key) => {
        if(!_private_key) return;
        debounced_SetupProvider();
    }

    const calculateNumToAddresses = (_to_ronin:string) => {
        num_to_ronins = _to_ronin.split(N).length || 0;
    }
    const calculateNumAxieIds = (_axie_id:string) => {
        num_axie_ids = _axie_id.split(N).length || 0;
    }
    const calculateNumSignedTransactions = (_signedTransactions:string) => {
        num_signed_transactions = _signedTransactions.split(N+N).filter(x => x).length || 0;
    }

    onMount(() => {
        //setupProvider();
    });


    const debounced_SetupProvider = debounce(() => setupProvider(), 900);

    const setupProvider = () => {
        // get provider
        provider = new ethers.providers.JsonRpcProvider("https://proxy.roninchain.com/free-gas-rpc", CHAIN_ID);
        //provider = new ethers.providers.JsonRpcProvider("https://api.roninchain.com/rpc", 2021);

        // get wallet from private_key
        wallet = new ethers.Wallet(private_key);
        wallet = wallet.connect(provider);

        // set public address
        public_address = wallet.address.replace("0x", "ronin:");
    }


    /**
     * Generates 'send-axie' transactions based on the input
     * Also signs the transaction with the private key
     */
    let generateTransactions = async () => {

        loading = true;

        setupProvider();

        // create axie contract
        let Axie_Contract_Address = `0x32950db2a7164ae833121501c797d79e7b79d74c`;
        
        let AxieContract = new ethers.Contract(Axie_Contract_Address, Axie_ABI, provider);
        
        console.log("axie contract", AxieContract);

        let fromRoninAddress = public_address.replace("ronin:", "0x");
        let toRoninAddresses = to_ronin.split(N).map(x => x.replace("ronin:", "0x"));
        let tokenIds = axie_id.split(N).map(x => parseInt(x.replace(/[^0-9]/g, '') ));

        // get tx count
        let tx_count = await wallet.getTransactionCount();

        completed_txs = 0;
        total_txs = toRoninAddresses.length;
        
        let tx_string = "";

        await sleep(650);

        await asyncForEach(toRoninAddresses, async (el,i) => {

            let _ronin_address = toRoninAddresses[i];
            let _token_id = tokenIds[i];

            await sleep(80);
            
            // create transaction
            let tx = await AxieContract.populateTransaction.safeTransferFrom(fromRoninAddress, _ronin_address, _token_id);

            // set gasLimit chainId and nonce
            tx.gasLimit = utils.parseUnits(450000 + "", "wei");
            tx.chainId = CHAIN_ID;
            tx.nonce = tx_count + i;
            
            // sign transaction
            let signedTx = await wallet.signTransaction(tx);
     
            console.log("tx", tx);
            console.log("signed", signedTx);

            if(signedTx) {
                tx_string += (signedTx + N + N);
            }

            completed_txs = completed_txs + 1;
        });

        signedTransactions = tx_string;

        loading = false;
    }


</script>


<div class="area">

    <div class="box warning">
        This tool is strongly recommended to be used in <b>Incognito mode</b> and <b>Offline</b>.
        <br> Source Code is public on github: <a target="_blank" href="https://github.com/precyx/ronin-tx-generator-multi-send-axies"><b>Precyx</b></a>
    </div>

    <div class="titlebar">
        <div class="sub">Ronin Transaction Generator</div>
        <h1 class="b">Send Axies Multi</h1>
    </div>

    <div class="box">

        <div class="input">
            <div class="line">
                <div class="label">Private Key:</div>
                <input type="password" bind:value={private_key} class="private_key_input"/>
            </div>
            <div class="line">
                <div class="label">Ronin Account:</div>
                <input type="text" bind:value={public_address} disabled={true}/>
            </div>
            <div class="line">
                <div class="label">
                    To Ronin: 
                    {#if num_to_ronins > 0}
                        <span class="counter">({num_to_ronins}X)</span>
                    {/if}
                </div>
                <textarea type="text" bind:value={to_ronin} />
            </div>
            <div class="line">
                <div class="label">
                    Axie ID:
                    {#if num_axie_ids > 0}
                        <span class="counter">({num_axie_ids}X)</span>
                    {/if}
                </div>
                <textarea bind:value={axie_id} />
            </div>
            <div class="center">
                <button on:click={generateTransactions} disabled={
                    !public_address
                }>
                    Generate TX
                </button>
            </div>
        </div>

        <div class="output">
            {#if loading}
                <div class="loader">
                    Generating... ({completed_txs} / {total_txs})
                </div>
            {/if}

            <div class="label">
                Signed Transaction
                {#if num_signed_transactions > 0}
                    <span class="counter">({num_signed_transactions}X)</span>
                {/if}
            </div>
            <textarea class="signed_tx_output" bind:value={signedTransactions} />
        </div>
    </div>
</div>


<style>

    .box.warning {
        padding: 8px;
        border-radius: 8px;
        margin-bottom: 30px;
        width: 100%;
        background: #ffede8;
        border: 1px solid #ff6940;
        color: #c00707;
        font-size:14px;
        box-sizing: border-box;
    }

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

    .loader {
        color: grey;
    text-align: center;
    margin: 10px 0;
    }

    .line {
        margin-bottom:10px;
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

    .counter {
        font-weight:bold;
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
    button:disabled {
        background: #dddddd;
        color: #828282;
        user-select: none;
        pointer-events: none;
    }

    h1 {
        margin:0;
        margin-bottom: 40px;
    }

    h1 .sub {
        font-weight:normal;
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
        border-color: #7c7c7c;
    }
    input:disabled {
        border-color: #d9d9d9;
    }
    .private_key_input {
        width: 100%;
    }

    textarea {
        font-size: 11px;
        font-family: monospace;
        font-weight: 400;
        line-height: 130%;

        width:100%;
        height:60px;
        border-color: #7c7c7c;
        /*background: #fafafa;*/
        resize: vertical;
    }

    .signed_tx_output {
        background: #fafafa;
    }
</style>