<script>
  import { replace } from 'svelte-spa-router';
  import NavBar from './components/NavBar.svelte';

  function goHome() {
    replace('/');
  }
</script>

<NavBar />

<div class="px-4 pt-5 mt-5 text-center border-bottom">
  <div class="container">
    <div class="container px-5">
      <img
        src="/assets/images/404.png"
        class="img-fluid border rounded-3 shadow-lg mb-4"
        alt="Not Found"
        width="300"
        height="200"
        loading="lazy"
      />
    </div>
  </div>
  <h1 class="display-4 fw-bold">404 page not found!</h1>
  <div class="col-lg-6 mx-auto">
    <p class="lead mb-4">
      <b>What is a 404 page?</b><br />A 404 error indicates that the webpage
      you're trying to reach can't be found. When your favourite store changes
      its location, they add a sign on its previous location so that you can
      navigate to the new store location. That's what 404 pages are.
    </p>
    <div class="d-grid gap-2 d-sm-flex justify-content-sm-center mb-5">
      <button
        on:click={goHome}
        type="button"
        class="btn btn-primary btn-lg px-4 me-sm-3">Go home</button
      >
    </div>
  </div>
</div>
