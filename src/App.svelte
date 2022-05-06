<script>
	import { tick } from 'svelte';
	import { fade } from 'svelte/transition';
	import Keyboard from './Keyboard.svelte';
	
	const AT_LEAST = 2;

	let value = ['?', '?', '?', '?', '?'];

    let cursor = 0;

	let showKb  = true;
	let loading = false;
	let showToast = false;
	let toastMsg = '';

	let candidates = [];
	let pronouns = [];
	let defs = [];

	// how many values in array is not '?'
	function countNonEmpty() {
		let count = 0;
		for (let i = 0; i < value.length; i++) {
			if (value[i] !== '?') {
				count++;
			}
		}
		return count;
	}

	function onKbInput(e) {
		let {value: keyVal} = e.detail;
		keyVal = keyVal.toLowerCase();

		if (keyVal === 'backspace') {
			value[cursor] = '?';
		} else if (keyVal === 'enter') {
			if (countNonEmpty() >= AT_LEAST) {
				showKb = false;
				fetchData();
			} else {

			}
		} else if (keyVal.match(/^[a-z]$/)) {
			value[cursor] = keyVal;
		}
	}

	function onKeyPress(e){
		onKbInput({detail: {value: e.key}});
	}

	function toast(msg, lastsFor=2000) {
		showToast = true;
		toastMsg  = msg;
		setTimeout(() => {
			showToast = false;
		}, lastsFor);
	}

	async function fetchData() {
		const ENDPOINT = 'https://api.datamuse.com/words';
		const query = '?sp=' + value.join('') + '&md=dr&ipa=1';

		loading = true;

		const response = await fetch(ENDPOINT + query);
		const data = await response.json();
		candidates = data;
		// candidates = data.map(item => item.word);
		// pronouns = data.map(item => item.tags.ipa_pron);
		// defs = data.map(item => item.defs);

		await tick();
		loading = false;
		console.log(candidates);
	}
</script>

<svelte:body on:keydown={onKeyPress} on:touchstart={()=>{}} />

<div class="container">
	<header class="header">
		<h1 class="header__title">Wordle Companion</h1>
	</header>

	<main class="main">
		<div class="char-wrap" on:click={()=>{showKb=true}}>
			{#each value as char, index}
			<div class="char" class:filled={value[index] != '?'} class:focus={cursor == index} on:click={()=>{cursor = index}}>{char == '?' ? '' : char}</div>
			{/each}
		</div>
	
		<div class="word-wrap">
			{#each candidates as candidate}
			<div class="word">{candidate.word}</div>
			{/each}
		</div>
	</main>
	
	{#if showKb}
	<div class="kb-wrap" transition:fade="{{ duration: 200 }}">
		<Keyboard on:input={onKbInput} />
	</div>
	{/if}
</div>


<style>
	@import url('https://fonts.googleapis.com/css2?family=Koulen&display=swap');

	:global(body) {
		--max-width: 90%;

		--background: #121214;
		--green: #528c4e;
		--border: #3a3a3c;

		background: var(--background);
	}

	.container {
		display: flex;
		flex-direction: column;
		height: 100vh;
		margin: 0 auto;
		max-width: var(--max-width);
	}

	.header {
		font-family: 'Koulen', cursive;
		color: #fff;
		padding: 0.5rem 0;
	}

	.header__title {
		margin: 0;
		font-size: 1.5rem;
		text-align: center;
	}

	.main {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;		
	}

	.char-wrap {
		display: flex;
		gap: 0.25rem;
		padding: 2rem;
	}

	.char {
		box-sizing: border-box;
		text-transform: uppercase;
		display: grid;
		place-items: center;
		border: 2px solid var(--border);
		color: #fff;
		font-size: 1.25rem;
		font-weight: bold;
		width: 3.5rem;
		height: 4rem;
	}

	.char.focus {
		background: rgba(255,255,255,0.1);
	}

	.char.filled {
		background: var(--green);
		border: 0;
	}

	.word-wrap {
		width: 100%;
		display: flex;
		flex-wrap: wrap;
		gap: 0.5rem;
		justify-content: center;
		padding: 2rem 0;
	}

	.word {
		color: rgba(255,255,255,0.6);
		background: #3a3a3c;
		border-radius: 4px;
		padding: 0.25rem;
	}

	.kb-wrap {
		position: sticky;
		bottom: 0;
		left: 0;
		right: 0;
		z-index: 1;
		padding: 1rem 0 env(safe-area-offset-bottom, 2rem);
		background: rgba(18,18,20,0.75);
		backdrop-filter: blur(10px);
	}

	/* .kb-wrap.hidden {
		transform: translateY(100%);
	} */

	@media (min-width: 640px) {
		:global(body) {
			--max-width: 640px;
		}
	}
</style>