<script context="module">
	/**
	* Async foreach
	* @param array
	* @param callback
	*/
	export const asyncForEach = async (array, callback) => {
		for (let index = 0; index < array.length; index++) {
			await callback(array[index], index, array);
		}
	};

	/*
	* Sleep for a duration ins Micro Seconds
	*/
	export const sleep = ms => new Promise(res => setTimeout(res, ms));

</script>