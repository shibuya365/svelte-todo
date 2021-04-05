<script>
  import { onMount } from 'svelte';
  import { items } from '../stores';
  import TodoApi from '../TodoApi';
  import Item from './Item.svelte';
  import Newitem from './Newitem.svelte';
  import { v4 as uuidv4 } from 'uuid';

  function handleNewItem(e) {
    // console.log(e);
    $items = [
      {
        id: uuidv4(),
        text: e.detail,
        completed: false,
      },
      ...$items,
    ];
    TodoApi.save($items);
  }
  function handleUpdate(e) {
    const index = $items.findIndex((item) => item.id === e.detail.id);
    $items[index] = e.detail;
    TodoApi.save($items);
  }
  function handleDelete(e) {
    $items = $items.filter((item) => item.id !== e.detail);
    TodoApi.save($items);
  }

  let itemSorted = [];
  $: {
    itemSorted = [...$items];
    itemSorted.sort((a, b) => {
      if (a.completed && b.completed) return 0;
      if (a.completed) return 1;
      if (b.completed) return -1;
    });
  }
  onMount(async () => {
    // fetch from API
    $items = await TodoApi.getAll();
    // $items = [];
  });
</script>

<div class="list">
  <Newitem on:newitem={handleNewItem} />

  {#each itemSorted as item (item)}
    <Item {...item} on:update={handleUpdate} on:delete={handleDelete} />
  {:else}
    <p class="list-status">No Items Exist</p>
  {/each}
</div>

<style>
  .list {
    padding: 15px;
  }
  .list-status {
    margin: 0;
    text-align: center;
    color: #ffffff;
    font-weight: bold;
    font-size: 1.1em;
  }
</style>
