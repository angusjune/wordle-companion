<script>
    import { fade } from 'svelte/transition';

    export let show = false;
</script>

{#if show}
<div class="mask" transition:fade={{duration: 300}} on:touchstart|capture={()=>{}}></div>
{/if}

<style>
    .mask {
        position: fixed;
        top: 0;
        right: 0;
        bottom: 0;
        left: 0;
        background: rgba(0,0,0,.7);
        -webkit-backdrop-filter: blur(10px);
        backdrop-filter: blur(10px);
    }
</style>