<script lang="ts">
  let buttonDiv: HTMLDivElement;
  let email = "";
  import { decodeJwt } from "jose";
  let access_token = "";
  let tokenInfo = {}
  function handleCredentialResponse(res) {
    console.log(res);
    access_token = res.credential;
    const resPayload = decodeJwt(res.credential);
    console.log({ resPayload });
    tokenInfo = resPayload
    email = resPayload.email as string;
  }
  console.log({ google });
  google.accounts.id.initialize({
    client_id:
      "1047629314149-f5sp9o7ol8ta174r41hur8untntt188j.apps.googleusercontent.com",
    callback: handleCredentialResponse,
  });
  $: if (buttonDiv) {
    google.accounts.id.renderButton(buttonDiv, {
      theme: "outline",
      size: "large",
    });
  }

  google.accounts.id.prompt();

  function logout() {
    google.accounts.id.revoke(email, (done) => {
      console.log({ done });
    });
  }
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
    // gapi.client.setToken({ access_token });
    gapi.auth.setToken({
      access_token,
      error:"" ,
      state:"",
      expires_in:tokenInfo.exp
    });
    const newSheet = await gapi.client.sheets.spreadsheets.create({
      properties: {
        title,
      },
    });

    console.log({ newSheet });
  }
  async function initClient() {
    await gapi.client.init({
      // apiKey: "AIzaSyCSn8kzs78gr1npZD-J-VwSbiyRp_KupCs",
      clientId:
        "1047629314149-f5sp9o7ol8ta174r41hur8untntt188j.apps.googleusercontent.com",
      discoveryDocs: [
        "https://sheets.googleapis.com/$discovery/rest?version=v4",
      ],
      scope: "https://www.googleapis.com/auth/spreadsheet",
    });
  }
  gapi.load("client", initClient);
</script>

<main>
  <h2>Auth with google</h2>
  <div bind:this={buttonDiv} />
  <button on:click={google.accounts.id.storeCredential}>store credential</button
  >
  <button on:click={logout}>revoke</button>
  <hr />
  <input type="text" bind:value={title} placeholder="New Sheet title" />
  <button on:click={createSheet}>create sheet</button>
</main>

<style>
</style>

