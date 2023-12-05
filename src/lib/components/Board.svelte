<!-- MyComponent.svelte -->

<script>
    import { onMount } from "svelte";

    export let boardData; // Define a prop named "message"
    export let dragLeaveHandler;
    export let tileIds;
    export let dropHandler;
    export let dragoverHandler;
    export let board;
    export let playerNumber;

    onMount(() => {
        console.log("Logging from board.svelte");
        console.log(boardData);
        console.log(board);
        setUpBoardDataAfterJoining(playerNumber, boardData);
    })

    function setUpBoardDataAfterJoining(playerNumber, boardData) {
        console.log("board data:");
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
    updateBoard(piece.getAttribute("id"), cell.getAttribute("id"));
}

function updateBoard(piece, cell) {
        const pieceData = Number(piece.split(",")[0]);
        const cellNumber = Number(cell);

        const x = Math.trunc(cellNumber / 3);
        const y = cellNumber % 3;
        board[x][y] = pieceData;
}

</script>

<div id="board" class="grid grid-cols-3 gap-1 bg-white p-1 rounded-lg shadow-lg">
    {#each tileIds as id, index (index)}
        <div
            class="flex border border-gray-400 h-28 w-28 items-center text-center"
            {id}
            on:dragleave={dragLeaveHandler}
            on:drop={dropHandler}
            on:dragover={dragoverHandler}
            role="gridcell"
            draggable="false"
            tabindex="0"
        >
            <!-- {id} -->
        </div>
    {/each}
</div>