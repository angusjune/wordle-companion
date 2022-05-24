<script>
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();

    export let letters = [{value: '?'},{value: '?'},{value: '?'},{value: '?'},{value: '?'}];
    export let cursor = 0;

    function onClick(index) {
        cursor = index;
        dispatch('click', { cursor });
    }
</script>

<div class="letters">
    {#each letters as letter, index}
    <div 
        class="letters__letter"
        class:confirmed={letter.status === 'confirmed'}
        class:maybe={letter.status === 'maybe'}
        class:focus={cursor === index}
        tabindex="0"
        on:click={() => onClick(index)}
        on:focus={() => onClick(index)}
    >
        {letter.value === '?' ? '' : letter.value}
    </div>
    {/each}
</div>

<style scoped lang="scss">
    .letters {
        --focus-color: var(--focus-theme, #528c4e);
        --box-border: #3a3a3c;

        display: flex;
		gap: 0.3em;
        max-width: 100%;

        &__letter {
            position: relative;
            box-sizing: border-box;
            text-transform: uppercase;
            display: grid;
            place-items: center;
            border: 2px solid var(--box-border);
            color: #fff;
            font-size: 2em;
            font-weight: bold;
            width: 4rem;
            height: 4.2rem;
            border-radius: 1px;;
            cursor: text;
            touch-action: manipulation;

            &:before {
                content: '';
                position: absolute;
                left: 0;
                right: 0;
                top: 0;
                bottom: 0;
            }
            
            &.focus, &:focus {
                outline: 0;
                border-color: var(--focus-color);

                &:before {
                    transition: background 0.15s;
                    background: var(--focus-color);
                    opacity: 0.2;
                }
            }

            &.confirmed, &.maybe {
                border-color: transparent;

                &.focus, &:focus{
                    border-color: rgba(0,0,0,.25);
                }
            }

            &.confirmed {
                background: var(--green);
            }

            &.maybe {
                background: var(--yellow);
            }
        }
    }
</style>