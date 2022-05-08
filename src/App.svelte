<script>
	import { tick } from 'svelte';
	import { fade, slide, scale } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';
	import Keyboard from './Keyboard.svelte';
	import Mask from './Mask.svelte';
	import Toast from './Toast.svelte';
	import Modal from './Modal.svelte';
	
	const AT_LEAST = 1;

	let value = ['?', '?', '?', '?', '?'];

	let filteredChars  = [];

    let cursor = 0;

	let showKb  = true;
	let loading = false;
	let showToast = false;
	let showModal = false;
	let showMask = false;
	let showNoSearchResult = false;
	let enteringFilteredChars = false;
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
		showNoSearchResult = false;

		if (!enteringFilteredChars) {
			if (keyVal === 'backspace') {
				value[cursor] = '?';
			} else if (keyVal === 'enter') {
				if (countNonEmpty() >= AT_LEAST) {
					showKb = false;
					fetchData();
				} else {
					toastOn('At least ' + AT_LEAST + ' letter' + (AT_LEAST> 1 ? 's' : ''));
				}
			} else if (keyVal.match(/^[a-z]$/i)) {
				filteredCandidates = [];
				value[cursor] = keyVal;
			}
		} else {
			if (keyVal === 'enter') {
				enteringFilteredChars = false;
			} else if (keyVal.match(/^[a-z]$/i) && value.indexOf(keyVal) < 0) {
				const index = filteredChars.indexOf(key);
				if (index < 0) {
					// key does not exists in list
					filteredChars = [...filteredChars, key];
				} else {
					filteredChars.splice(index, 1);
					filteredChars = filteredChars;
				}
			}
		}
	}

	function onKeyPress(e){
		onKbInput({detail: { key: e.key }});
	}

	function onClickEnterFilteredChars() {
		enteringFilteredChars = !enteringFilteredChars;
		showKb = true;
	}

	let pronun = '';
	let word = '';
	let defs = [];

	function onClickWord(item) {
		pronun = item.tags[1].replace('ipa_pron:', '');
		word = item.word;
		defs = item.defs;

		showModal = true;
	}

	function toastOn(msg) {
		showToast = true;
		toastMsg  = msg;
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
		showNoSearchResult = true;
	}

	$:filteredCandidates = candidates.filter(item => (!filteredChars.some(v => item.word.includes(v))) && /^[a-z]+$/i.test(item.word));
	$:showMask = enteringFilteredChars;
	$:showMask = showModal;
	$:if (showModal) { showKb = false }
</script>

<svelte:body on:keydown={onKeyPress} on:touchstart={()=>{}} />

<div class="container">
	<header class="header">
		<h1 class="header__title">Wordle Companion</h1>
	</header>

	<main class="main">

		<div class="char-tip">
			Enter <span class="green">confirmed letters</span> in their spots
		</div>
		<div class="char-wrap" on:click={()=>{showKb=true}}>
			{#each value as char, index}
			<div class="char" class:filled={value[index] != '?'} class:focus={cursor == index} on:click={()=>{cursor = index}}>{char == '?' ? '' : char}</div>
			{/each}
		</div>
	
		<div class="word-wrap">
			{#if loading}
			<div class="loader">Loading...</div>

			{:else}

			{#each filteredCandidates as candidate}
			<div class="word" on:click={()=>onClickWord(candidate)}>{candidate.word}</div>
			{/each}

			{#if filteredCandidates.length < 1 && showNoSearchResult}
				<div class="tip large" transition:scale={{easing:quintOut, duration: 500}}>🤷‍♀️</div>
			{/if}

			{/if}
			
		</div>
	</main>

	<div class="actions">
		{#if !showModal}
		<div class="actions__buttons">
			<button class="btn" class:at-bottom={!showKb} on:click={onClickEnterFilteredChars}>{enteringFilteredChars ? 'Finish' : 'Select not-in-the-word letters'}</button>

			{#if !enteringFilteredChars}
			<button class="kb-toggle" on:click={()=> showKb = !showKb}>
				{#if showKb}
				<span class="material-symbols-outlined">keyboard_hide</span>
				{:else}
				<span class="material-symbols-outlined">keyboard</span>
				{/if}
			</button>
			{/if}
		</div>
		{/if}
	
		{#if showKb}
		<div class="kb-wrap" transition:slide={{ duration: 200 }}>
			<div class="kb-wrap__inner">
				<Keyboard on:input={onKbInput} {filteredChars} confirmedChars={value} />
			</div>
		</div>
		{/if}
	</div>
	
</div>

<Mask show={showMask} />

{#if enteringFilteredChars}
<div class="tip centered" transition:fade={{ duration: 200 }}>
	Select not-in-the-word letters from the keyboard
</div>
{/if}

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
		<p class="dict__def">{@html def }</p>
		{/each}
		{/if}
	</div>
</Modal>

<Toast bind:show={showToast}>{toastMsg}</Toast>


<style>
	@import url('https://fonts.googleapis.com/css2?family=Koulen&display=swap');

	:global(body) {
		--width: 90%;
		--kb-width: 96%;

		--background: #121214;
		--green: #528c4e;
		--char-border: #3a3a3c;		

		background: var(--background);
		font-size: 16px;
		overflow: hidden;
		-webkit-user-select: none;
		user-select: none;
	}

	.container {
		display: flex;
		flex-direction: column;
		height: 100%;
		overflow-x: hidden;
	}

	.header {
		font-family: 'Koulen', cursive;
		color: #fff;
		width: 100%;
		padding: 0.5em 0;
		position: sticky;
		top: 0;
		background: rgba(18,18,20,0.75);
		-webkit-backdrop-filter: blur(10px);
		backdrop-filter: blur(10px);
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

	.char-tip {
		margin-top: 2em;
		text-align: center;
		color: rgba(255,255,255,.6);
	}

	.green {
		color: var(--green);
	}

	.char-wrap {
		width: 100%;
		display: flex;
		justify-content: center;
		gap: 0.3em;
		padding: 1em 2em 2em;
	}

	.char {
		box-sizing: border-box;
		text-transform: uppercase;
		display: grid;
		place-items: center;
		border: 2px solid var(--char-border);
		color: #fff;
		font-size: 2em;
		font-weight: bold;
		width: 2em;
		height: 2.1em;
	}

	.char.focus {
		border-color: var(--green);
		background: rgba(82, 140, 78, 0.2);
	}

	.char.filled {
		background: var(--green);
		border-color: transparent;
	}

	.char.focus.filled {
		border-color: #21631C;
	}

	.tip {
		padding: 0 4em;
		font-size: 1.5em;
		color: #fff;
		text-align: center;
	}

	.tip.centered {
		position: fixed;
		top: 50%;
		transform: translateY(-50%);
	}

	.tip.large {
		font-size: 3em;
	}

	.word-wrap {
		width: 100%;
		display: flex;
		flex-wrap: wrap;
		gap: 0.5em;
		justify-content: center;
		padding: 2em 0;
		touch-action: manipulation;
	}

	.word {
		color: rgba(255,255,255,0.6);
		background: #3a3a3c;
		border-radius: 4px;
		padding: 0.25em;
	}

	.actions {
		position: sticky;
		bottom: 0;
		left: 0;
		right: 0;
		z-index: 1;
	}

	.kb-wrap {
		padding: 1em 0 env(safe-area-offset-bottom, 1em);
		background: rgba(18,18,20,0.75);
		-webkit-backdrop-filter: blur(10px);
		backdrop-filter: blur(10px);
	}

	.kb-wrap__inner {
		position: relative;
		width: var(--kb-width);
		margin: auto;
	}

	.kb-toggle {
		position: absolute;
		right: 0;
		width: 3em;
		height: 3em;
		display: none;
		place-items: center;
		color: rgba(255,255,255,.6);
		background: rgba(18,18,20,0.75);
		-webkit-backdrop-filter: blur(10px);
		backdrop-filter: blur(10px);
		border: 0;
		margin: 0;
		padding: 0;
	}

	.kb-toggle:active {
		background: #3a3a3c;
	}

	.actions__buttons {
		position: relative;
		display: flex;
		justify-content: center;
		margin: auto;
	}

	.btn {
		--border: 1px solid rgba(255,255,255,.06);
		background: rgba(18,18,20,0.75);
		-webkit-backdrop-filter: blur(10px);
		backdrop-filter: blur(10px);
		color: #fff;
		font-weight: bold;
		line-height: 1;
		border: 0;
		border-top: var(--border);
		border-bottom: var(--border);
		padding: 1em 0;
		margin: 0;
		z-index: 1;
		width: 100%;
		touch-action: manipulation;
	}

	.btn:active {
		background: #3a3a3c;
	}

	.btn.at-bottom {
		padding-bottom: env(safe-area-offset-bottom, 1em);
	}

	.dict {
		padding: 0 1.5em 2.5em;
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

	.dict__def:last-of-type {
		margin-bottom: 0;
	}

	.loader,
	.loader:after {
		border-radius: 50%;
		width: 2em;
		height: 2em;
	}
	.loader {
		margin: 2em auto;
		font-size: 10px;
		position: relative;
		text-indent: -9999em;
		border-top: .4em solid rgba(255, 255, 255, 0.2);
		border-right: .4em solid rgba(255, 255, 255, 0.2);
		border-bottom: .4em solid rgba(255, 255, 255, 0.2);
		border-left: .4em solid #ffffff;
		transform: translateZ(0);
		animation: load 1.1s infinite linear;
	}

	@keyframes load {
		from { transform: rotate(0deg); }
		to   { transform: rotate(360deg); }
	}


	@media (min-width: 340px) {
        .btn {
            width: fit-content;
			border: var(--border);
			margin: 0 auto 1em auto;
			padding: 1em 1.5em;
			border-radius: 4px;
        }
		.kb-toggle {
			display: grid;
		}

		.actions__buttons {
			width: var(--kb-width);
		}
    }


	@media (min-width: 640px) {
		:global(body) {
			--width: 640px;
			--kb-width: 640px;
			font-size: 18px;
		}
	}
</style>