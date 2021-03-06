<script>
  import { beforeUpdate, afterUpdate, onMount } from "svelte";
  import { fade, fly } from "svelte/transition";
  import { quintOut } from "svelte/easing";
  import { crossfade } from "svelte/transition";
  import { flip } from "svelte/animate";
  import { user } from "./user-store.js";
  import { store } from "./gun-store";
  import Page from "./Page.svelte";
  import Nav from "./Nav.svelte";
  import Input from "./Input.svelte";

  let msgInput;
  let div;
  let autoscroll;

  beforeUpdate(() => {
    autoscroll =
      div && div.offsetHeight + div.scrollTop > div.scrollHeight - 50;
  });

  afterUpdate(() => {
    if (autoscroll) div.scrollTo(0, div.scrollHeight);
  });

  onMount(() => {
    if (div) div.scrollTo(0, div.scrollHeight);
  });

  const [send, receive] = crossfade({
    duration: d => Math.sqrt(d * 200),

    fallback(node, params) {
      const style = getComputedStyle(node);
      const transform = style.transform === "none" ? "" : style.transform;

      return {
        duration: 600,
        easing: quintOut,
        css: t => `
					transform: ${transform} scale(${t});
					opacity: ${t}
				`
      };
    }
  });

  function toHSL(str) {
    const opts = {
      hue: [60, 360],
      sat: [75, 100],
      lum: [70, 71]
    };

    function range(hash, min, max) {
      const diff = max - min;
      const x = ((hash % diff) + diff) % diff;
      return x + min;
    }

    let hash = 0;
    if (str === 0) return hash;

    for (let i = 0; i < str.length; i++) {
      hash = str.charCodeAt(i) + ((hash << 5) - hash);
      hash = hash & hash;
    }

    let h = range(hash, opts.hue[0], opts.hue[1]);
    let s = range(hash, opts.sat[0], opts.sat[1]);
    let l = range(hash, opts.lum[0], opts.lum[1]);

    return `hsl(${h}, ${s}%, ${l}%)`;
  }
</script>

<style>
  main {
    display: flex;
    height: 100%;
    flex-direction: column;
    background-color: white;
  }
  .scrollable {
    flex: 1 1 auto;
    margin: 0;
    padding: 0.5em 1em;
    overflow-y: auto;
  }

  article {
    margin: 1.5em 0;
  }

  .msg {
    display: inline-block;
    position: relative;
    line-height: 1.8;
    padding: 0.4em 1em;
    background-color: #eee;
    border-radius: 1em 1em 1em 0;
    hyphens: auto;
    word-break: break-word;
  }

  .user {
    text-align: right;
  }

  .user > .msg {
    margin-left: 4em;
    background-color: #0074d9;
    color: white;
    border-radius: 1em 1em 0 1em;
  }

  .msg:hover button {
    opacity: 1;
  }

  button.delete {
    position: absolute;
    top: 0;
    right: 0.2em;
    width: 2em;
    height: 100%;
    background: no-repeat 50% 50% url(/trash.svg);
    background-size: 1.4em 1.4em;
    border: none;
    opacity: 0;
    transition: opacity 0.2s;
    text-indent: -9999px;
    cursor: pointer;
  }
  .submit-form {
    padding: 0 1em 0.5em 1em;
  }
  .meta {
    font-size: 10px;
    margin: 0.5em;
  }
</style>

<Page>
  <Nav backTo="settings">Messages</Nav>
  <main>
    <div class="scrollable" bind:this={div}>
      {#each $store as val (val.msgId)}
        <article
          class:user={val.user === $user}
          animate:flip
          in:receive={{ key: val.msgId }}
          out:fade>
          <div class="meta">
            <span class="time">
              {new Date(val.time).toLocaleString('en-US')}
            </span>
            <span class="user">{val.user}</span>
          </div>
          <div
            class="msg"
            style="background-color: {val.user !== $user && toHSL(val.user)}">
            {val.msg}
            <button
              class="delete"
              on:click|preventDefault={() => {
                const yes = confirm('Are you sure?');
                if (yes) store.delete(val.msgId);
              }}>
              delete
            </button>
          </div>
        </article>
      {/each}
    </div>

    <form
      class="submit-form"
      method="get"
      on:submit|preventDefault={e => {
        if (!msgInput) return;
        $store = { msg: msgInput, user: $user };
        msgInput = '';
        e.target.msg.focus();
      }}>
      <Input
        bind:value={msgInput}
        placeholder="Message"
        ariaLabel="Message"
        name="msg" />
    </form>

  </main>
</Page>
