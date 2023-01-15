<script lang="ts">
    import Card from "./Card.svelte";
    import { onMount } from "svelte";
    import { fly } from "svelte/transition";

    import { supabase } from "../supabaseClient";

    import type { AuthSession } from "@supabase/supabase-js";

    export let session: AuthSession;

    let loading = false;
    let cardData = [];
    let avatars = {};

    let body = "";

    const getTasks = async () => {
        try {
            loading = true;
            const { user } = session;
            const { data, error, status } = await supabase
                .from("client_tasks_view")
                .select("id, username, body, created_at")
                .order("id", { ascending: false });

            if (error) throw error;

            if (data) {
                console.log(data);
                cardData = data;
            }
        } catch (error) {
            console.log(error);
        } finally {
            loading = false;
        }
    };

    const getUniqueProfiles = async () => {
        const response = await supabase
            .from("unique_avatar_urls")
            .select("username, avatar_url");

        for (const record of response.data) {
            const { data } = await supabase.storage
                .from("avatars")
                .download(record.avatar_url);
            const url = URL.createObjectURL(data);
            avatars[record.username] = url;
        }
        avatars = avatars;
        console.log(avatars);
        // avatars = {[response.data[0].username]: URL.createObjectURL(data)};
    };

    const addTask = async () => {
        const { user } = session;

        const updates = {
            user_id: user.id,
            body,
        };
        const { error } = await supabase.from("tasks").upsert(updates);
    };

    const removeTask = async (id: string) => {
        const { error } = await supabase.from("tasks").delete().eq("id", id);
    };

    onMount(() => getUniqueProfiles().then(getTasks));
</script>

<!-- <div in:fly={{ x: 400, duration: 1000 }} out:fly={{ x: -400, duration: 1000 }}> -->
<div id="container">
    <form on:submit|preventDefault={() => addTask().then(getTasks)}>
        <label for="task">{loading ? "Loading..." : "Add new task:"} </label>
        <input id="task" type="text" bind:value={body} />
        <input type="submit" hidden />
    </form>
    <div id="cards">
        {#each cardData as { id, ...card } (id)}
            <Card
                {...card}
                buttonCallback={() => removeTask(id).then(getTasks)}
                imageUrl={avatars[card.username]}
            />
        {/each}
    </div>
</div>

<style>
    /* #container {
        border: 2px solid black;
    } */

    form {
        font-size: larger;
    }
</style>
