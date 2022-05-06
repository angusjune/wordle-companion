<script>
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();

    export let disabledKeys = [];

    const keys = [
        ["q", "w", "e", "r", "t", "y", "u", "i", "o", "p"],
        ["a", "s", "d", "f", "g", "h", "j", "k", "l"],
        ["enter", "z", "x", "c", "v", "b", "n", "m", "backspace"],
    ];
</script>

<style>
    .keyboard {
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 8px;
    }

    .row {
        width: 100%;
        display: flex;
        gap: 6px;
        justify-content: center;
    }

    .key {
        -webkit-appearance: none;
        text-transform: uppercase;
        color: #fff;
        font-size: 0.75em;
        font-weight: bold;
        width: 8%;
        min-width: 8%;
        line-height: 3;
        overflow: hidden;
        border-radius: 4px;
        background: #858886;
        border: 0;
        margin: 0;
        display: grid;
        place-items: center;
    }

    .key.action {
        width: auto;
    }

    .key.disabled {
        background: #3a3a3c;
    }
</style>

<div class="keyboard">
{#each keys as row}
    <div class="row">
        {#each row as key}
            <button 
                class="key"
                class:action={!key.match(/^[a-z]$/i)} 
                class:disabled={disabledKeys.indexOf(key) > -1}
                on:click={()=>{dispatch('input', { key })}}
            >
                {#if key == 'backspace'}
                <span class="material-symbols-outlined">backspace</span>
                {:else}
                {key}
                {/if}
            </button>
        {/each}
    </div>
{/each}
</div>
