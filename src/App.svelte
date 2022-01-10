<script>
  import { onMount } from 'svelte'
  import { sortBy } from 'lodash'
  import PouchDB from 'pouchdb-browser'

  // On réutilise les méthodes et composants du fichier todo-item.svelte 
  import TodoItem from './todo-item.svelte'

  // Configuration de PouchDB localement et réplication remote continue à CouchDB
  let db = new PouchDB('db')
  const replication = PouchDB.sync('db', 'http://localhost:5984/svelte-todo-db', {
    live: true,
    retry: true
  }).on('change', async function (info) {
    await updateTodos()
  }).on('error', function (err) {
    console.log('Error de réplication :', err)
  })

  // Configuration des variables et defaults
  let newTodoText = ''
  let sortByWhat = 'createdAt'
  let filterByWhat = ''
  let isLoading = true
  // Tous les todos directement de PouchDB. Sorte et filtrage vont après
  let todos = []
  $: sortedAndFilteredTodos = sortBy(todos, [sortByWhat]).filter((todo) => {
    const [filterKey, filterValue] = filterByWhat.split(':')
    // Filtrage est fait uniquement quand il y a un groupe propre
    return filterKey && filterValue ? todo[filterKey].toString() === filterValue : true
  })

  async function updateTodos() {
    const allDocs = await db.allDocs({
      include_docs: true
    })
    todos = allDocs.rows.map(row => row.doc)
    isLoading = false
  }

  // Handlers pour ajouter, mettre à jour et supprimer les todos
  // Fonction pour ajouter des nouveaux todos
  async function add(event) {
    const newTodo = {
      text: newTodoText,
      complete: false,
      createdAt: new Date().toISOString()
    }
    const addition = await db.post(newTodo)
    if (addition.ok) {
      await updateTodos()
    }
    newTodoText = ''
  }

  // Fonction pour mettre à jour le status du todo
  async function updateStatus(event) {
    const { todo } = event.detail
    const update = await db.put(todo)
    if (update.ok) {
      await updateTodos()
    }
  }

  // Fonction pour supprimer un item
  async function removeItem(event) {
    const { todo: todoToRemove } = event.detail
    const removal = await db.remove(todoToRemove)
    if (removal.ok) {
      // Pour la suppression, on met à jour l'état local.
      todos = todos.filter((todo) => {
        return todo._id !== todoToRemove._id
        })
    }
  }

  // Charge todos en premier temps
  onMount(async () => {
    await updateTodos()
  })
</script>

<style>
  ul {
    list-style: none;
    padding: 0;
  }
  button {
    margin-left: 0.75em;
  }
  input[type='text'] {
    width: 440px;
  }
</style>

{#if isLoading}
  <h1>
    Charge de todos…
  </h1>
{:else}
  {#if todos.length === 0}
    <h1>
      Zero todos! Niquel ☻ 
    </h1>
  {:else}
    <h1>
      Affichant {sortedAndFilteredTodos.length} de {todos.length} todos
    </h1>
  {/if}
{/if}

<div>
  <label for="sorte">Sorte par:</label>
  <select bind:value={sortByWhat}>
    <option value='createdAt'>Heure</option>
    <option value='text'>Todo texte</option>
    <option value='complete'>Complete</option>
  </select>
</div>
<div>
  <label for="filtre">Filtre:</label>
  <select bind:value={filterByWhat}>
    <option value=''>Montrer tous les todos</option>
    <option value='complete:true'>Affichage de todos completés</option>
    <option value='complete:false'>Affichage de todos en attente</option>
  </select>
</div>

<ul>
  {#each sortedAndFilteredTodos as todo (todo._id)}
    <TodoItem todo={todo} on:remove={removeItem} on:update={updateStatus}/>
  {/each}
</ul>

<form on:submit|preventDefault={add}>
  <input type='text' bind:value={newTodoText}>
  <button type='submit'>➕ Ajout une nouvelle tâche</button>
</form>
