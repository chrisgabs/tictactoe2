<script>
    export let maxSize;
    export let minSize;
    export let numPieces;

    let pieces = [];
    let tileIds = [];
    let playerNumber = 1;

    setUpBoardData(playerNumber);

    function setUpBoardData(playerNumber) {
        // compute piece sizes
        let sizeIncrements = (maxSize - minSize) / numPieces;
        for (let i = 1; i != numPieces + 1; i++) {
            pieces.push({
                size: parseInt(i * sizeIncrements) + minSize,
            });
        }

        // tile ids
        for (let i = 0; i < 9; i++) {
            tileIds.push(i);
        }

        // reverse tile ids if player 2
        if (playerNumber % 2 == 0) {
            tileIds.reverse();
        }
    }
</script>

<div class="relative">
    <div id="game-bounds" class=" pt-6 pb-1 flex flex-col gap-2" role="table">
        <div class="pieces-container flex items-center justify-center gap-1 p-1">
            {#each [...pieces].reverse() as piece (piece.size)}
                <div
                    class="draggable transition-transform duration-300"
                    style="width: {piece.size}px; height: {piece.size}px;"
                    size={piece.size}
                    draggable="true"
                    id={"opponent-" + piece.size}
                    role="gridcell"
                    tabindex="0"
                >
                    <svg
                        xmlns="http://www.w3.org/2000/svg"
                        viewBox="0 0 24 24"
                        fill="none"
                        stroke="black"
                        stroke-width="3"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                    >
                        <line x1="18" y1="6" x2="6" y2="18" />
                        <line x1="6" y1="6" x2="18" y2="18" />
                    </svg>
                </div>
            {/each}
        </div>

        <!-- Create the Tic-Tac-Toe grid with 3x3 cells -->
        <!-- Your board content -->
        <div class="p-4 border shadow-lg rounded-lg bg-white">
            <div class="grid grid-cols-3 gap-4" id="board">
                {#each tileIds as id, index (index)}
                    <div
                        class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                        {id}
                        role="gridcell"
                        draggable="false"
                        tabindex="0"
                    >
                        <!-- <span class="text-2xl font-bold">X</span> -->
                    </div>
                {/each}
            </div>
        </div>

        <!-- Create draggable elements -->
        <div class="pieces-container flex items-center justify-center gap-1 p-1">
            {#each pieces as piece (piece.size)}
                <div class="mx-auto" style="width: {piece.size}px; height: {piece.size}px;" role="gridcell" tabindex="0">
                    <svg class="w-full h-full">
                        <circle cx="50%" cy="50%" r="40%" stroke="black" stroke-width="3.5" fill="none" />
                    </svg>
                </div>
            {/each}
        </div>
    </div>
    <!-- Loading overlay -->
    <div class="absolute inset-0 bg-white bg-opacity-5 backdrop-blur-sm flex items-center justify-center gap-2">
        <!-- Your loading indicator or content goes here -->
        <span class="loading loading-spinner loading-md"></span>
        <span class="text-l">Connecting...</span>
    </div>
</div>
