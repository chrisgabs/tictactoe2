<script>
    import { onMount } from "svelte";
    import Board2 from "$lib/components/Board2.svelte";
    import PiecesContainer from "$lib/components/PiecesContainer.svelte";
    import PiecesContainerOpponent from "$lib/components/PiecesContainerOpponent.svelte";
    import DummyBoard from "$lib/components/DummyBoard.svelte";
    import Console from "$lib/components/Console.svelte";
    import { API_ENDPOINT } from '$lib/config/env.js';

    let board = [
        [null, null, null],
        [null, null, null],
        [null, null, null],
    ];

    const SERVER_ADDRESS = import.meta.env.VITE_API_URL;
    console.log(process.env.NODE_ENV);
    console.log("API_ENDPOINT " + API_ENDPOINT)
    console.log("SERVER_ADDRESS " + SERVER_ADDRESS)

    let gameResets = 0;
    let socket = null;
    let playerNumber = null;
    let roomId = null;
    let opponentName = null;
    let playerWithTurn = 1;
    let displayName = null;
    let showWinScreen = false;
    let winMessage = null;

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
        // let elements = document.getElementById("board");
        // console.log(elements);
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
        if (data.winner === opponentName) {
            winMessage = "Opponent has won, play again?";
        }else {
            winMessage = "You won, play again?";
        }

        // prompt = data.winner + " won, would you like to start a new game?";
        // winnerName
        // set z value of pieeces to 0
        console.log(data)
        showWinScreen = true;
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
        const toRemove = document.getElementById("opponent-indicator");
        piece.removeChild(toRemove);
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
            const newElement = document.createElement("span");
            newElement.className = "indicator-item badge opacity-70 indicator-bottom indicator-center";
            newElement.textContent = "Opponent";
            newElement.id = "opponent-indicator";
            opponentPiece.appendChild(newElement);
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

    function playAgain() {
        socket.send(
            JSON.stringify({
                EventType: "ready",
                PlayerId: playerNumber,
                data: null,
            })
        );
        showWinScreen = false;
    }

    function closeWinScreen() {
        showWinScreen = false;
    }
</script>

<div class="absolute w-2 h-2 bg-slate-800 transition-transform duration-300 hidden" id="cursor" />
<!-- <div class="absolute left-5 top-5">Player: {playerNumber}</div>
<div class="absolute left-5 top-10">Opponent: {opponentName}</div>
<div class="absolute left-5 top-20">Room: {roomId}</div> -->

<div id="container" class="w-screen h-screen flex flex-col items-center justify-center">
    {#if playerNumber != null}
        <div class="flex flex-col gap-2 justify-center align-middle text-center">
            <p>Your Room Number: {roomId}</p>
            <div class="flex space-x-2">
                <input id="room-input" type="text" placeholder="Enter Room Number" class="input input-bordered w-full max-w-xs" />
                <button class="p-2 btn btn-outline" on:click={joinRoom}>Join Room</button>
            </div>
        </div>
    {/if}

    <div class="flex flex-col sm:flex-row gap-4">
        {#key gameResets}
            <!-- Create draggable elements -->
            {#if playerNumber != null}
                <div class="text-center">
                    <div class="relative">
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

                        {#if showWinScreen}
                            <div
                                class="absolute inset-0 bg-white bg-opacity-5 backdrop-blur-sm flex flex-col items-center justify-center gap-2 z-50"
                            >
                                <!-- Your loading indicator or content goes here -->
                                <span class="text-xl">{winMessage}</span>
                                <div class="flex gap-2">
                                    <button class="btn btn-circle btn-outline" on:click={playAgain}>
                                        <svg
                                            fill="#000000"
                                            version="1.1"
                                            id="Capa_1"
                                            xmlns="http://www.w3.org/2000/svg"
                                            xmlns:xlink="http://www.w3.org/1999/xlink"
                                            viewBox="-39.18 -39.18 156.73 156.73"
                                            xml:space="preserve"
                                            ><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g
                                                id="SVGRepo_tracerCarrier"
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                            ></g><g id="SVGRepo_iconCarrier">
                                                <g>
                                                    <path
                                                        d="M78.049,19.015L29.458,67.606c-0.428,0.428-1.121,0.428-1.548,0L0.32,40.015c-0.427-0.426-0.427-1.119,0-1.547l6.704-6.704 c0.428-0.427,1.121-0.427,1.548,0l20.113,20.112l41.113-41.113c0.429-0.427,1.12-0.427,1.548,0l6.703,6.704 C78.477,17.894,78.477,18.586,78.049,19.015z"
                                                    ></path>
                                                </g>
                                            </g></svg
                                        >
                                    </button>
                                    <button class="btn btn-circle btn-outline" on:click={closeWinScreen}>
                                        <svg viewBox="-5.28 -5.28 34.56 34.56" fill="none" xmlns="http://www.w3.org/2000/svg"
                                            ><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g
                                                id="SVGRepo_tracerCarrier"
                                                stroke-linecap="round"
                                                stroke-linejoin="round"
                                            ></g><g id="SVGRepo_iconCarrier">
                                                <g id="Menu / Close_MD">
                                                    <path
                                                        id="Vector"
                                                        d="M18 18L12 12M12 12L6 6M12 12L18 6M12 12L6 18"
                                                        stroke="#000000"
                                                        stroke-width="2"
                                                        stroke-linecap="round"
                                                        stroke-linejoin="round"
                                                    ></path>
                                                </g>
                                            </g></svg
                                        >
                                    </button>
                                </div>
                            </div>
                        {/if}
                    </div>
                    <!-- your turn prompt -->
                    {#if playerWithTurn == playerNumber}
                        <p>Your turn to move</p>
                    {:else}
                        <p>Opponent's turn to move</p>
                    {/if}
                </div>

                <Console {resetGame} {leaveRoom} {displayName} {opponentName} />
            {:else}
                <!-- <p>Connecting...</p> -->
                <DummyBoard {maxSize} {minSize} {numPieces} />
            {/if}
        {/key}
    </div>
</div>
