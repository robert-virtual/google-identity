<script lang="ts">
  import { sheetsApi } from "../axios/sheetsApi";

  import { useLocation, useNavigate } from "svelte-navigator";
  let { spreadsheetId, access_token } = localStorage;

  let note = ["", ""];
  let id = 1
  const location = useLocation();
  const navigate = useNavigate();
  console.log(!$location.state);
  $: if (!$location.state) {
    navigate(-1);
  } else {
    note = $location.state.note;
    id = $location.state.id;
  }
  async function deleteNote() {
    const range = `!A${id}:B${id}`;
    sheetsApi.post(
      `/${spreadsheetId}/values/${range}:clear`,
    );
  }
  async function saveNote() {
    const range = `!A${id}:B${id}`;
    sheetsApi.put(
      `/${spreadsheetId}/values/${range}`,
      {
        values: [note],
      },
      {
        params: {
          valueInputOption: "RAW",
        },
      }
    );
  }
</script>

<div class="note">
  <input type="text" bind:value={note[0]} placeholder="Title" />
  <textarea bind:value={note[1]} placeholder="Content" />
  <button on:click={saveNote}>save</button>
  <button on:click={deleteNote}>Delete</button>
</div>

<style>
  input,
  textarea {
    display: block;
    margin: 0.5rem;
  }
  .note {
    background-color: #333;
    padding: 0.5rem;
    border-radius: 0.5rem;
    width: 15rem;
  }
</style>
