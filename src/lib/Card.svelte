<script lang="ts">
    export let body = "card_body";
    export let username = "card_author";
    export let created_at;
    export let buttonCallback = () => console.log("ass");
    export let imageUrl: string | null = null;

    $: task_timestamp = new Date(created_at);
    $: time_since =
        Math.abs(new Date().getTime() - task_timestamp.getTime()) / 1000;

    // setInterval(() => (task_timestamp = task_timestamp), 1000);

    import { fly } from "svelte/transition";
    import { onMount } from "svelte";

</script>

<!-- <div id="container"> -->
<div transition:fly={{ x: 400, duration: 300 }} id="container">
    <div id="heading_container">
        <div>
            <span>{username} </span>
            {#if imageUrl}
                <img id="avatar" src={imageUrl} alt="profile" width="40" />
            {/if}
        </div>
        <div>Added <span>{time_since.toFixed(0)}s ago</span></div>
        <button on:click={buttonCallback}
            ><span class="material-icons">close</span></button
        >
    </div>
    <div class="body">{body}</div>
</div>

<style>
    #container {
        max-width: 30em;
        min-width: 30em;
        border: 2px solid black;
        margin: 5px;
        border-radius: 10px;
        padding: 0.1em;
        background-color: rgb(207, 239, 233);
    }

    button {
        padding: 0em;
        margin-left: 3em;
        border: 0px;
        text-align: center;
    }

    #heading_container {
        display: grid;
        grid-template-columns: 0.25fr 0.5fr 0.25fr;
        text-align: center;
        font-size: large;
        margin: 0.3em;
    }

    .material-icons {
        font-size: large;
    }

    img {
        border-radius: 2em;
    }

    span {
        vertical-align: middle;
        font-weight: 700;
    }

    .body {
        padding-left: 0.5em;
        padding-right: 0.5em;
    }
</style>
