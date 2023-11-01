<script>
    import { onMount } from "svelte";

    onMount(() => {
        const draggable = document.getElementById("draggable");
        const target = document.getElementById("target");

        draggable.addEventListener("dragstart", (event) => {
            event.dataTransfer.setData("text/plain", "This is some data");
        });

        target.addEventListener("dragover", (event) => {
            event.preventDefault();
        });

        target.addEventListener("drop", (event) => {
            event.preventDefault();
            const data = event.dataTransfer.getData("text/plain");

            // Create a clone of the draggable element
            const clone = draggable.cloneNode(true);
            // clone.classList.add("text-lg"); // Add a CSS class for animation

            // Append the clone to the target div
            target.appendChild(clone);

            // Remove the original draggable element
            draggable.remove();

            console.log(clone)
            // Apply a transform to animate the clone
            clone.style.transform = `translate(${event.clientX}px, ${event.clientY}px)`;
            // clone.style.transform = "translate(0, 0)";

            setTimeout(() => {
                clone.style.transform = "translate(500, 500)";
                console.log("sdklfj")
            }, 0);
        });
    });
</script>

<div id="draggable" class="font-mono transition-transform duration-300" draggable="true">Drag Me</div>
<div id="target" class="bg-slate-50"/>

<style>
    #target {
        width: 200px;
        height: 200px;
        border: 2px dashed #333;
        position: relative;
    }

</style>
