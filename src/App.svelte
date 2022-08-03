<script lang="ts">
  import { Router, Route, Link } from "svelte-navigator";
  import Inicio from "./pages/inicio.svelte";
  import Note from "./pages/Note.svelte";
</script>

<Router>
  <header>
    <nav>
      <Link to="/">Home</Link>
    </nav>
  </header>
  <main>
    <Route path="/">
      <Inicio />
    </Route>
    <Route path="note">
      <Note />
    </Route>
  </main>
</Router>

<style>
</style>
