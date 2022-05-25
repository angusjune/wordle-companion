<script context="module">
    const audioContext = new (window.AudioContext || window.webkitAudioContext)();
</script>

<script>
    import { fade, scale } from 'svelte/transition';
    import Mask from './Mask.svelte';

    export let show = false;
    export let word = '';

    let transition = {duration: 200};

    let dicts = [];
    let loading = false;

    let audio;
    let audioSrc;
    let audioLoading = false;

    async function dictionaryLookUp(query) {
        const ENDPOINT = 'https://api.dictionaryapi.dev/api/v2/entries/en/';

        loading = true;

        const response = await fetch(ENDPOINT + query);

        loading = false;

        dicts = await response.json();
    }

    function clickPlayAudio(src) {
        audioSrc = src;
        audioLoading = true;

        fetch(src)
        .then(response => response.arrayBuffer())
        .then(arrayBuffer => audioContext.decodeAudioData(arrayBuffer))
        .then(decodedAudio => {
            audio = decodedAudio;

            const playSound = audioContext.createBufferSource();
            playSound.buffer = audio;
            playSound.connect(audioContext.destination);
            playSound.start();

            audioLoading = false;
        })
        .catch(e => {
            console.error(e);
        });
    }

    $: if (word) { dictionaryLookUp(word) };
</script>

{#if show}
<Mask {show} {transition} on:click={()=>show = false} />
<div role="dialog" class="modal" transition:fade={transition}>
    <div class="modal__header">
        <div class="modal__close" role="button" on:click={()=> show = false}>
            <span class="material-symbols-outlined">close</span>
        </div>
    </div>

    <div class="modal__content">
        <div class="dict">

            {#if loading}
            <div class="loader" transition:scale={{duration:100}}></div>
            {:else}

            {#if Array.isArray(dicts)}
            {#each dicts as dict, dictIndex}
                <header class="dict__header">
                    <h1 class="dict__word">{dict.word}<sup>{dictIndex + 1}</sup></h1>
                    {#if dict.phonetics}
                    <div class="dict__phonetics">
                        {#each dict.phonetics as phonetic}

                        {#if phonetic.text}
                        <span class="dict__pron">{phonetic.text}</span>

                        {#if phonetic.audio}
                        <button 
                            class="dict__pron-btn" 
                            class:loading={audioLoading && audioSrc === phonetic.audio} 
                            on:click={()=>clickPlayAudio(phonetic.audio)}>
                            <span class="material-symbols-outlined">volume_up</span>
                        </button>
                        {/if}

                        {/if}

                        {/each}
                    </div>
                    {/if}
                </header>

                {#if dict.meanings}
                {#each dict.meanings as meaning}

                <div class="dict__meaning">
                    <p class="dict__pos">{meaning.partOfSpeech}</p>
                    <ol class="dict__def-list">
                    {#each meaning.definitions as def}
                        <li class="dict__def-list__item">{def.definition}</li>
                    {/each}
                    </ol>
                </div>

                {/each}

                {/if}
            {/each}
            {:else}

            <div class="tip"> No definations found</div>

            {/if}

            {/if}

        </div>
    </div>
        
</div>
{/if}

<style scoped lang="scss">
    @use './variables';
    @use './button';
    @use './loader' as *;

    @include loader($color: variables.$text-primary, $border: .3em);

    .loader {
        margin: auto;
    }

    .modal {
        display: flex;
        flex-direction: column;
        position: fixed;
        top: 40%;
        transform: translateY(-40%);
        border: 0;
        background: #323232;
        border-radius: 4px;
        color: #fff;
        width: 85%;
        max-width: 600px;
        max-height: 80%;
        overflow: hidden;
        padding: 0;
        inset-inline-start: 0;
        inset-inline-end: 0;
        margin: auto;
    }

    .modal__header {
        width: 100%;
        padding: 0.5rem 0.5rem 0;
        display: flex;
        justify-content: flex-end;
        box-sizing: border-box;
    }

    .modal__close {
        width: 2rem;
        height: 2rem;
        border-radius: 50%;
        background: #323232;
        display: grid;
        place-items: center;
    }

    .modal__content {
        flex: 1;
        overflow-y: auto;
    }

    .dict {
		padding: 0 1.5em 2.5em;

        &__header {
            display: flex;
            gap: 0.5em;
            align-items: baseline;
            margin-top: 1.5em;

            &:first-of-type {
                margin-top: 0;
            }
        }

        &__word {
            font-size: 1.5em;
            font-weight: normal;
            sup {
                opacity: 0.5;
                font-size: 0.5em;
            }
        }

        &__phonetics {
            display: flex;
            gap: 0.5em;
            align-items: center;
            flex-wrap: wrap;
        }

        &__pron {
            opacity: 0.6;
        }

        &__pron-btn {
            display: flex;
            color: var(--text-primary);
            opacity: 0.6;
            background: transparent;
            border: 0;
            padding: 0;
            margin: 0;

            * {
                font-size: 1em;
            }

            &.loading {
                animation: shake 0.5s ease-in-out infinite;
            }
        }

        &__meaning {
            padding-top: 1em;
        }

        &__pos {
            margin: 0;
        }

        &__def-list {
            font-size: 0.85em;
            list-style: none;
            counter-reset: counter;
            margin: 0;
            padding-left: 3em;

            &__item {
                counter-increment: counter;
                margin: 0.5em 0;
                position: relative;

                &:before {
                    content: counter(counter);
                    opacity: 0.5;
                    font-weight: bold;
                    position: absolute;
                    left: -3em;
                    min-width: 2em;
                    text-align: right;
                }
            }
        }
	}

    @keyframes shake {
        0%, 100% {
            transform: rotate(0deg);
        }
        50% {
            transform: rotate(-30deg);
        }
    }

</style>