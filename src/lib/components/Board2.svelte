<!-- MyComponent.svelte -->

<script>
    import { onMount } from "svelte";

    export let boardData; // Define a prop named "message"
    export let dragLeaveHandler;
    export let tileIds;
    export let dropHandler;
    export let dragoverHandler;
    export let playerNumber;

    onMount(() => {
        setUpBoardDataAfterJoining(playerNumber, boardData);
    });

    function setUpBoardDataAfterJoining(playerNumber, boardData) {
        // update board cell ids
        let elements = document.getElementById("board").children;
        for (let i = 0; i < elements.length; i++) {
            // console.log(elements[i])
            if (playerNumber == 1) {
                elements[i].id = i;
            } else {
                elements[i].id = elements.length - 1 - i;
            }
        }

        // place pieces in board
        for (let i in boardData) {
            const piece = boardData[i].split(",");
            let pieceId = "";
            if (piece[0] === "0") {
                continue;
            } else if (piece[0] == playerNumber) {
                pieceId = piece[1];
            } else {
                pieceId = "opponent-" + piece[1];
            }
            const pieceElement = document.getElementById(pieceId);
            const cellElement = document.getElementById(i);
            placePieceInCell(pieceElement, cellElement);
        }
    }

    function placePieceInCell(piece, cell) {
        // cell.setAttribute("style", "");
        // check if cell is empty
        if (cell.firstElementChild != null) {
            cell.removeChild(cell.firstElementChild);
        }
        piece.style.transform = `translate(0px, 0px)`;
        cell.appendChild(piece);
        // console.log("piece placed in cell, translte 0 0")
        cell.setAttribute("style", "background-color: ");
        piece.setAttribute("draggable", false);
    }
</script>

<div class="p-4 border shadow-lg rounded-lg bg-white">
    <div class="grid grid-cols-3 gap-4" id="board">
        {#each tileIds as id, index (index)}
            <div
                class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                {id}
                on:dragleave={dragLeaveHandler}
                on:drop={dropHandler}
                on:dragover={dragoverHandler}
                role="gridcell"
                draggable="false"
                tabindex="0"
            >
                <!-- <span class="text-2xl font-bold">X</span> -->
            </div>
        {/each}
    </div>
</div>
