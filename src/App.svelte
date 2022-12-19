<script>
  import Router from 'svelte-spa-router';
  import { replace } from 'svelte-spa-router';
  import axios from 'axios';
  import CompleteProfile from './lib/CompleteProfile.svelte';
  import Feed from './lib/Feed.svelte';
  import Home from './lib/Home.svelte';

  axios.defaults.baseURL = 'https://backend.eli5.club/api/v1';
  axios.defaults.withCredentials = true;

  const routes = {
    '/': Home,

    '/completeProfile': CompleteProfile,

    '/feed': Feed,
  };
  function getCookie(name) {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) {
      return parts.pop().split(';').shift();
    } else {
      return false;
    }
  }
  if (getCookie('loginState') != false) {
    replace('/feed');
  }
</script>

<Router {routes} />

<!-- <CompleteProfile /> -->

<!-- <Feed /> -->
