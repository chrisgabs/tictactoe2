<script>
    export let resetGame;
    export let leaveRoom;
    export let opponentName;
    export let displayName;
</script>

<div class="flex flex-col sm:py-24 gap-2 items-center sm:justify-between mx-auto">
    <div class="flex justify-evenly sm:justify-start sm:flex-col gap-1 mb-1 w-full px-1">
        <button on:click={resetGame} class="btn btn-outline btn-sm mb-3 hidden sm:block sm:w-full">New game</button>

        <div class="flex items-center justify-start gap-2">
            <span class="relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full bg-slate-300"></span>
            <div class="text-start flex-grow">
                <h2 class="font-bold">You</h2>
                <div class="text-sm">0 Wins</div>
            </div>
        </div>

        {#if opponentName}
            <div class="flex items-center justify-between gap-2">
                <span class="relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full bg-slate-300"></span>
                <div class="text-start flex-grow">
                    <h2 class="font-bold">Opponent</h2>
                    <div class="text-sm">0 Wins</div>
                </div>
            </div>
        {:else}
            <div class="flex items-center justify-between gap-2">
                <span class="relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full bg-slate-300"></span>
                <div class="text-start flex-grow">
                    <h2 class="font-bold">Opponent</h2>
                    <div class="text-sm">Disconnected</div>
                </div>
            </div>
        {/if}

    </div>


    <div class="flex gap-2 sm:w-full justify-center">
        <button id="newgame-button" on:click={resetGame} class="btn btn-outline btn-sm w-32  sm:w-12 sm:hidden">New game</button>
        <button id="leaveroom-button" on:click={leaveRoom} class="btn btn-outline w-32 whitespace-nowrap btn-sm">Leave Room</button>
    </div>
</div>
