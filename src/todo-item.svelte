<script>
  import { createEventDispatcher } from 'svelte'
  import { debounce } from 'lodash'

  const dispatch = createEventDispatcher()

  function remove() {
    dispatch('remove', {todo})
  }

  function updateText() {
    dispatch('update', {todo})
  }
  // We don’t want to clobber the local DB, so we debounce saving on every keystroke
  const debouncedUpdateText = debounce(updateText, 500)

  function toggleStatus() {
    dispatch('update', {
      todo: {
        ...todo,
        complete: !todo.complete
      }
    })
  }

  export let todo
</script>

<style>
  .is-complete {
    text-decoration: line-through;
    color: green;
    width: 440px;
    display: inline-block;
  }
  input[type="text"] {
    width: 440px;
  }
  input[disabled] {
    background: none;
    border: 1px solid #0000;
  }
  button {
    border-radius: 50%;
    width: 2.25em;
    height: 2.25em;
    line-height: 1.3em;
    text-align: center;
    margin-left: 0.75em;
    padding: 0;
  }
</style>

<li>
  {#if todo.complete}
    <input class='is-complete' value={todo.text} disabled />
    <button on:click={toggleStatus}>❌</button>
  {:else}
    <input type='text' on:keyup={debouncedUpdateText} bind:value={todo.text}>
    <button on:click={toggleStatus}>✔️</button>
  {/if}
  <button on:click={remove}>💥</button>
</li>
