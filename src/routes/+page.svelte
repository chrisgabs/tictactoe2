<script>
    import { onMount } from "svelte";

    let board = [
        [null, null, null],
        [null, null, null],
        [null, null, null],
    ];

    let socket = null;
    let intervalId = null;
    let playerNumber = null;

    let draggedElement = null;
    let numPieces = 8;
    let pieces = [];
    let tileIds = [];
    let maxSize = 50;
    let minSize = 20;
    let dragging = false;

    let ogOpponentPieceCoords = null;
    let opponentDragging = false;

    let mouseX = 0;
    let mouseY = 0;

    onMount(() => {
        // document.addEventListener("mousemove", handleMouseMove)
        socket = new WebSocket("ws://127.0.0.1:8080/ws");
        console.log("Attempting Connection...");

        socket.onopen = () => {
            console.log("Successfully Connected");
            socket.send(
                JSON.stringify({
                    EventType: "connect",
                    data: null,
                })
            );
        };

        socket.onclose = (event) => {
            console.log("Socket Closed Connection: ", event);
            socket.send("Client Closed!");
        };

        socket.onerror = (error) => {
            console.log("Socket Error: ", error);
        };

        socket.onmessage = (message) => {
            const received = JSON.parse(message.data);
            // console.log(received);
            const eventType = received.EventType;
            const data = received.Data;
            console.log(eventType);
            switch (eventType) {
                case "connect":
                    playerNumber = data.Id;
                    setUpBoardData(data.Id);
                    break;
                case "move":
                    // console.log(data);
                    handleOpponentMouseMove(data);
                    break;
                case "dragend":
                    handleOpponentDragEnd(data);
                    break;
                case "drop":
                    console.log(received)
                    handleOpponentDrop(data);
                    break;
            }
        };

        setInterval(() => {
            if (draggedElement != null) {
                socket.send(
                    JSON.stringify({
                        EventType: "move",
                        data: {
                            ClientX: mouseX,
                            ClientY: mouseY,
                            PlayerNumber: playerNumber,
                            PieceID: draggedElement.getAttribute("id"),
                        },
                    })
                );
            }
        }, 100);

    });

    function handleOpponentDrop(data) {
        console.log("opponent dropped");
        console.log(data)
    }
    
    function handleOpponentDragEnd(data) {
        opponentDragging = false;
    }

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
            console.log("reversed");
            tileIds.reverse();
        }
        console.log(tileIds)
    }

    function handleOpponentMouseMove(data) {
        const cursorElement = document.getElementById("cursor");
        const opponentPiece = document.getElementById("opponent-" + data.PieceID);
        if (!opponentDragging) {
            ogOpponentPieceCoords = opponentPiece.getBoundingClientRect();
            opponentDragging = true;
        }
        // Could be optimized. Maybe only update gameBounds if screen is resized or magnified.
        const gameBounds = document.getElementById("game-bounds").getBoundingClientRect();
        cursorElement.style.transform = `translate(${data.ClientX + gameBounds.x}px, ${data.ClientY + gameBounds.y}px)`;
        // place relative to game area - offset original position of piece - estimate opponent's pointer coordinates in middle
        // TODO: accurately calculate pointer coordinates of opponent
        const pieceX = (data.ClientX + gameBounds.x) - ogOpponentPieceCoords.x - (ogOpponentPieceCoords.width/2);
        const pieceY = (data.ClientY + gameBounds.y) - ogOpponentPieceCoords.y - (ogOpponentPieceCoords.height/2);
        opponentPiece.style.transform = `translate(${pieceX}px, ${pieceY}px)`;
    }

    function updateMouseCoordinates(event) {
        const bounds = event.currentTarget.getBoundingClientRect();
        // coords should be relative to play area and mirrored
        mouseX = bounds.width - (event.clientX - bounds.x);
        mouseY = bounds.height - (event.clientY - bounds.y);
    };

    function dragStart(event) {
        event.dataTransfer.setData("piece", event.target.id);
        draggedElement = event.target;
        dragging = true;
    }

    function drag(event) {
        return;
        // mouseX = event.clientX;
        // mouseY = event.clientY;
    }

    function allowDrop(event) {
        event.preventDefault();
        if (dragging && draggedElement) {
            dragging = false;
            let target = event.currentTarget;

            // check if cell is not empty
            if (target.firstElementChild != null) {
                if (isValidMove(draggedElement, target.firstElementChild)) {
                    target.setAttribute("style", "background-color: rgb(232, 255, 223)");
                    console.log("valid2");
                    return;
                }
                target.setAttribute("style", "background-color: rgb(255, 223, 223)");
                console.log("invalid2");
            } else {
                target.setAttribute("style", "background-color: rgb(232, 255, 223)");
                console.log("valid3");
            }
        }
    }

    function drop(event) {
        event.preventDefault();
        const data = event.dataTransfer.getData("piece");
        let piece = document.getElementById(data);
        let target = event.currentTarget;

        if (!draggedElement) {
            return;
        }

        target.setAttribute("style", "");
        // check if cell is empty
        if (target.firstElementChild != null) {
            if (!isValidMove(piece, target.firstElementChild)) {
                return;
            }
            target.removeChild(target.firstElementChild);
        }
        target.appendChild(piece);
        target.setAttribute("style", "background-color: ");
        piece.setAttribute("draggable", false);
        updateBoard(piece.getAttribute("id"), target.getAttribute("id"));

        socket.send(
            JSON.stringify({
                EventType: "drop",
                data: target.getAttribute("id")
            })
        );

        checkForWin();
    }

    function isValidMove(piece, target) {
        let x = parseInt(piece.getAttribute("size"));
        let y = parseInt(target.getAttribute("size"));
        console.log(x, y);
        return x > y;
    }

    function dragEnd() {
        dragging = false;
        draggedElement = null;
        socket.send(
            JSON.stringify({
                EventType: "dragend",
                data: null,
            })
        );
        clearInterval(intervalId);
    }

    function dragLeave(event) {
        if (event.target.getAttribute("class").includes("draggable")) {
            return;
        }
        event.target.setAttribute("style", "background-color: ");
        dragging = true;
    }

    function mouseDown(event) {
        if (event.target.getAttribute("draggable") === "false") {
            event.preventDefault();
        }
    }

    function updateBoard(piece, cell) {
        const pieceData = Number(piece.split(",")[0]);
        const cellNumber = Number(cell);

        const x = Math.trunc(cellNumber / 3);
        const y = cellNumber % 3;
        board[x][y] = pieceData;
        console.log(board);
    }

    // trash algo
    function checkForWin() {
        // Check rows
        board.forEach((row) => {
            const sum = row.reduce((acc, current) => acc + current, 0);
            if (checkSum(sum)) return;
        });

        // Check columns
        for (let i = 0; i < 3; i++) {
            const sum = board[0][i] + board[1][i] + board[2][i];
            if (checkSum(sum)) return;
        }

        // Check diagonals
        let sum = board[0][0] + board[1][1] + board[2][2];
        if (checkSum(sum)) return;
        sum = board[0][2] + board[1][1] + board[2][0];
        if (checkSum(sum)) return;
    }

    function checkSum(sum) {
        if (sum == 3) {
            alert("player 1 win");
            return true;
        } else if (sum == -3) {
            alert("player -1 win");
            return true;
        }
        return false;
    }
</script>

<div class="absolute w-2 h-2 bg-slate-800 transition-transform duration-300" id="cursor" />

<!-- Create draggable elements -->
<div id="container" class="w-screen h-screen bg-slate-200 flex items-center justify-center">
    {#if playerNumber != null}
        <div id="game-bounds" class="bg-black p-6 flex flex-col gap-2" on:dragover={updateMouseCoordinates} role="table">
            <div class="pieces-container">
                {#each [...pieces].reverse() as piece (piece.size)}
                    <div
                        class="draggable transition-transform duration-300"
                        style="width: {piece.size}px; height: {piece.size}px; background-color: #ff9d87"
                        size={piece.size}
                        draggable="true"
                        id={"opponent-" + piece.size}
                        role="gridcell"
                        tabindex="0"
                    >
                        <!-- {piece.size} -->
                    </div>
                {/each}
            </div>

            <!-- Create the Tic-Tac-Toe grid with 3x3 cells -->
            <div class="grid grid-cols-3 gap-1 bg-white p-1 rounded-lg shadow-lg">
                {#each tileIds as id, index (index)}
                    <div
                        class="cell z-10"
                        id={id}
                        on:dragleave={dragLeave}
                        on:drop={drop}
                        on:dragover={allowDrop}
                        role="gridcell"
                        draggable="false"
                        tabindex="0"
                    >
                        {id}
                    </div>
                {/each}
            </div>

            <!-- Create draggable elements -->
            <div class="pieces-container">
                {#each pieces as piece (piece.size)}
                    <div
                        class="draggable"
                        style="width: {piece.size}px; height: {piece.size}px; background-color: #8793ff"
                        size={piece.size}
                        draggable="true"
                        id={"" + piece.size}
                        on:drag={drag}
                        on:dragstart={dragStart}
                        on:dragend={dragEnd}
                        on:mousedown={mouseDown}
                        role="gridcell"
                        tabindex="0"
                    >
                    </div>
                {/each}
            </div>
        </div>
    {:else}
        <p>Connecting...</p>
    {/if}
</div>

<style>
    .cell {
        display: flex;
        border: 1px solid rgb(121, 121, 121);
        min-height: 100px;
        min-width: 100px;
        align-items: center;
        /* justify-items: center; */
        text-align: center;
    }

    .draggable {
        /* width: 50px;
        height: 50px; */
        /* background-color: #ff9d87; */
        /* background-color: #8793ff; */
        color: #fff;
        text-align: center;
        line-height: 50px;
        cursor: move;
        margin: auto;
        border-radius: 6%;
    }

    .pieces-container {
        display: flex;
        gap: 6px;
        /* border: 1px solid rgb(167, 167, 167); */
        border-radius: 3px;
        padding: 3px;
    }

    .valid-cell {
        background-color: rgb(232, 255, 223);
    }

    .invalid-cell {
        background-color: rgb(255, 223, 223);
    }
</style>
