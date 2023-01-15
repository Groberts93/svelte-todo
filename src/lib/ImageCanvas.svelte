<script lang="ts">
    import { onMount } from "svelte";
    export let src: string | null;

    let image: HTMLImageElement;
    let canvas: HTMLCanvasElement;
    
    let canvas_width = 0;
    let canvas_height = 0;

    const drawImage = () => {

        const {width, height} = image;
        let downsizeRatio = 3;


        canvas_width = width / downsizeRatio;
        canvas_height = height / downsizeRatio;


        const ctx = canvas.getContext("2d");
        ctx.drawImage(image, 0, 0, 300, 200);
    }

</script>

<img on:load={drawImage} bind:this={image} {src} alt="nasdfasf" />
<canvas id="image_resizer" bind:this={canvas}  width={300} height={200} />
