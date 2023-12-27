<script>
    export let pieces;
    export let playerWithTurn;
    export let playerNumber;
    export let drag;
    export let dragStart;
    export let dragEnd;
    export let mouseDown;

    // onMount(() => {
    //     setUpBoardDataAfterJoining(playerNumber, boardData);
    // });

    // function setUpBoardDataAfterJoining(playerNumber, boardData) {
    //     console.log("board data:");
    //     // update board cell ids
    //     let elements = document.getElementById("board").children;
    //     for (let i = 0; i < elements.length; i++) {
    //         // console.log(elements[i])
    //         if (playerNumber == 1) {
    //             elements[i].id = i;
    //         } else {
    //             elements[i].id = elements.length - 1 - i;
    //         }
    //     }

    //     // place pieces in board
    //     for (let i in boardData) {
    //         const piece = boardData[i].split(",");
    //         let pieceId = "";
    //         if (piece[0] === "0") {
    //             continue;
    //         } else if (piece[0] == playerNumber) {
    //             pieceId = piece[1];
    //         } else {
    //             pieceId = "opponent-" + piece[1];
    //         }
    //         const pieceElement = document.getElementById(pieceId);
    //         const cellElement = document.getElementById(i);
    //         placePieceInCell(pieceElement, cellElement);
    //     }
    // }
</script>

<div class="pieces-container flex items-center justify-center gap-1 p-1">
    {#each pieces as piece (piece.size)}
        <div class="draggable cursor-move mx-auto z-40" style="width: {piece.size}px; height: {piece.size}px;"
            size={piece.size}
            draggable={playerWithTurn == playerNumber}
            id={"" + piece.size}
            on:drag={drag}
            on:dragstart={dragStart}
            on:dragend={dragEnd}
            on:mousedown={mouseDown}
            role="gridcell"
            tabindex="0"
        >
            <svg class="w-full h-full">
                <circle cx="50%" cy="50%" r="40%" stroke="black" stroke-width="3.5" fill="none" />
            </svg>
        </div>
    {/each}
</div>
