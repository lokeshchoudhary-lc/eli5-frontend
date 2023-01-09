<script>
  import axios from 'axios';
  import { onMount } from 'svelte';
  import { profanity } from '@2toad/profanity';
  import { link, replace } from 'svelte-spa-router';
  import {
    textEditorHtml,
    userChoosenTagState,
    userChoosenQuestionState,
    noAnswerContent,
  } from './store';

  import Answers from './components/Answers.svelte';
  import TipTapEditor from './components/TipTapEditor.svelte';

  let textAreaAnswer = '';
  let textAreaAskQuestion = '';
  let boolAnswered = false;

  let profileUrl = '/assets/images/profile/';

  let profilePictureCode = 1;
  let firstName = '';
  let totalLikes = '';
  let totalAnswers = '';
  let streak = '';

  let userAnswer = {};

  let userChoosenTag;
  let userChoosenQuestion;

  let noAnswer;

  noAnswerContent.subscribe((value) => {
    noAnswer = value;
  });

  userChoosenTagState.subscribe((value) => {
    userChoosenTag = value;
    if (value == null) {
      userChoosenTag = localStorage.getItem('userChoosenExploreTag');
    }
  });

  userChoosenQuestionState.subscribe((value) => {
    userChoosenQuestion = value;
  });

  let selectedQuestion;
  let selectedQuestionId;

  let nextPage = 1;
  let backPage = 1;
  let sortType;
  let reRender = false;
  let explore = [];

  let leaderboard = [];

  let smleaderboard = [];

  let questionFeed = new Map();
  let counter = 0;

  async function getUserAnswer(questionId) {
    try {
      const response = await axios.get(`/userAnswer/${questionId}`);
      // console.log(response);
      userAnswer = response.data;
      if (response.data.message != 'no_answer') {
        boolAnswered = true;
      } else {
        boolAnswered = false;
      }
    } catch (error) {
      console.log(error);
    }
  }

  async function submitAnswer(event) {
    textEditorHtml.subscribe((value) => {
      textAreaAnswer = value.getHTML();
    });

    if (profanity.exists(textAreaAnswer)) {
      alert('Usage of Bad Words Found');
      return;
    }
    let id = event.currentTarget.id;

    await axios
      .post(`/answer/${id}`, {
        answer: textAreaAnswer,
        questionId: id,
      })
      .then(function (response) {
        // console.log(response);
        textAreaAnswer = '';
        boolAnswered = true;
        userAnswer = response.data.userAnswer;
        streak = response.data.streak;
        reRender = true;
      })
      .catch(function (error) {
        console.log(error);
        if (error.response.status == 400) {
          boolAnswered = true;
        }
      });
  }

  function chooseSort(event) {
    sortType = event.currentTarget.value;
  }

  function eraseCookie(name) {
    document.cookie =
      name + '=; Path=/; Expires=Thu, 01 Jan 1970 00:00:01 GMT;';
  }

  function logout() {
    axios
      .get('/logout')
      .then(function (response) {
        eraseCookie('loginState');
        replace('/');
        // console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
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
  function chooseExploreTag(event) {
    let tag = event.currentTarget.id;
    userChoosenTagState.set(tag);
    localStorage.setItem('userChoosenExploreTag', tag);
    localStorage.setItem('userChoosenTag', tag);
    replace('/exploreQuestions');
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

  async function getLeaderboard() {
    try {
      const response = await axios.get('/leaderboard');
      // console.log(response);
      leaderboard = response.data;
    } catch (error) {
      console.log(error);
    }
  }
  async function nextQuestion() {
    try {
      let tmpCounter = counter;
      tmpCounter++;
      if (questionFeed.has(tmpCounter)) {
        if (questionFeed.get(tmpCounter) == 'no_content') {
          return;
        }
        counter = tmpCounter;
        let result = questionFeed.get(counter);
        selectedQuestion = result.question;
        selectedQuestionId = result.id;
        await getUserAnswer(selectedQuestionId);
      } else {
        const response = await axios.get(
          `/question/${selectedQuestionId}?tag=${userChoosenTag}&page=${nextPage}&action=next`
        );
        // console.log(response);
        if (response.status == 204) {
          questionFeed.set(tmpCounter, 'no_content');
          return;
        }
        nextPage++;
        response.data.forEach((element) => {
          counter++;
          questionFeed.set(counter, element);
        });
        counter = tmpCounter;
        let result = questionFeed.get(counter);
        selectedQuestion = result.question;
        selectedQuestionId = result.id;
        await getUserAnswer(selectedQuestionId);
      }
    } catch (error) {
      console.log(error);
    }
  }
  async function previousQuestion() {
    try {
      let tmpCounter = counter;
      tmpCounter--;
      if (questionFeed.has(tmpCounter)) {
        if (questionFeed.get(tmpCounter) == 'no_content') {
          return;
        }
        counter = tmpCounter;
        let result = questionFeed.get(counter);
        selectedQuestion = result.question;
        selectedQuestionId = result.id;
        await getUserAnswer(selectedQuestionId);
      } else {
        const response = await axios.get(
          `/question/${selectedQuestionId}?tag=${userChoosenTag}&page=${backPage}&action=back`
        );
        // console.log(response);
        if (response.status == 204) {
          questionFeed.set(tmpCounter, 'no_content');
          return;
        }
        backPage++;
        response.data.forEach((element) => {
          counter--;
          questionFeed.set(counter, element);
        });
        counter = tmpCounter;
        let result = questionFeed.get(counter);
        selectedQuestion = result.question;
        selectedQuestionId = result.id;
        await getUserAnswer(selectedQuestionId);
      }
    } catch (error) {
      console.log(error);
    }
  }
  async function askQuestion() {
    try {
      if (textAreaAskQuestion == '') {
        return;
      }
      await axios.post('/question/ask', {
        question: textAreaAskQuestion,
      });
      textAreaAskQuestion = '';
    } catch (error) {
      console.log(error);
    }
  }

  onMount(async () => {
    if (userChoosenQuestion == null) {
      userChoosenTag = localStorage.getItem('userChoosenTag');
      const response = await axios.get(`/singleQuestion/${userChoosenTag}`);
      selectedQuestion = response.data.question;
      selectedQuestionId = response.data.id;
      questionFeed.set(0, response.data);
    } else {
      if (
        userChoosenQuestion.question != null &&
        userChoosenQuestion.id != null
      ) {
        localStorage.setItem(
          'userChoosenQuestion',
          userChoosenQuestion.question
        );
        localStorage.setItem('userChoosenQuestionId', userChoosenQuestion.id);
      }

      selectedQuestion = localStorage.getItem('userChoosenQuestion');
      selectedQuestionId = localStorage.getItem('userChoosenQuestionId');
      questionFeed.set(0, userChoosenQuestion);
    }

    if (localStorage.getItem('userChoosenQuestion') != null) {
      selectedQuestion = localStorage.getItem('userChoosenQuestion');
      selectedQuestionId = localStorage.getItem('userChoosenQuestionId');
      questionFeed.set(0, userChoosenQuestion);
    }

    sortType = 'trending';

    await getUserDetails();
    await getLeaderboard();

    await getUserAnswer(selectedQuestionId);
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
  <title>Feed</title>
</svelte:head>

<!-- Mobile Bootom NavBar -->
<nav
  class="navbar navbar-light bg-light fixed-bottom d-lg-none px-4 py-2 shadow-lg border-top rounded"
>
  <a on:click={previousQuestion} href={null} class="btn btn-primary border-0"
    ><i class="bi bi-arrow-left fs-4" /></a
  >
  <a
    href={null}
    class="btn btn-outline-primary border-0"
    data-bs-toggle="modal"
    data-bs-target="#suggest"
    ><i class="bi bi-plus-circle fs-2" style="color: #0d6efd" /></a
  >
  <a on:click={nextQuestion} href={null} class="btn btn-primary border-0"
    ><i class="bi bi-arrow-right fs-4" /></a
  >
</nav>

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
        <button
          class="btn btn-outline-secondary mx-4 btn-sm"
          data-bs-toggle="modal"
          data-bs-target="#suggest"
          ><i class="bi bi-plus" /> Ask Question</button
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
    class="container mt-3 d-none d-lg-block overflow-auto"
    role="group"
    aria-label="Basic radio toggle button group"
  >
    <button
      on:click={getExploreTags}
      type="button"
      class="btn btn-outline-secondary mx-1 rounded-pill explore"
      data-bs-toggle="modal"
      data-bs-target="#explore">Explore</button
    >
    <label class="btn btn-outline-primary mx-1" for={userChoosenTag}
      >{userChoosenTag}</label
    >
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
      class="btn btn-outline-secondary mx-1 rounded-pill explore"
      data-bs-toggle="modal"
      data-bs-target="#explore">Explore</button
    >
    <label class="btn btn-outline-primary mx-2" for={userChoosenTag}
      >{userChoosenTag}</label
    >
  </div>
</div>

<div class="container mt-3">
  <div class="row mb-5">
    <div class="col-sm-8 mb-4">
      <!-- Deskptop -->
      <div
        class="container p-2 shadow-sm rounded d-none d-lg-block"
        style="background-color: #F3F6FF; border-style: solid; border-color: #3366FF;"
      >
        <div class="row align-items-center">
          <div class="col-1 text-center fs-4">
            <a on:click={previousQuestion} class="btn btn-primary" href={null}
              ><i class="bi bi-arrow-left" /></a
            >
          </div>
          <div class="col-7">
            <p class="h5">{selectedQuestion} ?</p>
            <span class="badge text-bg-primary">Explain like I'm five</span>
          </div>
          <div class="col-3 text-end">
            <button
              type="button"
              class={boolAnswered ? 'btn btn-disable' : 'btn btn-primary'}
              data-bs-toggle="modal"
              data-bs-target={boolAnswered ? '' : '#writeelif'}
              ><i class="bi bi-pen" /> Eli5</button
            >
          </div>
          <div class="col-1 text-center fs-4">
            <a on:click={nextQuestion} class="btn btn-primary" href={null}
              ><i class="bi bi-arrow-right" /></a
            >
          </div>
        </div>
      </div>
      <!-- Mobile -->
      <div
        class="container p-2 shadow-sm rounded d-lg-none"
        style="background-color: #F3F6FF; border-style: solid; border-color: #3366FF;"
      >
        <div class="row align-items-center">
          <div class="col-10">
            <b>{selectedQuestion} ?</b>
            <span class="badge text-bg-primary">Explain like I'm five</span>
          </div>
          <div class="col-2 text-end">
            <button
              type="button"
              class={boolAnswered ? 'btn btn-disable' : 'btn btn-primary'}
              data-bs-toggle="modal"
              data-bs-target={boolAnswered ? '' : '#writeelif'}
              ><i class="bi bi-pen" />
            </button>
          </div>
        </div>
      </div>
      <!-- user answer for the above question -->
      {#if userAnswer.answer !== undefined}
        <div class="card border-success mt-2 shadow-sm rounded">
          <div class="card-header bg-white border-light">
            <img
              src={profileUrl + 'pic' + profilePictureCode + '.png'}
              alt=""
              height="30"
            />
            <b class="small">{userAnswer.answeredBy}</b> &emsp;
            <span class="badge rounded-pill bg-success">Your Answer</span>
          </div>
          <div class="card-body text-secondary my-0 py-0">
            <p class="card-text m-0 py-0 small text-secondary">
              {@html userAnswer.answer}
            </p>
          </div>
          <div class="card-body text-secondary my-1 py-0">
            <input
              type="button"
              class="btn-check"
              id={userAnswer.id}
              autocomplete="off"
            />
            <small class="text-muted">{userAnswer.likeNumber} likes</small> &ensp;
          </div>
        </div>
      {/if}
      <!-- end here user answer  -->
      {#if noAnswer != true}
        <div class="container mt-3">
          <div class="row align-items-center">
            <div class="col-2 d-none d-lg-block">Sort by</div>
            <div class="col-10">
              <div
                class="btn-group"
                role="group"
                aria-label="Basic radio toggle button group"
              >
                <input
                  type="radio"
                  on:change={chooseSort}
                  class="btn-check"
                  name="radio"
                  id="radio1"
                  value="trending"
                  autocomplete="off"
                  checked
                />
                <label class="btn btn-outline-primary" for="radio1"
                  >Trending</label
                >

                <input
                  type="radio"
                  on:change={chooseSort}
                  class="btn-check"
                  name="radio"
                  id="radio3"
                  value="lastest"
                  autocomplete="off"
                />
                <label class="btn btn-outline-primary" for="radio3"
                  >Latest</label
                >
              </div>
            </div>
          </div>
        </div>
      {/if}

      {#if noAnswer == true}
        <div class="containe pt-2">
          <img
            src="/assets/images/noanswer.png"
            class="img-fluid"
            alt="noanswer"
          />
        </div>
      {/if}

      {#key [sortType, selectedQuestionId, reRender]}
        <Answers {sortType} {selectedQuestionId} />
      {/key}
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
          <div class="alert alert-primary" role="alert">
            <i class="bi bi-pen" />
            {totalAnswers} Answer
          </div>

          <div class="alert alert-primary" role="alert">
            <i class="bi bi-sunglasses" />
            {streak} Streak
          </div>

          <div class="alert alert-primary" role="alert">
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
        <table class="table p-0 m-0" style="font-size: small;">
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
                    height="20"
                  /> <span class="small">{lead.uniqueAlias}</span></td
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

<!-- Suggest question -->

<div
  class="modal fade"
  id="suggest"
  tabindex="-1"
  aria-labelledby="exampleModalLabel"
  aria-hidden="true"
>
  <div class="modal-dialog modal-fullscreen-sm-down">
    <div class="modal-content">
      <div class="modal-header">
        <h1 class="modal-title fs-5" id="exampleModalLabel">
          Share your thoughts
        </h1>
        <button
          type="button"
          class="btn-close"
          data-bs-dismiss="modal"
          aria-label="Close"
        />
      </div>
      <div class="modal-body">
        <form>
          <div class="mb-3">
            <label for="recipient-name" class="col-form-label"
              >What do you want to learn as five year old ?</label
            >
            <div class="mb-3 mt-1">
              <label for="message-text" class="col-form-label">Question:</label>
              <textarea
                class="form-control"
                id="message-text"
                bind:value={textAreaAskQuestion}
              />
            </div>
          </div>
        </form>
      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal"
          >Close</button
        >
        <button
          on:click={askQuestion}
          type="button"
          class="btn btn-primary"
          data-bs-dismiss="modal">Send message</button
        >
      </div>
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

<!-- Write Elif Modal -->
<div
  class="modal fade"
  id="writeelif"
  tabindex="-1"
  aria-labelledby="writeeliflabel"
  aria-hidden="true"
>
  <div class="modal-dialog modal-fullscreen-sm-down">
    <div class="modal-content">
      <div class="modal-header">
        <img
          src={profileUrl + 'pic' + profilePictureCode + '.png'}
          alt=""
          height="30"
        />
        <h6 class="modal-title mx-1 ms-2" id="exampleModalLabel">
          {selectedQuestion} ?
        </h6>
        <button
          type="button"
          class="btn-close"
          data-bs-dismiss="modal"
          aria-label="Close"
        />
      </div>
      <div class="modal-body">
        <!-- <div class="form-floating">
          <textarea
            class="form-control"
            placeholder="Leave a comment here"
            id="floatingTextarea2"
            bind:value={textAreaAnswer}
            style="height: 200px"
            maxlength="2000"
          />
          
          <label for="floatingTextarea2">Explain like I’m five</label>
        </div> -->

        <TipTapEditor />

        <div class="modal-footer">
          <button
            type="button"
            class="btn btn-primary"
            data-bs-dismiss="modal"
            id={selectedQuestionId}
            on:click={submitAnswer}>Eli5 it</button
          >
        </div>
        <div
          class="alert alert-primary alert-dismissible fade show mt-3"
          role="alert"
          style=" border-style: solid; border-color: #3366FF;"
        >
          <strong>Approach to write the Answer</strong>

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

          <button
            type="button"
            class="btn-close"
            data-bs-dismiss="alert"
            aria-label="Close"
          />
        </div>
      </div>
    </div>
  </div>
</div>

<!-- Toast messages -->

<!-- <div class="toast-container position-fixed top-0 end-0 p-3">
  <div
    id="liveToast"
    class="toast"
    role="alert"
    aria-live="assertive"
    aria-atomic="true"
  >
    <div class="toast-header">
      <strong class="me-auto"
        ><i class="bi bi-clipboard-check" /> Link Copied</strong
      >
      <small />
      <button
        type="button"
        class="btn-close"
        data-bs-dismiss="toast"
        aria-label="Close"
      />
    </div>
    <div class="toast-body">https://www.eli5.club successfully copied!</div>
  </div>
</div> -->
<style>
  .tags {
    -ms-overflow-style: none; /* Internet Explorer 10+ */
    scrollbar-width: none; /* Firefox */
  }
  .tags::-webkit-scrollbar {
    display: none; /* Safari and Chrome */
  }
  /* .toast-container {
    z-index: 10000;
  }
  .toast {
    color: #0f5132;
    background: #d1e7dd;
  }
  .toast-header {
    color: #0f5132;
    background: #ffffff;
  }
  .toastclose {
    color: #ffffff;
  } */
</style>
