<script>
	import { onMount } from 'svelte';

	// import or require (depending on your module system)
	import { ExtismContext } from '@extism/runtime-browser';

	// @ts-ignore
	import plugin from '../../plugin/target/wasm32-unknown-unknown/release/plugin.wasm?url';

	/**
	 * @type {Uint8Array}
	 */
	let output;

	const manifest = {
		wasm: [{ path: plugin }]
	};

	// const manifest = {
	// 	wasm: [{ path: 'https://raw.githubusercontent.com/extism/extism/main/wasm/code.wasm' }]
	// };

	const ctx = new ExtismContext();

	/**
	 * @type {(arg0: string) => void}
	 */
	let count;
	/**
	 * @type {string}
	 */
	let value;

	onMount(async () => {
		/**
		 * @type {Record<string, any>}
		 */
		const functions = {
			hello_world: function (/** @type {number} */ index) {
				//  + this.allocator.getString(index)
				console.log('Hello world!');
				return index;
			}
		};
		/**
		 * @type {Map<string, string>}
		 */
		const config = new Map([['thing', 'my value']]);
		const plugin = await ctx.newPlugin(manifest, functions, config);

		let funcs = Object.keys(await plugin.getExports());
		console.log('funcs ', funcs);

		count = async function (/** @type {string | Uint8Array} */ text) {
			output = await plugin.call('count_vowels', text);
		};

		count('this is a test');
	});
</script>

<!-- Input for text -->
<input type="text" bind:value on:keyup={(e) => count(value)} />

{#if output}
	{#await output}
		Loading...
	{:then output}
		<pre>{JSON.stringify(JSON.parse(new TextDecoder().decode(output)), null, 2)}</pre>
	{/await}
{/if}
