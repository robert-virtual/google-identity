<script lang="ts">
  let gclient;
  console.log({ google });
  let { spreadsheetId, access_token } = localStorage;
  let gapiLoaded = false;
  async function initClient() {
    try {
      await gapi.client.init({
        apiKey: "AIzaSyCSn8kzs78gr1npZD-J-VwSbiyRp_KupCs",
        clientId:
          "1047629314149-f5sp9o7ol8ta174r41hur8untntt188j.apps.googleusercontent.com",
        discoveryDocs: [
          "https://sheets.googleapis.com/$discovery/rest?version=v4",
        ],
        scope: "https://www.googleapis.com/auth/spreadsheet",
      });
    } catch (error) {
      console.warn(error);
    }
    gapiLoaded = true;
    console.log("gapi.client loaded", gapi.client);

    if (!access_token) {
      console.log("not access token available");
      return;
    }
    gapi.client.setToken({ access_token });
    console.log("sheetId: ", spreadsheetId);
    if (!spreadsheetId) {
      await createSheet();
    }
  }
  $: if (gapiLoaded) {
    console.log("gapi loaded");
  }
  let notes = [];
  gapi.load("client", initClient);
  async function loadNotes() {
    console.log("gapi.client", gapi.client);
    const res = await gapi.client.sheets.spreadsheets.values.get({
      spreadsheetId,
      range: "Hoja 1!A:B",
    });
    notes = res.result.values;
  }
  let title = "";
  async function createSheet() {
    if (!gapi.client) {
      console.log("gapi.client not initialized");
      return;
    }
    const newSheet = await gapi.client.sheets.spreadsheets.create({
      properties: {
        title: "notes-app-db",
      },
    });
    localStorage.spreadsheetId = newSheet.result.spreadsheetId;
    spreadsheetId = localStorage.spreadsheetId;

    console.log("new sheet creaed: ",{ newSheet });
    const res = await gapi.client.sheets.spreadsheets.values.update({
      spreadsheetId,
      range: "Hoja 1!A1:B1",
      resource: {
        values: [["title", "content"]],
      },
    });
  }
  function oauth2Login() {
    gclient = google.accounts.oauth2.initTokenClient({
      client_id:
        "1047629314149-f5sp9o7ol8ta174r41hur8untntt188j.apps.googleusercontent.com",
      scope: "https://www.googleapis.com/auth/spreadsheets",
      callback: (token) => {
        localStorage.access_token = token.access_token;
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
