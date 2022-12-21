<script>
  import Router from 'svelte-spa-router';
  import { replace } from 'svelte-spa-router';
  import axios from 'axios';
  import CompleteProfile from './lib/CompleteProfile.svelte';
  import Feed from './lib/Feed.svelte';
  import Home from './lib/Home.svelte';
  import PrivacyPolicy from './lib/PrivacyPolicy.svelte';
  import RulesOfExplaining from './lib/RulesOfExplaining.svelte';

  //for productionuncomment below line
  axios.defaults.baseURL = 'https://backend.eli5.club/api/v1';

  //for testing on localhost uncomment below line
  // axios.defaults.baseURL = 'http://localhost:8080/api/v1';
  axios.defaults.withCredentials = true;

  const routes = {
    '/': Home,

    '/completeProfile': CompleteProfile,

    '/feed': Feed,

    '/privacyPolicy': PrivacyPolicy,

    '/rules': RulesOfExplaining,
  };
</script>

<Router {routes} />

<!-- <CompleteProfile /> -->

<!-- <Feed /> -->
