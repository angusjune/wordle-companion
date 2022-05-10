<script>
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();

    export let excluded  = [];
    export let confirmed = [];
    export let maybe = [];

    const keys = [
        ["q", "w", "e", "r", "t", "y", "u", "i", "o", "p"],
        ["a", "s", "d", "f", "g", "h", "j", "k", "l"],
        ["enter", "z", "x", "c", "v", "b", "n", "m", "backspace"],
    ];
</script>

<style scoped lang="scss">
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
        --key-line-height: 3.5;

        -webkit-appearance: none;
        text-transform: uppercase;
        color: #fff;
        font-size: 0.75em;
        font-weight: bold;
        flex: 1;
        max-width: calc((100% - 54px) / 10);
        line-height: var(--key-line-height);
        overflow: hidden;
        border-radius: 4px;
        background: #858886;
        border: 0;
        margin: 0;
        display: grid;
        place-items: center;
        touch-action: manipulation;

        &:active {
            background: #a0a4a2;
        }

        &.action {
            max-width: none;
        }

        &.excluded {
            background: #3a3a3c;
        }

        &.maybe {
            background: var(--yellow);
        }

        &.confirmed {
            background: var(--green);
        }
    }

    @media (min-width: 640px) {
		.key {
            --key-line-height: 2.5;
        }
	}
</style>

<div class="keyboard">
{#each keys as row}
    <div class="row">
        {#each row as key}
            <button 
                class="key"
                class:action={!key.match(/^[a-z]$/i)} 
                class:excluded={excluded.indexOf(key) > -1}
                class:maybe={maybe.indexOf(key) > -1}
                class:confirmed={confirmed.indexOf(key) > -1}
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
