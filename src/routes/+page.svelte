<script>
    import { onMount } from "svelte";
    import Board from "$lib/components/Board.svelte";
    import Board2 from "$lib/components/Board2.svelte";
    import PiecesContainer from "$lib/components/PiecesContainer.svelte";
    import PiecesContainerOpponent from "$lib/components/PiecesContainerOpponent.svelte";
    import Console from "$lib/components/Console.svelte";
    import { API_BASE_URL } from "$lib/config/constants";

    let board = [
        [null, null, null],
        [null, null, null],
        [null, null, null],
    ];

    const SERVER_ADDRESS = API_BASE_URL;

    let gameResets = 0;
    let socket = null;
    let playerNumber = null;
    let roomId = null;
    let opponentName = null;
    let playerWithTurn = 1;
    let displayName = null;

    let draggedElement = null;
    let numPieces = 8;
    let pieces = [];
    let tileIds = [];
    let maxSize = 50;
    let minSize = 20;
    let dragging = false;
    let pieceDropped = true;

    let ogOpponentPieceCoords = null;
    let opponentDragging = false;

    let mouseX = 0;
    let mouseY = 0;

    let boardData;

    onMount(() => {
        // Retrieve session informtion from backend
        setUpBoardData(1);
        (async () => {
            console.log("------- connect -------");
            const response = await fetch("https://" + SERVER_ADDRESS + "/connect", {
                // const response = await fetch("http://" + "192.168.19.7:8080" + "/connect", {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                },
                credentials: "include",
            });

            if (response.ok) {
                const data = await response.json();
                console.log(data);
                displayName = data.displayName;
                if (data.newPlayer) {
                    socket = new WebSocket("wss://" + SERVER_ADDRESS + "/ws/newPlayer");
                    console.log("Attempting Connection... new player");
                } else {
                    socket = new WebSocket("wss://" + SERVER_ADDRESS + "/ws/existingPlayer");
                    console.log("Attempting Connection... existing player");
                    if (data.gameOngoing) {
                        // load ongoing game
                    } else {
                        // show new player view
                    }
                }
                setupWebsocketConnection();
            } else {
                console.log("Error in receiving session information");
            }
        })();
        let elements = document.getElementById("board");
        console.log(elements);
        // document.addEventListener("mousemove", handleMouseMove)
    });

    // function initializeWebSocketConnection() {}

    function setupWebsocketConnection() {
        socket.onopen = () => {
            console.log("Successfully Connected");
            socket.send(
                JSON.stringify({
                    EventType: "connect",
                    PlayerId: playerNumber,
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
            console.log("WS received: --");
            console.log(received);
            const eventType = received.EventType;
            const data = received.Data;
            // console.log(eventType);
            switch (eventType) {
                case "connect":
                    playerNumber = data.playerNumber;
                    roomId = data.roomId;
                    boardData = data.boardData;
                    playerWithTurn = data.playerWithTurn;
                    opponentName = data.opponentDisplayName;
                    console.log(data);
                    // setUpBoardDataAfterJoining(data.playerNumber, data.boardData);
                    break;
                case "move":
                    // console.log(data);
                    handleOpponentMouseMove(data);
                    break;
                case "dragend":
                    handleOpponentDragEnd(data);
                    break;
                case "drop":
                    // console.log(received);
                    handleOpponentDrop(data);
                    break;
                case "join":
                    handleOpponentJoin(data);
                    break;
                case "reset":
                    handleGameReset(data);
                    break;
                case "leave":
                    opponentName = null;
                    playerWithTurn = playerNumber;
                    gameResets += 1;
                    break;
                case "win":
                    handleWin(data);
                    break;
                case "game_start":
                    handleGameStart(data);
                    break;
            }
        };

        setInterval(() => {
            if (!pieceDropped && draggedElement != null) {
                socket.send(
                    JSON.stringify({
                        EventType: "move",
                        PlayerId: playerNumber,
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

        // to prevent cloudflare from disconnecting ws
        // send every 95 seconds
        setInterval(() => {
            if (socket && socket.readyState === WebSocket.OPEN) {
                socket.send(
                    JSON.stringify({
                        EventType: "keepAlive",
                        PlayerId: playerNumber,
                        data: null,
                    })
                );
            }
        }, 95000);
    }

    function handleGameStart(data) {
        console.log("Game has started player with turn: " + data.playerWithTurn);
        playerWithTurn = data.playerWithTurn;
        gameResets += 1;
    }

    function handleWin(data) {
        prompt = data.winner + " won, would you like to start a new game?";
        if (confirm(prompt) == true) {
            socket.send(
                JSON.stringify({
                    EventType: "ready",
                    PlayerId: playerNumber,
                    data: null,
                })
            );
        }
    }

    function handleGameReset(data) {
        gameResets += 1;
        console.log("Game is reset by " + data.displayName);
        console.log(playerWithTurn);
        playerWithTurn = data.playerWithTurn;
    }

    function handleOpponentJoin(data) {
        opponentName = data.opponentDisplayName;
        return;
    }

    function handleOpponentDrop(data) {
        // console.log("opponent dropped");
        if (data.isValidMove) {
            playerWithTurn = playerNumber == 1 ? 1 : 2;
            const piece = document.getElementById("opponent-" + data.piece);
            const cell = document.getElementById(data.cell);
            placePieceInCell(piece, cell);
            // console.log("placed");
        }
    }

    function handleOpponentDragEnd(data) {
        const piece = document.getElementById("opponent-" + data.piece);
        piece.style.transform = `translate(0px, 0px)`;
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
            tileIds.reverse();
        }
    }

    function setUpBoardDataAfterJoining(playerNumber, boardData) {
        console.log("board data:");
        // update board cell ids
        let elements = document.getElementById("board").children;
        console.log(elements);
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
        const pieceX = data.ClientX + gameBounds.x - ogOpponentPieceCoords.x - ogOpponentPieceCoords.width / 2;
        const pieceY = data.ClientY + gameBounds.y - ogOpponentPieceCoords.y - ogOpponentPieceCoords.height / 2;
        opponentPiece.style.transform = `translate(${pieceX}px, ${pieceY}px)`;
        // console.log("moved")
    }

    function updateMouseCoordinates(event) {
        const bounds = event.currentTarget.getBoundingClientRect();
        // coords should be relative to play area and mirrored
        mouseX = bounds.width - (event.clientX - bounds.x);
        mouseY = bounds.height - (event.clientY - bounds.y);
    }

    function dragStart(event) {
        // To remove drag ghost image
        // let ghostImageDummy = new Image();
        // ghostImageDummy.src = 'data:image/gif;base64,R0lGODlhAQABAIAAAAUEBAAAACwAAAAAAQABAAACAkQBADs=';
        // event.dataTransfer.setDragImage(ghostImageDummy, 0, 0);
        event.dataTransfer.setData("piece", event.target.id);
        pieceDropped = false;
        draggedElement = event.currentTarget;
        draggedElement.classList.add("opacity-70");
        dragging = true;
    }

    function drag(event) {
        event.preventDefault();
    }

    function dragOverHandler(event) {
        event.preventDefault();
        if (dragging && draggedElement) {
            console.log("dragger");
            dragging = false;
            let target = event.currentTarget;

            // target.classList.add("bg-amber-500")
            // check if cell is not empty
            if (target.firstElementChild != null) {
                if (checkIfValidMove(draggedElement, target)) {
                    target.setAttribute("style", "background-color: rgb(232, 255, 223)");
                    console.log("valid2");
                    return;
                } else {
                    target.setAttribute("style", "background-color: rgb(255, 223, 223)");
                    console.log("invalid2");
                }
            } else {
                target.setAttribute("style", "background-color: rgb(232, 255, 223)");
                console.log("valid3");
            }
        }
    }

    function drop(event) {
        event.preventDefault();
        pieceDropped = true;
        const data = event.dataTransfer.getData("piece");
        let piece = document.getElementById(data);
        let cell = event.currentTarget;
        let isValidMove = checkIfValidMove(piece, cell);
        cell.setAttribute("style", "background-color: ");

        if (!draggedElement) {
            return;
        }

        if (isValidMove) {
            placePieceInCell(piece, cell);
            playerWithTurn = playerNumber == 1 ? 2 : 1;
        }

        socket.send(
            JSON.stringify({
                EventType: "drop",
                PlayerId: playerNumber,
                data: {
                    playerNumber: playerNumber,
                    cell: cell.getAttribute("id"),
                    piece: piece.getAttribute("id"),
                    isValidMove: isValidMove,
                },
            })
        );

        // checkForWin();
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

    function checkIfValidMove(piece, tile) {
        if (tile.firstElementChild == null) {
            return true;
        }
        let x = parseInt(piece.getAttribute("size"));
        let y = parseInt(tile.firstElementChild.getAttribute("size"));
        return x > y;
    }

    function dragEnd() {
        dragging = false;
        draggedElement.classList.remove("opacity-70");
        socket.send(
            JSON.stringify({
                EventType: "dragend",
                PlayerId: playerNumber,
                data: {
                    piece: draggedElement.getAttribute("id"),
                },
            })
        );
        draggedElement = null;
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

    async function joinRoom() {
        const roomInput = document.getElementById("room-input");
        console.log("-- Joining room: " + roomInput.value);
        const response = await fetch("https://" + SERVER_ADDRESS + "/join", {
            method: "PUT",
            headers: {
                "Content-Type": "application/json",
            },
            credentials: "include",
            body: JSON.stringify({
                RoomId: roomInput.value,
            }),
        });
        if (response.ok) {
            console.log("successfully joined room: " + roomInput.value);
            const data = await response.json();
            roomId = roomInput.value;
            opponentName = data.opponentDisplayName;
            playerNumber = data.playerNumber;
            setUpBoardDataAfterJoining(playerNumber, data.data);
            console.log(data);
        } else {
            console.log("ERROR could not join room: " + roomInput.value);
        }
    }

    function resetGame() {
        socket.send(
            JSON.stringify({
                EventType: "reset",
                PlayerId: playerNumber,
                data: null,
            })
        );
    }

    async function leaveRoom() {
        socket.send(
            JSON.stringify({
                EventType: "leave",
                PlayerId: playerNumber,
                data: null,
            })
        );
        const response = await fetch("https://" + SERVER_ADDRESS + "/leave", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            credentials: "include",
            body: null,
        });
        if (response.ok) {
            console.log("successfully left room: " + roomId);
            const data = await response.json();
            gameResets += 1;
            opponentName = null;
            playerNumber = data.playerNumber;
            roomId = data.roomId;
            playerWithTurn = playerNumber;
            console.log(data);
        } else {
            console.log("ERROR could not reassign new room");
        }
    }
</script>

<div class="absolute w-2 h-2 bg-slate-800 transition-transform duration-300" id="cursor" />
<div class="absolute left-5 top-5">Player: {playerNumber}</div>
<div class="absolute left-5 top-10">Opponent: {opponentName}</div>
<div class="absolute left-5 top-20">Room: {roomId}</div>

<div id="container" class="w-screen h-screen flex flex-col items-center justify-center">
    <div class="flex flex-col gap-2 justify-center align-middle text-center">
        <p>Your Room Number: {roomId}</p>
        <div class="flex space-x-2">
            <input id="room-input" type="text" placeholder="Enter Room Number" class="input input-bordered w-full max-w-xs" />
            <button class="p-2 btn btn-outline" on:click={joinRoom}>Join Room</button>
        </div>
    </div>

    <div class="flex flex-col sm:flex-row gap-4">
        {#key gameResets}
            <!-- Create draggable elements -->
            {#if playerNumber != null}
                <div class="text-center">
                    <div id="game-bounds" class=" pt-6 pb-1 flex flex-col gap-2" on:dragover={updateMouseCoordinates} role="table">
                        <PiecesContainerOpponent {pieces} />

                        <!-- Create the Tic-Tac-Toe grid with 3x3 cells -->
                        {#if boardData != null}
                            <Board2
                                {playerNumber}
                                {boardData}
                                {tileIds}
                                dragLeaveHandler={dragLeave}
                                dropHandler={drop}
                                dragoverHandler={dragOverHandler}
                            />
                        {/if}

                        <!-- Create draggable elements -->
                        <PiecesContainer {pieces} {playerWithTurn} {playerNumber} {drag} {dragStart} {dragEnd} {mouseDown} />
                    </div>
                    <!-- your turn prompt -->
                    {#if playerWithTurn == playerNumber}
                        <p>Your turn to move</p>
                    {:else}
                        <p>{opponentName}'s turn to move</p>
                    {/if}
                </div>
            {:else}
                <p>Connecting...</p>
            {/if}

            <Console
                {resetGame}
                {leaveRoom}
                {displayName}
                {opponentName}
            />
        {/key}
    </div>
</div>
