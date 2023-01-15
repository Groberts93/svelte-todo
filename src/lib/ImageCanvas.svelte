<script lang="ts">
    import { onMount } from "svelte";
    import { claim_text } from "svelte/internal";

    export let src: string | null;

    let canvas: HTMLCanvasElement;
    let ctx: CanvasRenderingContext2D;
    const image: HTMLImageElement = new Image();

    let canvas_width = 100;
    let canvas_height = 100;
    let scale = 0.5;

    let tx = 0;
    let ty = 0;
    /**
     * A component allowing the user to rescale an image, select an area to be masked for use as an avatar, and save the resulting avatar.
     */
    image.src = src;
    image.addEventListener("load", () => {
        ctx = canvas.getContext("2d");
        ctx.drawImage(image, 0, 0, scale * image.width, scale * image.height);
    });

    const drawImage = (event: WheelEvent | MouseEvent) => {
        if (ctx) {
            if (event instanceof WheelEvent) {
                scale = scale + event.deltaY / 1000;
            } else if (event instanceof MouseEvent) {
                tx = event.offsetX;
                ty = event.offsetY;
            }

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            ctx.drawImage(
                image,
                tx,
                ty,
                scale * image.width,
                scale * image.height
            );
        }
    };
</script>

<!-- <img on:load={drawImage} bind:this={image} {src} alt="nasdfasf" /> -->
<div id="container">
    <canvas
        id="image_resizer"
        on:pointermove={drawImage}
        on:wheel|preventDefault|nonpassive={drawImage}
        bind:this={canvas}
        width={250}
        height={250}
    />

    <!-- <label for="scale">Scale: {scale}</label> -->
    <!-- <input
        on:change={changeScale}
        bind:value={scale}
        id="scale"
        type="range"
        min="0.25"
        max="2.0"
        step="0.1"
    /> -->
</div>

<style>
    #container {
        display: grid;
    }
</style>
