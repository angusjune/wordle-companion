<script>
    import { createEventDispatcher } from 'svelte';
    import { scale } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';

    const dispatch = createEventDispatcher();

    export let words = [];
    export let loading = false;
    export let showNoSearchResult = false;
</script>

<section class="word-list" role="list">
    {#each words as word, index}
    <div class="word" role="listitem" tabindex="0" on:click={()=>dispatch('click', {index})}>
        {word.word}
    </div>
    {/each}

    {#if loading}
    <div class="loader word-list__loader" transition:scale></div>
    {/if}

    {#if words.length < 1 && showNoSearchResult}
        <div class="word-list__empty" transition:scale={{easing:quintOut, duration: 500}} aria-label="no search result">🤷‍♀️</div>
    {/if}
</section>

<style scoped lang="scss">
    @use './variables';
    @use './loader' as *;

    @include loader();

    .word-list {
        display: flex;
        flex-wrap: wrap;
        justify-content: center;
        gap: 0.4em;

        &__empty {
            margin-bottom: 2em 0;
            font-size: 3em;
        }

        &__loader {
            margin-top: 2em;
        }
    }

    .word {
        background: darken(variables.$surface, 10%);
        color: lighten(variables.$text-primary, 10%);
        padding: 0.25em 0.5em;
        border-radius: 4px;
        cursor: pointer;
        touch-action: manipulation;

        &:acitve {
            filter: brightness(1.1);
        }
    }
</style>