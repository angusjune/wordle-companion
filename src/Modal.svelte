<script>
    import { fade } from 'svelte/transition';

    export let show = false;
</script>

{#if show}
<div role="dialog" class="modal" transition:fade={{duration: 300}}>
    <div class="modal__header">
        <div class="modal__close" role="button" on:click={()=> show = false}>
            <span class="material-symbols-outlined">close</span>
        </div>
    </div>

    <div class="modal__content">
        <slot></slot>
    </div>
        
</div>
{/if}

<style>
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
        width: var(--width);
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

</style>