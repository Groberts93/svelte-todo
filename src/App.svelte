<script lang="ts">
  import { onMount } from "svelte";
  import { supabase } from "./supabaseClient";
  import type { AuthSession } from "@supabase/supabase-js";
  import Profile from "./lib/Profile.svelte";
  import Auth from "./lib/Auth.svelte";
  import Tasks from "./lib/Tasks.svelte";


  let session: AuthSession;

  let pageComponents = [
    { id: "Profile", comp: Profile },
    { id: "Tasks", comp: Tasks },
  ];

  let ind = 0;

  function incrementPage(count) {
    let nextInd = ind + count;

    if (nextInd > 1) {
      nextInd = 1;
    }

    if (nextInd < 0) {
      nextInd = 0;
    }

    ind = nextInd;
  }

  onMount(() => {
    supabase.auth.getSession().then(({ data }) => {
      session = data.session;
    });

    supabase.auth.onAuthStateChange((_event, _session) => {
      session = _session;
    });
  });
</script>

<nav>
  <button id="left" on:click={() => incrementPage(-1)}
    ><span class="material-icons">arrow_back</span></button
  >
  <p id="pagename">{pageComponents[ind].id}</p>
  <button id="right" on:click={() => incrementPage(1)}
    ><span class="material-icons">arrow_forward</span></button
  >
</nav>

<div class="container" style="padding: 0px 0 100px 0">
  {#if !session}
    <Auth />
  {:else}
    <!-- {#each pageComponents as component (component.id)} -->
    <svelte:component this={pageComponents[ind].comp} session={session} />
  {/if}
</div>

<style>
  nav {
    display: grid;
    grid-template-columns: 1fr 4fr 1fr;
    margin-bottom: 1em;
    min-width: 30em;
    max-width: 30em;
  }

  #pagename {
    font-size: x-large;
  }
</style>
