<script>
	import { tick } from 'svelte';
	import { fade } from 'svelte/transition';
	import Keyboard from './Keyboard.svelte';
	import Mask from './Mask.svelte';
	import Toast from './Toast.svelte';
	import Modal from './Modal.svelte';
	
	const AT_LEAST = 1;

	let value = ['?', '?', '?', '?', '?'];

	let disabledKeys = [];

    let cursor = 0;

	let showKb  = true;
	let loading = false;
	let showToast = false;
	let showModal = false;
	let showMask = false;
	let enteringDisabledKeys = false;
	let toastMsg = '';

	let candidates = [];
	let filteredCandidates = [];

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
		const { key } = e.detail;
		const keyVal = key.toLowerCase();

		if (!enteringDisabledKeys) {
			if (keyVal === 'backspace') {
				value[cursor] = '?';
			} else if (keyVal === 'enter') {
				if (countNonEmpty() >= AT_LEAST) {
					showKb = false;
					fetchData();
				} else {

				}
			} else if (keyVal.match(/^[a-z]$/i)) {
				value[cursor] = keyVal;
			}
		} else {
			if (keyVal === 'enter') {
				enteringDisabledKeys = false;
			} else if (keyVal.match(/^[a-z]$/i)) {
				const index = disabledKeys.indexOf(key);
				if (index < 0) {
					// key does not exists in list
					disabledKeys = [...disabledKeys, key];
				} else {
					disabledKeys.splice(index, 1);
					disabledKeys = disabledKeys;
				}
			}
		}
	}

	function onKeyPress(e){
		onKbInput({detail: { key: e.key }});
	}

	function onClickEnterDisabledKeys() {
		enteringDisabledKeys = !enteringDisabledKeys;
		showKb = true;
	}

	let pronun = '';
	let word = '';
	let defs = [];

	function onClickWord(item) {
		console.log(item);
		// const { tags } = item;

		pronun = item.tags[1].replace('ipa_pron:', '');
		word = item.word;
		defs = item.defs;

		showModal = true;
	}

	function toastOn(msg, lastsFor=2000) {
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

		await tick();
		loading = false;
	}

	$:filteredCandidates = candidates.filter(item => !disabledKeys.some(v => item.word.includes(v)))
	$:console.log(filteredCandidates);
	$:showMask = enteringDisabledKeys;
	$:showMask = showModal;
	$:showKb = !showModal;
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
			{#each filteredCandidates as candidate}
			<div class="word" on:click={()=>onClickWord(candidate)}>{candidate.word}</div>
			{/each}
		</div>
	</main>

	{#if !showModal}
	<button class="btn" class:at-bottom={!showKb} on:click={onClickEnterDisabledKeys}>{enteringDisabledKeys ? 'Done' : 'Select not-in-the-word letters'}</button>
	{/if}

	{#if showKb}
	<div class="kb-wrap" transition:fade="{{ duration: 200 }}">
		<Keyboard on:input={onKbInput} {disabledKeys} />
	</div>
	{/if}
</div>

<Mask show={showMask} />

<Modal bind:show={showModal}>
	<div class="dict">
		<header class="dict__header">
			<span class="dict__word">{word}</span>
			{#if pronun}
			<span class="dict__pronun">|{pronun}|</span>
			{/if}
		</header>
		{#if defs}
		{#each defs as def}
		<p class="dict_def">{@html def }</p>
		{/each}
		{/if}
	</div>
</Modal>


<style>
	@import url('https://fonts.googleapis.com/css2?family=Koulen&display=swap');

	:global(body) {
		--width: 90%;

		--background: #121214;
		--green: #528c4e;
		--border: #3a3a3c;

		background: var(--background);
		font-size: 16px;
		user-select: none;
	}

	.container {
		display: flex;
		flex-direction: column;
		height: 100vh;
	}

	.header {
		font-family: 'Koulen', cursive;
		color: #fff;
		width: 100%;
		padding: 0.5em 0;
	}

	.header__title {
		margin: 0;
		font-size: 1.5em;
		text-align: center;
	}

	.main {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;		
		width: var(--width);
		margin: 0 auto;
	}

	.char-wrap {
		width: 100%;
		display: flex;
		justify-content: center;
		gap: 0.25em;
		padding: 2em;
	}

	.char {
		box-sizing: border-box;
		text-transform: uppercase;
		display: grid;
		place-items: center;
		border: 2px solid var(--border);
		color: #fff;
		font-size: 1.25em;
		font-weight: bold;
		width: 20%;
		max-width: 3.5em;
		height: 4em;
	}

	.char.focus {
		border-color: #aaa;
		background: rgba(255,255,255,0.05);
	}

	.char.filled {
		background: var(--green);
		border-color: transparent;
	}

	.char.filled.focus {
		border-color: rgba(255,255,255,.4);
	}

	.word-wrap {
		width: 100%;
		display: flex;
		flex-wrap: wrap;
		gap: 0.5em;
		justify-content: center;
		padding: 2em 0;
	}

	.word {
		color: rgba(255,255,255,0.6);
		background: #3a3a3c;
		border-radius: 4px;
		padding: 0.25em;
	}

	.kb-wrap {
		position: sticky;
		bottom: 0;
		left: 0;
		right: 0;
		z-index: 1;
		padding: 1em 0 env(safe-area-offset-bottom, 1em);
		background: rgba(18,18,20,0.75);
		-webkit-backdrop-filter: blur(10px);
		backdrop-filter: blur(10px);
	}

	.btn {
		--border: 0.5px solid rgba(255,255,255,.06);
		background: var(--background);
		color: #fff;
		border: 0;
		border-top: var(--border);
		border-bottom: var(--border);
		padding: 1em 0;
		margin: 0;
		z-index: 1;
	}

	.btn:active {
		background: #3a3a3c;
	}

	.btn.at-bottom {
		padding-bottom: env(safe-area-offset-bottom, 1em);
	}

	.dict {
		padding: 0 1.5em 1.5em;
	}

	.dict__header {
		display: flex;
		gap: 1em;
		align-items: center;
	}

	.dict__word {
		font-size: 1.5em;
		font-weight: bold;
	}

	.dict__pronun {
		opacity: 0.6;
	}

	.dict__def {
	}


	@media (min-width: 640px) {
		:global(body) {
			--width: 640px;
			font-size: 18px;
		}
	}
</style>