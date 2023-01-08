<script>
  import axios from 'axios';
  import { onMount } from 'svelte';
  import { link, push, replace } from 'svelte-spa-router';
  import { userChoosenQuestionState, userChoosenTagState } from './store';

  let profileUrl = '/assets/images/profile/';

  let profilePictureCode = 1;
  let firstName = '';
  let totalLikes = '';
  let totalAnswers = '';
  let streak = '';

  let greet;
  let icon;

  let bestAnswer = {};
  let topTags = [];
  let topQuestions = [];
  let banners = [];

  let page = 1;
  let loadMore = true;
  let allTags = [];
  let explore = [];

  let leaderboard = [];

  let smleaderboard = [];

  function chooseBestAnswer() {
    let question = {
      id: bestAnswer.questionId,
      question: bestAnswer.question,
    };
    userChoosenQuestionState.set(question);
    userChoosenTagState.set(bestAnswer.tag);
    localStorage.setItem('userChoosenTag', bestAnswer.tag);
    push('/feed');
  }

  function chooseQuestion(event) {
    let questionId = event.currentTarget.id;
    let index = topQuestions.findIndex((question) => questionId == question.id);
    let question = topQuestions[index];
    userChoosenQuestionState.set(question);
    userChoosenTagState.set(question.tag);
    localStorage.setItem('userChoosenTag', question.tag);
    push('/feed');
  }

  function chooseExploreTag(event) {
    let tag = event.currentTarget.id;
    userChoosenTagState.set(tag);
    localStorage.setItem('userChoosenTag', tag);
    replace('/exploreQuestions');
  }

  function chooseTag(event) {
    let tag = event.currentTarget.id;
    userChoosenTagState.set(tag);
    userChoosenQuestionState.set(null);
    localStorage.clear();
    localStorage.setItem('userChoosenTag', tag);
    push('/feed');
  }

  function logout() {
    axios
      .get('/logout')
      .then(function (response) {
        // console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });

    replace('/');
  }

  async function getExploreTags() {
    try {
      const response = await axios.get(`/explore`);
      // console.log(response);
      explore = [...response.data];
    } catch (error) {
      console.log(error);
    }
  }

  async function getAllTags() {
    try {
      const response = await axios.get('/tags');
      // console.log(response);
      if (response.data.length < 10) {
        loadMore = false;
      }
      allTags = [...response.data];
    } catch (error) {
      console.log(error);
    }
  }

  async function loadMoreTags() {
    try {
      if (loadMore == false) {
        return;
      }
      page++;
      const response = await axios.get(`/tags?page=${page}`);
      // console.log(response);

      if (response.status == 204) {
        loadMore = false;
        return;
      }
      if (response.data.length < 10) {
        loadMore = false;
      }

      allTags = [...allTags, ...response.data];
    } catch (error) {
      console.log(error);
    }
  }

  async function getUserDetails() {
    try {
      const response = await axios.get('/userDetails');
      // console.log(response);

      firstName = response.data.firstName;
      profilePictureCode = response.data.profilePictureCode;
      streak = response.data.streak;
      totalLikes = response.data.totalLikes;
      totalAnswers = response.data.totalAnswers;
    } catch (error) {
      console.log(error);
    }
  }

  async function getHomeFeed() {
    try {
      const response = await axios.get('/feed');
      // console.log(response);

      bestAnswer = response.data.bestAnswer;
      banners = [...response.data.banner];
      const tmpTags = new Set();
      response.data.topTags.forEach((tag) => {
        tmpTags.add(tag);
      });
      topTags = [...tmpTags];
      topQuestions = [...response.data.topQuestions];
    } catch (error) {
      console.log(error);
    }
  }
  async function getLeaderboard() {
    try {
      const response = await axios.get('/leaderboard');
      // console.log(response);
      leaderboard = response.data;
    } catch (error) {
      console.log(error);
    }
  }

  function greetUser() {
    let today = new Date();
    let curHr = today.getHours();
    if (curHr < 12) {
      icon = 'brightness-alt-high';
      greet = 'Good morning';
    } else if (curHr < 18) {
      icon = 'brightness-high';
      greet = 'Good afternoon';
    } else {
      icon = 'moon';
      greet = 'Good evening';
    }
  }

  onMount(async () => {
    await getUserDetails();
    await getAllTags();
    await getHomeFeed();
    greetUser();
    await getLeaderboard();

    let len = leaderboard.length;
    if (len <= 3) {
      for (let i = 0; i < len; i++) {
        smleaderboard[i] = leaderboard[i];
      }
    } else {
      for (let i = 0; i < 3; i++) {
        smleaderboard[i] = leaderboard[i];
      }
    }
  });
</script>

<svelte:head>
  <title>HomeFeed</title>
</svelte:head>

<div class="container">
  <nav class="navbar navbar-light border-bottom border-light">
    <div class="container justify-content-start">
      <a class="navbar-brand" href={null}>
        <img
          src={profileUrl + 'pic' + profilePictureCode + '.png'}
          alt=""
          height="30"
        />
        <b class="blockquote">Hey {firstName}</b>
      </a>
      <div class="text-end d-lg-none">
        <a
          href={null}
          class="btn btn-outline-secondary rounded align-center"
          data-bs-toggle="modal"
          data-bs-target="#profile"><i class="bi bi-sunglasses" /></a
        >
        <a
          href={null}
          class="btn btn-outline-secondary rounded align-center"
          data-bs-toggle="modal"
          data-bs-target="#leaderboard"><i class="bi bi-bar-chart-fill" /></a
        >
        <a
          class="btn btn-outline-danger rounded align-center"
          on:click={logout}
          href={null}><i class="bi bi-box-arrow-right" /></a
        >
      </div>

      <div class="form d-none d-lg-block">
        <button class="btn btn-outline-secondary mx-4 btn-sm"
          ><i class="bi bi-pen" />{totalAnswers} Answer</button
        >
        <button class="btn btn-outline-secondary mx-4 btn-sm"
          ><i class="bi bi-sunglasses" />{streak} Streak</button
        >
        <button class="btn btn-outline-secondary mx-4 btn-sm"
          ><i class="bi bi-heart-fill" />{totalLikes} Likes</button
        >
        <a
          on:click={logout}
          href={null}
          class="btn btn-outline-danger mx-4 btn-sm"
          ><i class="bi bi-box-arrow-right" /> Logout</a
        >
      </div>
    </div>
  </nav>

  <!-- Desktop -->
  <div
    class="d-flex justify-content-start mt-3 d-none d-lg-block"
    role="group"
    aria-label="Basic radio toggle button group"
    style="max-width: 70%"
  >
    <button
      on:click={getExploreTags}
      type="button"
      class="btn btn-outline-secondary mx-1 rounded-pill explore"
      data-bs-toggle="modal"
      data-bs-target="#explore">Explore</button
    >

    {#each allTags as tag}
      <input
        on:click={chooseTag}
        type="button"
        class="btn-check"
        id={tag.tag}
      />
      <label class="btn btn-outline-primary mx-2 my-2" for={tag.tag}
        >{tag.tag}</label
      >
    {/each}
    {#if loadMore != false}
      <button
        type="button"
        on:click={loadMoreTags}
        class="btn btn-outline-primary mx-1 rounded-pill explore">More..</button
      >
    {/if}
  </div>
  <!-- Mobile overflow-auto -->
  <div
    class="d-flex justify-content-start mt-3 d-lg-none overflow-auto tags"
    role="group"
    aria-label="Basic radio toggle button group"
  >
    <button
      on:click={getExploreTags}
      type="button"
      class="btn btn-outline-secondary mx-1 rounded-pill explore css-selector"
      data-bs-toggle="modal"
      data-bs-target="#explore">Explore</button
    >
    {#each allTags as tag}
      <input
        on:click={chooseTag}
        type="button"
        class="btn-check"
        id={tag.tag}
      />
      <label class="btn btn-outline-primary mx-2" for={tag.tag}>{tag.tag}</label
      >
    {/each}
    {#if loadMore != false}
      <button
        type="button"
        on:click={loadMoreTags}
        class="btn btn-outline-primary mx-1 rounded-pill explore">More..</button
      >
    {/if}
  </div>
</div>

<div class="container mt-3">
  <div class="row">
    <div class="col-sm-8" id="slide">
      <!-- New design start -->

      <p class="fw-normal"><i class="bi bi-{icon} fs-4" /> {greet}</p>

      <!-- carousel start here -->
      <div
        id="carouselExampleSlidesOnly"
        class="carousel slide"
        data-bs-ride="carousel"
      >
        <div class="carousel-inner">
          {#each banners as banner, i}
            {#if i == 0}
              <div class="carousel-item active">
                <a
                  class="text-decoration-none link-secondary"
                  href={banner.cta}
                >
                  <div class="card mb-3 shadow-sm round">
                    <div class="row g-0 p-2">
                      <div class="col-2 my-auto">
                        <img
                          src={banner.imageUrl}
                          class="img-fluid rounded-start"
                          width="50px"
                          alt=""
                        />
                      </div>
                      <div class="col-10">
                        <div class="card-body">
                          <p class="card-text small">
                            {banner.contentText}
                          </p>
                        </div>
                      </div>
                    </div>
                  </div>
                </a>
              </div>
            {:else}
              <div class="carousel-item">
                <a
                  class="text-decoration-none link-secondary"
                  href={banner.cta}
                >
                  <div class="card mb-3 shadow-sm round">
                    <div class="row g-0 p-2">
                      <div class="col-2 my-auto">
                        <img
                          src={banner.imageUrl}
                          class="img-fluid rounded-start"
                          width="50px"
                          alt=""
                        />
                      </div>
                      <div class="col-10">
                        <div class="card-body">
                          <p class="card-text small">
                            {banner.contentText}
                          </p>
                        </div>
                      </div>
                    </div>
                  </div>
                </a>
              </div>
            {/if}
          {/each}
        </div>
      </div>
      <!-- carousel end here -->
      <p class="fw-normal mb-0">
        <i class="bi bi-sunglasses fs-4" /> Trending categories
      </p>

      <div class="card mb-0 border-0">
        <div class="row g-0">
          <div class="col-12">
            <div class="card-body">
              {#each topTags as tag}
                <button
                  on:click={chooseTag}
                  id={tag}
                  type="button"
                  style="background: #F3F6FF"
                  class="btn btn-outline-primary mb-2 mx-1">{tag}</button
                >
              {/each}
            </div>
          </div>
        </div>
      </div>

      <p class="fw-normal">
        <i class="bi bi-hash fs-4 bold" /> Trending questions
      </p>

      {#each topQuestions as question, i}
        <div
          id={question.id}
          on:click={chooseQuestion}
          on:keydown={null}
          class="card mb-3 shadow-sm round"
        >
          <div class="row g-0 p-2">
            <div class="col-2 my-auto text-center">
              <p class="lead fw-bold fs-2" style="color:#3366FF">#{i + 1}</p>
            </div>
            <div class="col-10">
              <div class="card-body">
                <p class="card-text small">
                  {question.question}
                </p>
              </div>
            </div>
          </div>
        </div>
      {/each}

      <p class="fw-normal mt-4"><i class="bi bi-suit-heart" /> Best answer</p>

      <div
        on:click={chooseBestAnswer}
        on:keydown={null}
        class="card mb-3 shadow-sm round"
      >
        <h5 class="card-header fs-6 bg-white">
          <img
            src={profileUrl + 'pic' + bestAnswer.profilePictureCode + '.png'}
            alt=""
            height="30"
          />
          {bestAnswer.answeredBy} &ensp;
          <i style="color: #3366FF;" class="bi bi-heart-fill fs-6" />
          <small class="text-muted"> {bestAnswer.likeNumber} likes</small>
        </h5>
        <div class="card-body mt-1 pt-0">
          <small
            ><span class="badge rounded-pill text-bg-primary">Question</span
            ></small
          >
          <h5 class="card-title fs-6 border-bottom py-2">
            {bestAnswer.question}?
          </h5>
          <p class="card-text text-secondary">
            <small>
              <span class="badge rounded-pill text-bg-success mb-1">Answer</span
              >
              {@html bestAnswer.answer}
            </small>
          </p>
        </div>
      </div>

      <!-- Bottom Image start -->
      <div class="container d-lg-none mb-2">
        <img
          src="/assets/images/explore.png"
          class="img-fluid"
          alt="noanswer"
        />
      </div>
      <!-- Bottom Image End -->

      <!-- New design end -->
    </div>
    <div
      class="col-sm-4 fixed-top d-none d-lg-block"
      style="margin-left: 65%; margin-top: 2%;"
    >
      <div
        class="card rounded"
        style="background-color: #F3F6FF; border: none;"
      >
        <div class="card-body">
          <h5 class="card-title">What is eli5?</h5>
          <p class="card-text">
            ELI5 is short for “Explain Like I’m 5,” a request for a simple
            explanation to an otherwise complicated question or concept.
          </p>
          <b class="card-text">How can I answer question?</b>
          <p class="card-text">Just hit EIL5 button follow the basic rule:</p>
          <ul>
            <li>State it: State the idea clearly, in a few sentences.</li>
            <li>
              Elaborate: Explain the idea further in your own words. Elaborate
              on the concept in a paragraph or less.
            </li>
            <li>
              Exemplify: Provide concrete examples (and counter-examples, if
              necessary).
            </li>
            <li>Summarise: Summarise your explanation.</li>
          </ul>

          <i class="bi bi-info-circle" />
          <a class="link-dark" href="/rules" use:link>Rules</a>
        </div>
      </div>

      <div class="card" style="border:none">
        <div class="card-body">
          <h5 class="card-title">Leaderboard</h5>
          <table class="table" style="border: solid; border-color: #E6E8F0;">
            <thead style="background-color:#FAFBFF;">
              <tr>
                <th scope="col">Rank</th>
                <th scope="col">User</th>
                <th scope="col">Likes</th>
                <th scope="col">Answers</th>
              </tr>
            </thead>
            <tbody>
              {#each smleaderboard as lead, i}
                <tr>
                  <td> <b>{i + 1}</b></td>
                  <td
                    ><img
                      src={profileUrl +
                        'pic' +
                        lead.profilePictureCode +
                        '.png'}
                      alt=""
                      height="30"
                    /> <b>{lead.uniqueAlias}</b></td
                  >
                  <td>{lead.totalLikes}</td>
                  <td>{lead.totalAnswers}</td>
                </tr>
              {/each}
              <tr>
                <th
                  ><a
                    href={null}
                    class="small"
                    data-bs-toggle="modal"
                    data-bs-target="#leaderboard">More</a
                  ></th
                >
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- Profile Modal -->
<div
  class="modal fade"
  id="profile"
  tabindex="-1"
  aria-labelledby="exampleModalLabel"
  aria-hidden="true"
>
  <div class="modal-dialog modal-fullscreen-sm-down">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel">Profile Detials</h5>
        <button
          type="button"
          class="btn-close"
          data-bs-dismiss="modal"
          aria-label="Close"
        />
      </div>
      <div class="modal-body">
        <div class="container">
          <div class="alert alert-success" role="alert">
            <i class="bi bi-pen" />
            {totalAnswers} Answer
          </div>

          <div class="alert alert-success" role="alert">
            <i class="bi bi-sunglasses" />
            {streak} Streak
          </div>

          <div class="alert alert-success" role="alert">
            <i class="bi bi-heart-fill" />
            {totalLikes} Likes
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<!-- Leaderboard Modal -->
<div
  class="modal fade"
  id="leaderboard"
  tabindex="-1"
  aria-labelledby="exampleModalLabel"
  aria-hidden="true"
>
  <div class="modal-dialog modal-fullscreen-sm-down">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel">Leaderboard</h5>
        <button
          type="button"
          class="btn-close"
          data-bs-dismiss="modal"
          aria-label="Close"
        />
      </div>
      <div class="modal-body p-0">
        <table class="table p-0 m-0" style=" font-size: small;">
          <thead style="background-color:#FAFBFF;">
            <tr>
              <th scope="col">#</th>
              <th scope="col">User</th>
              <th scope="col">Likes</th>
              <th scope="col">Answers</th>
            </tr>
          </thead>
          <tbody>
            {#each leaderboard as lead, i}
              <tr>
                <td><span>{i + 1}</span></td>
                <td
                  ><img
                    src={profileUrl + 'pic' + lead.profilePictureCode + '.png'}
                    alt=""
                    height="30"
                  /> <span>{lead.uniqueAlias}</span></td
                >
                <td>{lead.totalLikes}</td>
                <td>{lead.totalAnswers}</td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
      <div class="modal-footer" />
    </div>
  </div>
</div>

<div
  class="modal fade"
  id="explore"
  tabindex="-1"
  aria-labelledby="exampleModalLabel"
  aria-hidden="true"
>
  <div class="modal-dialog modal-fullscreen-sm-down">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel">
          <i class="bi bi-binoculars" /> Explore on Eli5
        </h5>
        <button
          type="button"
          class="btn-close"
          data-bs-dismiss="modal"
          aria-label="Close"
        />
      </div>

      <div class="modal-body">
        <div class="container">
          <div class="row text-center">
            {#each explore as tag}
              <div
                id={tag.tag}
                on:click={chooseExploreTag}
                on:keypress={null}
                data-bs-dismiss="modal"
                class="col m-1 p-2 border border-1 rounded shadow-sm"
              >
                {tag.tag}
              </div>
            {/each}
          </div>
        </div>
      </div>

      <div class="card-footer">
        <div class="container mb-2">
          <img
            src="/assets/images/explore.png"
            class="img-fluid"
            alt="noanswer"
          />
        </div>
      </div>
    </div>
  </div>
</div>

<style>
  .tags {
    -ms-overflow-style: none; /* Internet Explorer 10+ */
    scrollbar-width: none; /* Firefox */
  }
  .tags::-webkit-scrollbar {
    display: none; /* Safari and Chrome */
  }

  .explore {
    background: linear-gradient(-45deg, #3366ff, #ffffff, #3366ff, #3366ff);
    background-size: 400% 400%;
    animation: gradient 2s ease infinite;
    animation-iteration-count: 10;
    color: white;
    border-color: #3366ff;
  }

  @keyframes gradient {
    0% {
      background-position: 0% 50%;
    }
    50% {
      background-position: 100% 50%;
    }
    100% {
      background-position: 0% 50%;
    }
  }
</style>
