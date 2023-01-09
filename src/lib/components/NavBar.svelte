<script>
  import { link, location } from 'svelte-spa-router';
</script>

<nav class="navbar navbar-light">
  <div class="container">
    <a class="navbar-brand" href={null}>
      <img src="/assets/images/logo.png" alt="" height="50" />
    </a>
    {#if $location != '/exploreQuestions'}
      <ul class="nav nav-pills">
        <li class="nav-item">
          <a class="nav-link" href="/rules" use:link>Rules</a>
        </li>
        <!-- <li class="nav-item">
          <a class="nav-link" href="/privacyPolicy" use:link>Privacy Policy</a>
        </li> -->
      </ul>
    {/if}
  </div>
</nav>
