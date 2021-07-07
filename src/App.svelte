<script lang="ts">
	import SendAxies from "././components/SendAxies.svelte";
	import SendAxiesMulti from "./components/SendAxiesMulti.svelte";

	// mode
	let mode = "multi";

</script>

<main>

	<div class="view">
		{#if false}
			<button on:click={() => { mode = "single"; }} class:selected={mode == "single"}>
				Send Axies
			</button>
		{/if}
		<button on:click={() => { mode = "multi"; }} class:selected={mode == "multi"}>
			Send Axies (Multi)
		</button>
	</div>

	{#if mode == "single"}
		<SendAxies />
	{:else if mode == "multi"}
		<SendAxiesMulti />
	{/if}

</main>

<style>
	.view {
		max-width:600px;
		margin:0 auto;
		margin-bottom:10px;
		display:flex;
		justify-content:center;
	}

	button {
		cursor:pointer;
		border-radius: 50px;
		border: none;
		background: #dddddd;
		font-size: 14px;
		padding: 8px 18px;
		margin:0 5px;
	}
	button:hover {
		background:#d2d2d2;
	}
	button.selected {
		background: #3192f5;
    	color: white;
	}
</style>