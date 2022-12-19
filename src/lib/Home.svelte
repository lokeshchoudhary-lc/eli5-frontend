<script>
  import { replace } from 'svelte-spa-router';
  import axios from 'axios';
  import NavBar from './components/NavBar.svelte';
  import { firstName, email } from './store';

  function decodeJwtResponse(token) {
    let base64Url = token.split('.')[1];
    let base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
    let jsonPayload = decodeURIComponent(
      atob(base64)
        .split('')
        .map(function (c) {
          return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
        })
        .join('')
    );
    return JSON.parse(jsonPayload);
  }

  async function checkEmail(email) {
    try {
      const response = await axios.get(`/userCheck/${email}`);
      if (response.data == 'go_to_feed') {
        replace('/feed');
      }
      if (response.data == 'go_to_completeProfile') {
        replace('/completeProfile');
      }
    } catch (err) {
      console.log(err);
    }
  }

  globalThis.handleCredentialResponse = async (response) => {
    let responsePayload = decodeJwtResponse(response.credential);

    let firstNameRes = responsePayload.name.split(' ')[0];
    let emailRes = responsePayload.email;
    firstName.set(firstNameRes);
    email.set(emailRes);
    await checkEmail(emailRes);
  };
</script>

<svelte:head>
  <script src="https://accounts.google.com/gsi/client" async defer></script>
</svelte:head>

<NavBar />

<div class="px-4 pt-5 mt-3 text-center">
  <h1 class="display-4 fw-bold">Why don’t you</h1>
  <h1 class="display-4 fw-bold text-primary">
    Explain this to me like I’m five
  </h1>
  <div class="col-lg-6 mx-auto">
    <p class="lead mb-4">
      Join the ELI5 club and explain questions like you would do to 5 year
      olds!!
    </p>

    <!-- Google SignIn -->
    <!-- <div class="d-grid gap-2 d-sm-flex justify-content-sm-center mb-2">
      <a href="/completeProfile" class="text-decoration-none" use:link>
        <button class="btn btn-primary btn-lg px-4 py-3 me-sm-3">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="16"
            height="16"
            fill="currentColor"
            class="bi bi-google"
            viewBox="0 0 16 16"
          >
            <path
              d="M15.545 6.558a9.42 9.42 0 0 1 .139 1.626c0 2.434-.87 4.492-2.384 5.885h.002C11.978 15.292 10.158 16 8 16A8 8 0 1 1 8 0a7.689 7.689 0 0 1 5.352 2.082l-2.284 2.284A4.347 4.347 0 0 0 8 3.166c-2.087 0-3.86 1.408-4.492 3.304a4.792 4.792 0 0 0 0 3.063h.003c.635 1.893 2.405 3.301 4.492 3.301 1.078 0 2.004-.276 2.722-.764h-.003a3.702 3.702 0 0 0 1.599-2.431H8v-3.08h7.545z"
            />
          </svg>
          Sign in with Google
        </button></a
      >
    </div> -->
    <!-- Google SignIn -->

    <!-- Google SignIn -->

    <div class="d-grid gap-2 d-sm-flex justify-content-center mb-2">
      <div
        id="g_id_onload"
        data-client_id="267296457508-3aijf171n0nge0qu32fti4ttn1o5cv52.apps.googleusercontent.com"
        data-context="signin"
        data-ux_mode="popup"
        data-callback="handleCredentialResponse"
        data-auto_prompt="false"
      />

      <div
        class="g_id_signin"
        data-type="standard"
        data-shape="pill"
        data-theme="filled_blue"
        data-text="continue_with"
        data-size="large"
        data-logo_alignment="left"
      />
    </div>

    <!-- Google SignIn -->

    <div class="container mt-5 mb-5">
      <a href="/VIDEO-LINK-LOGIC-HERE"
        ><i
          class="bi bi-play-circle"
          style="color: #3366FF; font-size: xx-large; padding-right: 10px;"
        /> See how it works</a
      >
    </div>
  </div>
  <div class="container" style="width: 100%">
    <img
      src="/assets/images/testm.png"
      class="img-fluid rounded-3 d-lg-none"
      loading="lazy"
      alt=""
    />
    <img
      src="/assets/images/testimonial.png"
      class="img-fluid rounded-3 d-none d-lg-block"
      loading="lazy"
      alt=""
    />
  </div>
</div>
