<script>
  import Router from 'svelte-spa-router';
  import { replace } from 'svelte-spa-router';
  import axios from 'axios';
  import CompleteProfile from './lib/CompleteProfile.svelte';
  import Feed from './lib/Feed.svelte';
  import Feed2 from './lib/Feed2.svelte';
  import Error from './lib/404.svelte';
  import Home from './lib/Home.svelte';
  import Garage from './lib/garage.svelte';

  axios.defaults.baseURL = 'http://localhost:8080/api/v1';
  axios.defaults.withCredentials = true;

  const routes = {
    '/': Home,

    '/completeProfile': CompleteProfile,

    '/feed': Feed,
    
    '/feed2': Feed2,

    '/404': Error,

    '/garage': Garage,
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

<!-- <Feed2 /> -->