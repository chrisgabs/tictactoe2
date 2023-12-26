<script>
    export let resetGame;
    export let leaveRoom;
    export let opponentName;
    export let displayName;
</script>

<div class="flex flex-col sm:py-24 gap-1 sm:justify-between mx-auto">
    <div class="flex justify-evenly sm:justify-start sm:flex-col gap-1 mb-1">
        <button on:click={resetGame} class="btn btn-outline btn-sm mb-3 hidden sm:block">New game</button>

        <div class="flex items-center justify-between gap-2">
            <span class="relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full bg-slate-300"></span>
            <div class="text-center">
                <h2 class="font-bold">{displayName}</h2>
                <div class="text-sm">1 Wins</div>
            </div>
        </div>

        {#if opponentName}
            <div class="flex items-center justify-between gap-2">
                <span class="relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full bg-slate-300"></span>
                <div class="text-center">
                    <h2 class="font-bold">{opponentName}</h2>
                    <div class="text-sm">1 Win</div>
                </div>
            </div>
        {:else}
            <div class="flex items-center justify-between gap-2">
                <span class="relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full bg-slate-300"></span>
                <div class="text-center">
                    <h2 class="font-bold">----</h2>
                    <div class="text-sm">----</div>
                </div>
            </div>
        {/if}

    </div>

    <button on:click={resetGame} class="btn btn-outline btn-sm w-56  sm:w-12 sm:hidden">New game</button>
    <button on:click={leaveRoom} class="btn btn-outline btn-error w-56 sm:w-min whitespace-nowrap btn-sm">Leave Game</button>
</div>
