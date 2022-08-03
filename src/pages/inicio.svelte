<script lang="ts">
  import { useNavigate } from "svelte-navigator";
  import { sheetsApi } from "../axios/sheetsApi";
  let { spreadsheetId, access_token } = localStorage;

  let gclient = google.accounts.oauth2.initTokenClient({
    client_id:
      "1047629314149-f5sp9o7ol8ta174r41hur8untntt188j.apps.googleusercontent.com",
    scope: "https://www.googleapis.com/auth/spreadsheets",
    callback: (token) => {
      localStorage.access_token = token.access_token;
      access_token = localStorage.access_token;
      sheetsApi.defaults.params = {
        access_token,
      };
    },
  });
  console.log({ google });
  $: if (access_token && !spreadsheetId) {
    createSheet();
  } else if (access_token && spreadsheetId) {
    loadNotes();
  }
  let notes: [string, string][] = [];
  async function loadNotes() {
    const range = "!A:B";
    try {
      const { data } = await sheetsApi.get(`/${spreadsheetId}/values/${range}`);
      console.log("notes: ", { data });
      data.values.shift();
      notes = data.values;
    } catch (error) {
      console.warn(error.message);
      localStorage.removeItem("access_token");
    }
  }
  let title = "";
  async function createSheet() {
    const { data } = await sheetsApi.post("/", {
      properties: {
        title: "notes-app-db",
      },
    });

    console.log("new sheet created: ", { data });

    const range = "!A1:B1";
    await sheetsApi.put(
      `/${data.spreadsheetId}/values/${range}`,
      {
        values: [["title", "content"]],
      },
      {
        params: {
          valueInputOption: "RAW",
        },
      }
    );
    localStorage.spreadsheetId = data.spreadsheetId;
    spreadsheetId = localStorage.spreadsheetId;
  }
  function oauth2Login() {
    gclient.requestAccessToken();
  }
  let content = "";
  async function saveNote() {
    const range = "!A:B";
    const { data } = await sheetsApi.post(
      `/${spreadsheetId}/values/${range}:append`,
      {
        values: [[title, content]],
      },
      {
        params: {
          valueInputOption: "RAW",
        },
      }
    );
    console.log("append: ", { data });
    notes = [...notes, [title, content]];
  }
  const navigate = useNavigate();
  function logout() {
    localStorage.removeItem("access_token");
    localStorage.removeItem("spreadsheetId");
    access_token = null
    spreadsheetId = null
  }
</script>

{#if !access_token}
  <h2>Auth with google</h2>
  <button on:click={oauth2Login}>Login</button>
{:else}
  <button on:click={logout}>Logout</button>
  <hr />
  <h3>Create note</h3>
  <input type="text" bind:value={title} placeholder="Note title" />
  <textarea
    bind:value={content}
    cols="30"
    rows="10"
    placeholder="Note content"
  />
  <button on:click={saveNote}>save</button>
  <div class="grid">
    {#each notes as note, i}
      <div
        class="note"
        on:click={() => navigate("note", { state: { id: i + 2, note } })}
      >
        <h3>{note[0]}</h3>
        <h3>{note[1]}</h3>
      </div>
    {/each}
  </div>
{/if}

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
  .grid {
    display: grid;
    margin-top: 1rem;
    width: 80vw;
    grid-template-columns: repeat(auto-fit, minmax(15rem, 1fr));
    gap: 0.5rem;
  }
</style>
