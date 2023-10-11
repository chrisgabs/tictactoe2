<script>
    let board = [
        [null, null, null],
        [null, null, null],
        [null, null, null],
    ];

    let draggedElement = null;
    let numPieces = 8;
    let pieces = [];
    let maxSize = 50;
    let minSize = 20;
    let dragging = false;

    let sizeIncrements = (maxSize - minSize) / numPieces;
    for (let i = 1; i != numPieces + 1; i++) {
        pieces.push({
            size: parseInt(i * sizeIncrements) + minSize,
        });
    }

    function drag(event) {
        console.log(event.target);
        event.dataTransfer.setData("piece", event.target.id);
        draggedElement = event.target;
        dragging = true;
        console.log("draggin");
    }

    function allowDrop(event) {
        event.preventDefault();
        if (dragging && draggedElement) {
            dragging = false;
            let target = event.target;
            // check if target is a cell
            if (target.getAttribute("class").includes("draggable")) {
                let parent = event.target.parentElement;
                if (isValidMove(draggedElement, parent.firstElementChild)) {
                    target.parentElement.setAttribute("style", "background-color: rgb(232, 255, 223)");
                    console.log("valid1");
                    return;
                }
                target.parentElement.setAttribute("style", "background-color: rgb(255, 223, 223)");
                console.log("invalid1");
                return;
            }

            // check if cell is not empty
            if (target.firstElementChild != null) {
                if (isValidMove(draggedElement, event.target.firstElementChild)) {
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

        if (!draggedElement) {
            return;
        }

        // check if target is a cell
        if (event.target.getAttribute("class").includes("draggable")) {
            let parent = event.target.parentElement;
            parent.setAttribute("style", "");
            if (!isValidMove(piece, parent.firstElementChild)) {
                return;
            }
            parent.removeChild(parent.firstElementChild);
            parent.appendChild(piece);
            piece.setAttribute("draggable", null);
            return;
        }

        event.target.setAttribute("style", "");
        // check if cell is empty
        if (event.target.firstElementChild != null) {
            if (!isValidMove(piece, event.target.firstElementChild)) {
                return;
            }
            event.target.removeChild(event.target.firstElementChild);
        }
        event.target.appendChild(piece);
        event.target.setAttribute("style", "background-color: ");
        piece.setAttribute("draggable", false);
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
</script>

<div id="container" class="w-screen bg-slate-200 gap-4 h-screen flex flex-col items-center justify-center">
    <div class="pieces-container">
        {#each pieces as piece (piece.size)}
            <div
                class="draggable"
                style="width: {piece.size}px; height: {piece.size}px; background-color: #ff9d87"
                size={piece.size}
                draggable="true"
                id={"X" + piece.size}
                on:dragstart={drag}
                on:dragend={dragEnd}
                on:mousedown={mouseDown}
                role="gridcell"
                tabindex="0"
            >
                <!-- {piece.size} -->
            </div>
        {/each}
    </div>

    <div class="grid grid-cols-3 gap-1 bg-white p-1 rounded-lg shadow-lg">
        <!-- Create the Tic-Tac-Toe grid with 3x3 cells -->
        <div
            class="cell z-10"
            cell="0"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
        <div
            class="cell z-10"
            cell="1"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
        <div
            class="cell z-10"
            cell="2"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
        <div
            class="cell z-10"
            cell="3"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
        <div
            class="cell z-10"
            cell="4"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
        <div
            class="cell z-10"
            cell="5"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
        <div
            class="cell z-10"
            cell="6"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
        <div
            class="cell z-10"
            cell="7"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
        <div
            class="cell z-10"
            cell="8"
            on:dragleave={dragLeave}
            on:drop={drop}
            on:dragover={allowDrop}
            on:drag={(e) => console.log("asd")}
            role="gridcell"
            draggable="false"
            tabindex="0"
        />
    </div>

    <!-- Create a draggable element -->
    <div class="pieces-container">
        {#each pieces as piece (piece.size)}
            <div
                class="draggable"
                style="width: {piece.size}px; height: {piece.size}px; background-color: #8793ff"
                size={piece.size}
                draggable="true"
                id={"O" + piece.size}
                on:dragstart={drag}
                on:dragend={dragEnd}
                on:mousedown={mouseDown}
                role="gridcell"
                tabindex="0"
            >
                <!-- {piece.size} -->
            </div>
        {/each}
    </div>
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
