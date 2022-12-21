<script>
  import { link } from 'svelte-spa-router';
</script>

<nav class="navbar navbar-light">
  <div class="container">
    <!--TODO: rediredct home page -->
    <a class="navbar-brand" href="/" use:link>
      <img src="/assets/images/logo.png" alt="" height="50" />
    </a>

    <ul class="nav nav-pills">
      <li class="nav-item">
        <a class="nav-link" href="/rules" use:link>Rules</a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="/privacyPolicy" use:link>Privacy Policy</a>
      </li>
    </ul>
  </div>
</nav>
