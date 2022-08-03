<script lang="ts">
  let access_token = "";
  let gclient;
  console.log({ google });
  async function initClient() {
    await gapi.client.init({
      apiKey: "AIzaSyCSn8kzs78gr1npZD-J-VwSbiyRp_KupCs",
      clientId:
        "1047629314149-f5sp9o7ol8ta174r41hur8untntt188j.apps.googleusercontent.com",
      discoveryDocs: [
        "https://sheets.googleapis.com/$discovery/rest?version=v4",
      ],
      scope: "https://www.googleapis.com/auth/spreadsheet",
      // scope:"profile"
    });
  }
  gapi.load("client", initClient);

  let title = "";
  async function createSheet() {
    if (!title) {
      console.log("title empty");
      return;
    }
    if (!gapi.client.sheets.spreadsheets) {
      console.log("spreadsheets not initialized");
      return;
    }
    gapi.client.setToken({ access_token });
    // gapi.auth.setToken(tokenInfo);
    const newSheet = await gapi.client.sheets.spreadsheets.create({
      properties: {
        title,
      },
    });

    console.log({ newSheet });
  }
  function oauth2Login() {
    gclient = google.accounts.oauth2.initTokenClient({
      client_id:
        "1047629314149-f5sp9o7ol8ta174r41hur8untntt188j.apps.googleusercontent.com",
      scope: "https://www.googleapis.com/auth/spreadsheets",
      callback: (token) => {
        access_token = token.access_token;
        console.log({ token });
      },
    });
    gclient.requestAccessToken();
  }
  let noteTitle = "";
  let noteContent = "";
  function saveNote() {
    
  }
</script>

<main>
  <h2>Auth with google</h2>
  <button on:click={oauth2Login}>Login</button>
  <hr />
  <input type="text" bind:value={title} placeholder="New Sheet title" />
  <button on:click={createSheet}>create sheet</button>
  <hr />
  <h3>Create note</h3>
  <input type="text" bind:value={noteTitle} placeholder="Note title" />
  <textarea bind:value={noteContent}  cols="30" rows="10" placeholder="Note content"></textarea>
  <button on:click={saveNote}>save</button>

</main>

<style>
</style>
