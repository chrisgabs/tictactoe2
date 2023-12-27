<script>
    export let pieces;
</script>

<div class="pieces-container flex items-center justify-center gap-1 p-1">
    {#each [...pieces].reverse() as piece (piece.size)}
        <div
            class="draggable transition-transform duration-300 z-40 indicator"
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

<style>
    .draggable {
        /* color: #fff; */
        /* text-align: center; */
        /* line-height: 50px; */
        cursor: move;
        margin: auto;
        /* border-radius: 6%; */
    }
</style>
