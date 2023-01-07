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
        replace('/homeFeed');
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

<div class="px-4 pt-5 mt-1 text-center ">
  <h1 class="display-4 fw-bold">Why don’t you</h1>
  <h1 class="display-4 fw-bold text-primary">
    Explain this to me like I’m five
  </h1>
  <div class="col-lg-6 mx-auto">
    <p class="lead mb-4 fs-6">
      Join the ELI5 club and explain questions like you would do to 5 year
      olds!!
    </p>

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
    <br />

    <!-- Google SignIn -->
  </div>

  <!-- Desktop -->

  <div
    id="carouselExampleControls"
    class="carousel slide d-none d-lg-block"
    data-bs-ride="carousel"
  >
    <div class="carousel-inner">
      <div class="carousel-item active" data-bs-interval="2500">
        <img
          src="/assets/images/carousel/img1.png"
          class="img-fluid w-75"
          alt="Eli5 - Put on your explaining hat and start answering"
        />
      </div>
      <div class="carousel-item" data-bs-interval="2500">
        <img
          src="/assets/images/carousel/img2.png"
          class="img-fluid w-75"
          alt="Eli5 - Share and challenge your friends to answer questions"
        />
      </div>
      <div class="carousel-item" data-bs-interval="2500">
        <img
          src="/assets/images/carousel/img3.png"
          class="img-fluid w-75"
          alt="Explore top answers and show off"
        />
      </div>
    </div>
    <button
      class="carousel-control-prev"
      type="button"
      data-bs-target="#carouselExampleControls"
      data-bs-slide="prev"
    >
      <span class="carousel-control-prev-icon" aria-hidden="true" />
      <span class="visually-hidden">Previous</span>
    </button>
    <button
      class="carousel-control-next"
      type="button"
      data-bs-target="#carouselExampleControls"
      data-bs-slide="next"
    >
      <span class="carousel-control-next-icon" aria-hidden="true" />
      <span class="visually-hidden">Next</span>
    </button>
  </div>

  <!-- Mobile -->

  <div
    id="carouselExampleControls"
    class="carousel slide d-lg-none"
    data-bs-ride="carousel"
  >
    <div class="carousel-inner">
      <div class="carousel-item active" data-bs-interval="2500">
        <img
          src="/assets/images/carousel/imgm1.png"
          class="img-fluid"
          alt="Eli5 - Put on your explaining hat and start answering"
        />
      </div>
      <div class="carousel-item" data-bs-interval="2500">
        <img
          src="/assets/images/carousel/imgm2.png"
          class="img-fluid"
          alt="Eli5 - Share and challenge your friends to answer questions"
        />
      </div>
      <div class="carousel-item" data-bs-interval="2500">
        <img
          src="/assets/images/carousel/imgm3.png"
          class="img-fluid"
          alt="Explore top answers and show off"
        />
      </div>
    </div>
    <button
      class="carousel-control-prev"
      type="button"
      data-bs-target="#carouselExampleControls"
      data-bs-slide="prev"
    >
      <span class="carousel-control-prev-icon" aria-hidden="true" />
      <span class="visually-hidden">Previous</span>
    </button>
    <button
      class="carousel-control-next"
      type="button"
      data-bs-target="#carouselExampleControls"
      data-bs-slide="next"
    >
      <span class="carousel-control-next-icon" aria-hidden="true" />
      <span class="visually-hidden">Next</span>
    </button>
  </div>
</div>
