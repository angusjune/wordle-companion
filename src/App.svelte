<script>
	import { tick } from 'svelte';
	import { fade, slide } from 'svelte/transition';
	import LetterBox from './LetterBox.svelte';
	import WordList from './WordList.svelte';
	import Keyboard from './Keyboard.svelte';
	import Mask from './Mask.svelte';
	import Toast from './Toast.svelte';
	import DictModal from './DictModal.svelte';
	
	const AT_LEAST = 1;

	let letters   = [{value: '?'},{value: '?'},{value: '?'},{value: '?'},{value: '?'}];
	let confirmed = [];
	let maybe     = [];
	let excluded  = [];

	$:if (letters) {
		confirmed = [];
		maybe = [];
		letters.forEach(({value, status}, index) => {
			if (status) {
				if (status === 'confirmed') {
					confirmed[index] = value;
				} else if (status === 'maybe') {
					maybe = [...maybe, value];
					confirmed[index] = '?';
				}
			} else {
				confirmed[index] = '?';
			}
		});
	}

    let cursor = 0;

	let showKb  = true;
	let loading = false;
	let showToast = false;
	let showModal = false;
	let showNoSearchResult = false;
	let enteringExcludedLetters = false;
	let enteringMaybeLetters = false;
	let toastMsg = '';

	let candidates = [];

	$:focusTheme = enteringMaybeLetters ? '#b59e39' : '#528c4e';

	// how many values in array is not '?'
	function countNonEmpty() {
		let count = 0;
		for (const {value} of letters) {
			if (value !== '?') {
				count++;
			}
		}
		return count;
	}

	function onKeyboardInput(e) {
		const { key } = e.detail;
		const keyVal = key.toLowerCase();
		showNoSearchResult = false;

		if (!enteringExcludedLetters) {
			// entering confirmed or maybe letters

			if (keyVal === 'backspace') {
				letters[cursor] = {
					value: '?',
					status: '',
				};
			} else if (keyVal === 'enter') {
				if (countNonEmpty() >= AT_LEAST) {
					fetchData();
				} else {
					toastOn('At least ' + AT_LEAST + ' letter' + (AT_LEAST> 1 ? 's' : ''));
				}
			} else if (keyVal.match(/^[a-z]$/i)) {
				candidates = [];
				letters[cursor] = {
					value: keyVal,
					status: enteringMaybeLetters ? 'maybe' : 'confirmed',
				};
			}

		} else {

			if (keyVal === 'enter') {
				// done inputting excluded letters
				enteringExcludedLetters = false;
				if (!enteringExcludedLetters && countNonEmpty() >= AT_LEAST) {
					fetchData();
				}

			} else if (keyVal.match(/^[a-z]$/i) && letters.findIndex(letter => letter.value === keyVal) < 0) {
				// add excluded letters if letter is not confirmed or maybe
				const index = excluded.indexOf(keyVal);
				if (index < 0) {
					// key does not exists in list, add it
					excluded = [...excluded, keyVal];
				} else {
					// key exists in list, remove it
					excluded.splice(index, 1);
					excluded = excluded;
				}
			}
		}
	}

	function onKeyPress(e){
		onKeyboardInput({detail: { key: e.key }});
	}

	function onClickEnterFilteredChars() {
		enteringExcludedLetters = !enteringExcludedLetters;
		if (!enteringExcludedLetters && countNonEmpty() >= AT_LEAST) {
			fetchData();
		}
		showKb = true;
	}

	let dict = {
		word: '',
		pron: '',
		defs: [],
	};

	async function onClickWord(e) {
		const { index } = e.detail;

		const word = candidates[index];

		await tick();
		dict.word = word.word;
		dict.pron = word.tags[word.tags.findIndex(value => value.indexOf('ipa_pron') >= 0)].replace('ipa_pron:', '');
		dict.defs = word.defs;

		showModal = true;
	}

	function toastOn(msg) {
		showToast = true;
		toastMsg  = msg;
	}

	async function fetchData() {
		const ENDPOINT = 'https://api.datamuse.com/words';

		let confirmedQueryStr = confirmed.join('');
		let maybeQueryStr     = maybe.length > 0 ? `,*${maybe.join('*')}*` : '';
		let excludedQueryStr  = excluded.length > 0 ? `-${excluded.join('')}` : '';

		console.log(`${confirmedQueryStr}${excludedQueryStr}${maybeQueryStr}`);

		const query = '?sp=' + encodeURIComponent(`${confirmedQueryStr}${excludedQueryStr}${maybeQueryStr}`) + '&md=dr&ipa=1&max=30';

		loading = true;

		const response = await fetch(ENDPOINT + query);
		const data = await response.json();
		candidates = data;

		await tick();
		loading = false;
		showNoSearchResult = true;
	}
</script>

<svelte:body on:keydown={onKeyPress} on:touchstart={()=>{}} />

<div class="container">
	<header class="header">
		<h1 class="header__title">Wordle Companion</h1>
	</header>

	<main class="main">

		<div class="letterbox">
			<div class="letterbox__header">
				<div class="letterbox__label">
					{#if !enteringMaybeLetters}
					Enter <span class="green">confirmed</span> letters
					{:else}
					Enter <span class="yellow">unconfirmed</span> letters
					{/if}
				</div>

				<button class="btn btn--icon btn--switch" on:click={()=>enteringMaybeLetters=!enteringMaybeLetters}>
					<svg viewBox="0 0 48 48" fill="none" xmlns="http://www.w3.org/2000/svg">
						<path d="M13.65 40L4 30.35L13.65 20.7L15.75 22.8L9.7 28.85H25.5V31.85H9.7L15.75 37.9L13.65 40Z" fill="#B59E39"/>
						<path d="M32.25 25.2L34.35 27.3L44 17.65L34.35 8L32.25 10.1L38.3 16.15H22.5V19.15H38.3L32.25 25.2Z" fill="#5B9657"/>
					</svg>				
				</button>
			</div>

			<LetterBox --focus-theme={focusTheme} {letters} bind:cursor />
		</div>

		<div class="wordlist">
			<WordList {loading} {showNoSearchResult} words={candidates} on:click={onClickWord} />
		</div>
	</main>

	{#if enteringExcludedLetters}
	<Mask show={enteringExcludedLetters} />
	<div class="tip centered" transition:fade={{ duration: 200 }}>
		Select not-in-the-word letters from the keyboard
	</div>
	{/if}

	<div class="actions">
		{#if !showModal}
		<div class="actions__buttons">
			<button class="btn btn--enter-excluded" on:click={onClickEnterFilteredChars}>{enteringExcludedLetters ? 'Finish' : 'Select not-in-the-word letters'}</button>

			{#if !enteringExcludedLetters}
			<button class="btn btn--icon btn--kb-toggle" on:click={()=> showKb = !showKb}>
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
		<div class="actions__keyboard" transition:slide={{ duration: 200 }}>
			<div class="actions__keyboard-inner">
				<Keyboard on:input={onKeyboardInput} {excluded} {confirmed} {maybe} />
			</div>
		</div>
		{/if}
	</div>
	
</div>

<DictModal bind:show={showModal} {...dict} />

<Toast bind:show={showToast}>{toastMsg}</Toast>


<style lang="scss">
	@use './variables';
	@use './button';

	@import url('https://fonts.googleapis.com/css2?family=Koulen&display=swap');

	:global(body) {
		--char-border: #3a3a3c;		

		background: var(--background);
		color: var(--text-primary);
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
		@include variables.blurredSurface();
		font-family: 'Koulen', cursive;
		width: 100%;
		padding: 0.5em 0;
		position: sticky;
		top: 0;

		&__title {
			margin: 0;
			font-size: 1.5em;
			text-align: center;
		}
	}

	.main {
		flex: 1;
		display: flex;
		flex-direction: column;
		align-items: center;		
		width: 90%;
		max-width: 640px;
		margin: 0 auto;
	}

	.letterbox {
		width: 100%;
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 1em 0 2em;

		&__header {
			display: flex;
			gap: 0.5em;
			justify-content: center;
			align-items: center;
			padding-bottom: 1em;
		}

		&__label {
			color: rgba(255,255,255,.6);

			.green {
				color: var(--green);
			}

			.yellow {
				color: var(--yellow);
			}
		}
	}

	.wordlist {
		padding-bottom: 2em;
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

	.actions {
		position: sticky;
		bottom: 0;
		left: 0;
		right: 0;

		&__buttons {
			position: relative;
			display: flex;
			justify-content: center;
			margin: auto;
			width: 96%;
			max-width: 640px;
			padding-bottom: env(safe-area-offset-bottom, 1em);
		}

		&__keyboard {
			@include variables.blurredSurface();
			padding: 1em 0 env(safe-area-offset-bottom, 1em);
		}

		&__keyboard-inner {
			position: relative;
			width: 96%;
			max-width: 640px;
			margin: auto;
		}
	}

	.btn {
		&--switch {
			@include button.with-icon(2em);
			border: none;
		}

		&--kb-toggle {
			border: none;
			position: absolute;
			right: 0;
		}
	}

	@media (max-width: 340px) {
		:global(body) {
			font-size: 12px;
		}
	}

	@media (min-width: 640px) {
		:global(body) {
			font-size: 18px;
		}
	}
</style>