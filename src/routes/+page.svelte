
<script>

  import { onMount }    from 'svelte';

  import { Button }     from "$lib/components/ui/button";
  import { Textarea }   from "$lib/components/ui/textarea";
  import { Input }      from "$lib/components/ui/input";

  let key = "";                   // Track the current key in use
  let value = "";                 // Track the current value in the textarea
  let noteArray = [];             // Populate this array with your retrieved notes
  let localStorageKeys = [];      // Populate this array with your stored keys
  let isUpdateMode = false;       // Track whether we're in "Update" mode

  // Load keys from localStorage on component mount
  onMount(() => {
    localStorageKeys = Object.keys(localStorage);
  });

  function resetForm() {
    key = "";
    value = "";
    isUpdateMode = false;
  }

  function saveData() {
    if (isUpdateMode) { // Update mode: save the changes to the selected key
      try {
        const dataArray = value
          .split('\n')
          .filter(line => line.trim() !== '')
          .map((blob, index) => ({ 
            id: index + 1, blob 
          }));
        localStorage.setItem(key, JSON.stringify(dataArray));
      } catch (error) {
        console.error("Error updating data:", error);
      }

    } else { // Save mode: save new data to a new key
      try {
        const dataArray = value
          .split('\n')
          .map((blob, index) => ({ 
            id: index + 1, blob 
          }));
        localStorage.setItem(key, JSON.stringify(dataArray));
        // Retrieve updated keys
        localStorageKeys = Object.keys(localStorage);
      } catch (error) {
        console.error("Error saving data:", error);
      }
    }

    resetForm();
  }

  function retrieveData(storageKey) {
    try {
      const rawData = localStorage.getItem(storageKey);
      const data = JSON.parse(rawData);
      if (Array.isArray(data)) {
        noteArray = data;
        key = storageKey;
        value = data.map(note => note.blob).join('\n');
        isUpdateMode = true;
      } else {
        console.error("Expected an array but got:", data);
      }
    } catch (error) {
      console.error("Error parsing JSON from localStorage:", error);
    }
  }

  function deleteKey(storageKey) {
    localStorage.removeItem(storageKey);
    localStorageKeys = localStorageKeys.filter(k => k !== storageKey);
    resetForm();
  }
</script>


<section class="p-6 bg-gray-50 rounded-lg shadow-md">
  <!-- Form Section -->
  <form class="space-y-4" on:submit|preventDefault={saveData}>
    <div class="flex flex-col">
      <Input required type="text" id="key" 
        class="input input-bordered w-full mt-1"
        placeholder="Key"
        disabled={isUpdateMode}
        bind:value={key} />
    </div>

    <div class="flex flex-col">
      <Textarea required id="value" 
        class="textarea textarea-bordered w-full mt-1" 
        placeholder="Enter list of items, one per line"
        bind:value={value} />
    </div>

    <Button type="submit" class="btn btn-primary w-full" > 
      {isUpdateMode ? "Update" : "Save"}
      </Button>
  </form>

  <!-- Available Keys Section -->
  <h2 class="text-lg font-semibold mt-6">Available Keys</h2>
  <ul class="mt-2 space-y-2">
    {#each localStorageKeys as storageKey}
      <li class="flex items-center justify-between">
        <a href="##" class="text-blue-500 hover:underline"
          on:click={(event)=>{
            event.preventDefault();
            retrieveData(storageKey);
          }} >
          {storageKey}
          </a>
        <Button class="btn btn-danger ml-4"
          on:click={(event) => {
            event.preventDefault();
            deleteKey(storageKey);
          }} >
          Delete
          </Button>
      </li>
    {/each}
  </ul>

</section>


<style>

</style>