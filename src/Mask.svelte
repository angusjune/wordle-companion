<script>
    import { createEventDispatcher } from 'svelte';
    import { fade } from 'svelte/transition';

    const dispatch = createEventDispatcher();

    export let show = false;
    export let transition = {duration: 300};
</script>

{#if show}
<div class="mask" transition:fade={transition} on:touchstart|stopPropagation={()=>{}} on:click={()=>dispatch('click')}></div>
{/if}

<style>
    .mask {
        position: fixed;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        background: rgba(18,18,20,0.7);
        -webkit-backdrop-filter: blur(10px);
        backdrop-filter: blur(10px);
    }
</style>