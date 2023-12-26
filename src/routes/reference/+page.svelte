<!--
// v0 by Vercel.
// https://v0.dev/t/HS7jV1uxqN6
-->
<div class="flex flex-col items-center justify-center min-h-screen bg-gray-50 ">
    <h1 class="text-3xl font-bold mb-4">Tic Tac Toe</h1>
    <div class="flex flex-col lg:flex-row gap-4">
        <div class="p-4 border shadow-lg rounded-lg bg-white ">
            <div class="grid grid-cols-3 gap-4">
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold">X</span>
                </div>
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold">O</span>
                </div>
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold"></span>
                </div>
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold">O</span>
                </div>
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold">X</span>
                </div>
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold"></span>
                </div>
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold">X</span>
                </div>
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold"></span>
                </div>
                <div
                    class="rounded-lg border bg-card text-card-foreground shadow-sm h-24 w-24 flex items-center justify-center"
                    data-v0-t="card"
                >
                    <span class="text-2xl font-bold">O</span>
                </div>
            </div>
        </div>
        <div class="flex flex-col gap-4">
            <button
                class="inline-flex items-center justify-center rounded-md text-sm font-medium ring-offset-background transition-colors focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50 border border-input bg-background hover:bg-accent hover:text-accent-foreground h-10 px-4 py-2 w-full"
                >New game</button
            >
            <div class="flex items-center justify-between">
                <span class="relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full"></span>
                <div class="text-center">
                    <h2 class="font-bold">Player 1</h2>
                    <div
                        class="inline-flex items-center rounded-full border px-2.5 py-0.5 w-fit text-xs font-semibold transition-colors focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 border-transparent bg-primary text-primary-foreground hover:bg-primary/80"
                    >
                        2 Wins
                    </div>
                </div>
            </div>
            <div class="flex items-center justify-between">
                <span class="relative flex h-10 w-10 shrink-0 overflow-hidden rounded-full"></span>
                <div class="text-center">
                    <h2 class="font-bold">Player 2</h2>
                    <div
                        class="inline-flex items-center rounded-full border px-2.5 py-0.5 w-fit text-xs font-semibold transition-colors focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2 border-transparent bg-primary text-primary-foreground hover:bg-primary/80"
                    >
                        1 Win
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
