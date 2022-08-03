<script lang="ts">
  import { sheetsApi } from "./axios/sheetsApi";

  let gclient;
  console.log({ google });
  let { spreadsheetId, access_token } = localStorage;
  console.log({ spreadsheetId, access_token });
  if (!spreadsheetId) {
    createSheet();
  }
  let notes = [];
  async function loadNotes() {
    const range = "Hoja 1!A:B";
    const res = await sheetsApi.get(`/${spreadsheetId}/values/${range}`);
    console.log({ res });
  }
  let title = "";
  async function createSheet() {
    const { data } = await sheetsApi.post("/", {
      properties: {
        title: "notes-app-db",
      },
    });
    localStorage.spreadsheetId = data.spreadsheetId;
    spreadsheetId = localStorage.spreadsheetId;

    console.log("new sheet creaed: ", { data });

    const range = "Hoja 1!A1:B1";
    const res = await sheetsApi.put(
      `/${spreadsheetId}/values/${range}`,
      {
        values: [["title", "content"]],
      },
      {
        params: {
          valueInputOption: "RAW",
        },
      }
    );
  }
  function oauth2Login() {
    gclient = google.accounts.oauth2.initTokenClient({
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
    gclient.requestAccessToken();
  }
  let content = "";
  function saveNote() {}
</script>

<main>
  <h2>Auth with google</h2>
  <button on:click={oauth2Login}>Login</button>
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
</main>

<style>
  input,
  textarea {
    display: block;
    margin: 0.5rem;
  }
</style>
