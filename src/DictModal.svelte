<script>
    import { fade } from 'svelte/transition';
    import Mask from './Mask.svelte';

    export let show = false;
    export let word = '';
    export let pron = '';
    export let defs = [];

    let transition = {duration: 200};
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
            <header class="dict__header">
                <span class="dict__word">{word}</span>
                {#if pron}
                <span class="dict__pron">|{pron}|</span>
                {/if}
            </header>
            {#if defs}
            {#each defs as def}
            <p class="dict__def">{@html def }</p>
            {/each}
            {/if}
        </div>
    </div>
        
</div>
{/if}

<style scoped lang="scss">
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
            align-items: center;
        }

        &__word {
            font-size: 1.5em;
            font-weight: bold;
        }

        &__pron {
            opacity: 0.6;
        }

        &__def {
            margin: 1em 0;

            &:last-of-type {
                margin-bottom: 0;
            }
        }
	}

</style>