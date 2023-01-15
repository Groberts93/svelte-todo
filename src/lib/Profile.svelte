<script lang="ts">
    import { onMount } from "svelte";
    import type { AuthSession } from "@supabase/supabase-js";
    import { supabase } from "../supabaseClient";
    import { fly } from "svelte/transition";
    import ImageCanvas from "./ImageCanvas.svelte";

    export let session: AuthSession;

    let loading = false;
    let username: string | null = null;
    let website: string | null = null;
    let avatarUrl: string | null = null;
    let avatarImg: HTMLImageElement | null = null;

    let files: FileList;

    onMount(() => {
        getProfile();
        // getAvatar();
    });

    const getProfile = async () => {
        try {
            loading = true;
            const { user } = session;

            const { data, error, status } = await supabase
                .from("profiles")
                .select("username, website, avatar_url")
                .eq("id", user.id)
                .single();

            if (error && status !== 406) throw error;

            if (data) {
                username = data.username;
                website = data.website;
                await getAvatar(data.avatar_url);
            }
        } catch (error) {
            if (error instanceof Error) {
                alert(error.message);
            }
        } finally {
            loading = false;
        }
    };

    const updateProfile = async () => {
        try {
            loading = true;
            const { user } = session;

            const updates = {
                id: user.id,
                username,
                website,
                // avatar_url: avatarUrl,
                updated_at: new Date().toISOString(),
            };

            let { error } = await supabase.from("profiles").upsert(updates);

            const { data } = await supabase
                .from("profiles")
                .select("avatar_url")
                .eq("id", user.id)
                .single();

            const avatarFile = files[0];

            await supabase.storage
                .from("avatars")
                .upload(data.avatar_url, avatarFile, {
                    cacheControl: "3600",
                    upsert: true,
                });

            if (error) {
                throw error;
            }
        } catch (error) {
            if (error instanceof Error) {
                alert(error.message);
            }
        } finally {
            loading = false;
        }
    };

    const getAvatar = async (avatar: string) => {
        const { data } = await supabase.storage
            .from("avatars")
            .download(avatar);

        const url = URL.createObjectURL(data);
        avatarUrl = url;
    };
</script>

<div id="account">
    <form on:submit|preventDefault={() => updateProfile()} class="form-widget">
        {#if avatarUrl}
            <img src={avatarUrl} alt="user avatar" />
            <ImageCanvas src={avatarUrl} />
        {/if}

        <!-- {#if avatarImg} -->
        <!-- {/if} -->
        <div>Email: {session.user.email}</div>
        <div>
            <label for="username">Name</label>
            <input id="username" type="text" bind:value={username} />
        </div>
        <div>
            <label class="special" for="website">Website</label>
            <input id="website" type="text" bind:value={website} />
        </div>

        <div>
            <label for="avatar_upload">Avatar</label>
            <input
                id="avatar_upload"
                type="file"
                accept="image/png, image/jpeg"
                bind:files
            />
        </div>

        <div>
            <button
                type="submit"
                class="button primary block"
                disabled={loading}
            >
                {loading ? "Saving ..." : "Update profile"}
            </button>
        </div>
        <div>
            <button
                type="button"
                class="button block"
                on:click={() => supabase.auth.signOut()}
            >
                Sign Out
            </button>
        </div>
    </form>
</div>

<style>
    #account {
        border: solid 3px black;
        background-color: rgb(193, 207, 233);
        padding: 15px;
        border-radius: 15px;
        font-size: larger;
    }

    form div {
        display: table;
        padding: 5px;
    }

    form input {
        margin-left: 0.5em;
        font-size: large;
        border: 2px solid transparent;
        /* max-width: fit-content; */
        padding-left: 5px;
        border-radius: 5px;
    }

    button {
        padding: 10px;
    }

    button:hover {
        color: blue;
    }
</style>
