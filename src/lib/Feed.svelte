<script>
  import axios from 'axios';
  import { onMount } from 'svelte';
  import { profanity } from '@2toad/profanity';
  import { replace } from 'svelte-spa-router';

  import Answers from './components/Answers.svelte';

  let textAreaAnswer = '';
  let boolAnswered = false;

  let profileUrl = '/assets/images/profile/';

  let profilePictureCode = 1;
  let firstName = '';
  let totalLikes = '';
  let totalAnswers = '';
  let streak = '';

  //////////////////////////////////////
  let userAnswer = {};
  // let userAnswer = {
  //   uniqueAlias: 'rider_ritik',
  //   answeredBy: '33433',
  //   answer: `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
  //       tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
  //       veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
  //       commodo consequat. Duis aute irure dolor in reprehenderit in voluptate
  //       velit esse cillum dolore eu fugia`,
  //   id: '33',
  //   likeNumber: 100,
  //   liked: true,
  // };
  ////////////////////////////////////////////

  let selectedQuestion;
  let selectedQuestionId;

  let sortType;
  let feed = [];
  let reRender = false;

  // let feed = [
  //   {
  //     tag: 'Computer',
  //     question: {
  //       id: '1',
  //       question: 'question 1',
  //     },
  //   },
  //   {
  //     tag: 'Golang',
  //     question: {
  //       id: '2',
  //       question: 'question 2',
  //     },
  //   },
  // ];

  // let leaderboard = [
  //   {
  //     uniqueAlias: 'rider_ritik',
  //     totalLikes: 100,
  //     totalAnswers: 1000,
  //     profilePictureCode: 1,
  //   },
  //   {
  //     uniqueAlias: 'rider_ritik',
  //     totalLikes: 100,
  //     totalAnswers: 1000,
  //     profilePictureCode: 16,
  //   },
  //   {
  //     uniqueAlias: 'rider_ritik',
  //     totalLikes: 100,
  //     totalAnswers: 1000,
  //     profilePictureCode: 11,
  //   },
  //   {
  //     uniqueAlias: 'rider_ritik',
  //     totalLikes: 100,
  //     totalAnswers: 1000,
  //     profilePictureCode: 5,
  //   },
  // ];
  let leaderboard = [];

  let smleaderboard = [];

  async function getUserAnswer(questionId) {
    try {
      const response = await axios.get(`/userAnswer/${questionId}`);
      console.log(response);
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

  async function chooseTag(event) {
    selectedQuestionId = event.currentTarget.id;
    selectedQuestion = event.currentTarget.value;
    await getUserAnswer(selectedQuestionId);

  }

  async function submitAnswer(event) {
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
        console.log(response);
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

  function logout() {
    axios
      .get('/logout')
      .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });

    replace('/');
  }

  function scrollOnTop() {
    document.body.scrollIntoView();
  }

  async function getFeed() {
    try {
      const response = await axios.get('/feed');
      console.log(response);

      firstName = response.data.firstName;
      profilePictureCode = response.data.profilePictureCode;
      streak = response.data.streak;
      totalLikes = response.data.totalLikes;
      totalAnswers = response.data.totalAnswers;
      feed = [...response.data.feed];
    } catch (error) {
      console.log(error);
    }
  }
  async function getLeaderboard() {
    try {
      const response = await axios.get('/leaderboard');
      console.log(response);
      leaderboard = response.data;
    } catch (error) {
      console.log(error);
    }
  }

  // async function next() {
  //   document.getElementById("slide").style.translate='-100px';

  //   let id = null;
  // const elem = document.getElementById("slide");   
  // let pos = 0;
  // clearInterval(id);
  // id = setInterval(frame, 5);
  // function frame() {
  //   if (pos == 350) {
  //     clearInterval(id);
  //   } else {
  //     pos++; 
  //     elem.style.top = pos + "px"; 
  //     elem.style.left = pos + "px"; 
  //   }
  // }
  // }

  // async function previous() {
  //   alert("previous");
  // }

  onMount(async () => {
    await getFeed();
    await getLeaderboard();

    sortType = 'trending';

    selectedQuestion = feed[0].question.question;
    selectedQuestionId = feed[0].question.id;

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


<!-- Mobile Bootom NavBar -->
<nav
  class="navbar navbar-light bg-light fixed-bottom d-lg-none px-4 py-2 shadow-lg border-top rounded"
>
<a class="btn btn-primary border-0" href={null}><i class="bi bi-arrow-left fs-4"></i></a>
<a class="btn btn-outline-primary border-0" data-bs-toggle="modal" data-bs-target="#suggest" data-bs-whatever="@amazing_ritik"><i class="bi bi-plus-circle fs-2" style="color: #0d6efd"></i></a>
<button class="btn btn-primary border-0" href={null} id="next"><i class="bi bi-arrow-right fs-4"></i></button>
</nav>

<!-- floating pen write button will see about it 
<div
  class="fixed-bottom d-lg-none"
  style="margin-bottom: 4.4rem; margin-left: 83%;"
>
  <a
    class="btn btn-primary"
    data-bs-toggle="modal"
    data-bs-target="#writeelif"
    style="border-radius:100%"><i class="bi bi-pen fs-4 text-light" /></a
  >
</div> -->

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
        <a class="btn btn-outline-secondary rounded align-center" data-bs-toggle="modal" data-bs-target="#profile"><i class="bi bi-sunglasses"></i></a>
        <a class="btn btn-outline-secondary rounded align-center" data-bs-toggle="modal" data-bs-target="#leaderboard"><i class="bi bi-bar-chart-fill"></i></a>
        <a class="btn btn-outline-danger rounded align-center" on:click={logout} href={null}><i class="bi bi-box-arrow-right"></i></a>
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
        <!-- logout logic needed -->
        <a
          on:click={logout}
          href={null}
          class="btn btn-outline-secondary mx-4 btn-sm"
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

  <button type="button" class="btn btn-outline-secondary mx-1 rounded-pill explore" data-bs-toggle="modal" data-bs-target="#explore">Explore</button>

    {#each feed as feeditem}
      <input
        on:click={chooseTag}
        type="button"
        class="btn-check"
        id={feeditem.question.id}
        value={feeditem.question.question}
      />
      <label class="btn btn-outline-primary mx-1" for={feeditem.question.id}
        >{feeditem.tag}</label
      >
    {/each}

    <button type="button" class="btn btn-outline-primary mx-1 rounded-pill explore">More..</button>
  </div>
  <!-- Mobile overflow-auto -->
  <div
    class="d-flex justify-content-start mt-3 d-lg-none overflow-auto tags"
    role="group"
    aria-label="Basic radio toggle button group"
  > 

  <button type="button" class="btn btn-outline-secondary mx-1 rounded-pill explore" data-bs-toggle="modal" data-bs-target="#explore">Explore</button>

    {#each feed as feeditem}
      <input
        on:click={chooseTag}
        type="button"
        class="btn-check"
        id={feeditem.question.id}
        value={feeditem.question.question}
      />
      <label class="btn btn-outline-primary mx-1" for={feeditem.question.id}
        >{feeditem.tag}</label
      >
    {/each}

    <button type="button" class="btn btn-outline-primary mx-1 rounded-pill explore" >More..</button>
  </div>
</div>

<div class="container mt-3">
  <div class="row mb-5">
    <div class="col-sm-8" id="slide">
      <!-- Deskptop -->
        <div
          class="container p-4 shadow-sm rounded d-none d-lg-block"
          style="background-color: #F3F6FF; border-style: solid; border-color: #3366FF;"
        >
              <div class="row align-items-center">
                <div class="col-1 text-center fs-4"><a class="btn btn-primary" href={null}><i class="bi bi-arrow-left"></i></a></div>
                  <div class="col">
                    <p class="h5">{selectedQuestion} ?</p> 
                    <span class="badge text-bg-primary">Explain like I'm five</span>
                  </div>
                  <div class="col text-end">
                    <button
                      type="button"
                      class={boolAnswered ? 'btn btn-disable' : 'btn btn-primary'}
                      data-bs-toggle="modal"
                      data-bs-target={boolAnswered ? '' : '#writeelif'}
                      ><i class="bi bi-pen" /> Eli5</button
                    >
                  </div>
                  <div class="col-1 text-center fs-4"><a class="btn btn-primary" href={null}><i class="bi bi-arrow-right"></i></a></div>
              </div>

        </div>
      <!-- Mobile -->
        <div
          class="container p-4 shadow-sm rounded d-lg-none"
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
                ><i class="bi bi-pen" /> </button
              >
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
            <b>{userAnswer.answeredBy}</b> &emsp; <span class="badge rounded-pill bg-success">Your Answer</span>
          </div>
          <div class="card-body text-secondary">
            <p class="card-text">
              {userAnswer.answer}
            </p>
          </div>
          <div class="card-body text-secondary">
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
              <label class="btn btn-outline-primary" for="radio3">Latest</label>
            </div>
          </div>
        </div>
      </div>

      {#key [sortType, selectedQuestionId, reRender]}
        <Answers {sortType} {selectedQuestionId} />
      {/key}

      <div class="container text-center mt-3 mb-4">
        <button type="button" class="btn btn-outline-primary mx-2 rounded-pill" ><i class="bi bi-plus"></i>See More</button>
     </div>

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

          <a
          data-bs-toggle="modal"
          data-bs-target="#video"
          href={null}
            class="btn btn-outline-secondary">Watch how it works</a
          >
           <i class="bi bi-info-circle" /> <a class="link-dark" href="google.com">Rules</a>
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


<!-- Button trigger modal -->

<!-- Video Modal -->
<div
  class="modal fade"
  id="video"
  tabindex="-1"
  aria-labelledby="exampleModalLabel"
  aria-hidden="true"
>
  <div class="modal-dialog modal-fullscreen-sm-down">
    <div class="modal-content">
      <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel">How it works</h5>
        <button
          type="button"
          class="btn-close"
          data-bs-dismiss="modal"
          aria-label="Close"
        />
      </div>
      <div class="modal-body">
        <div class="embed-responsive embed-responsive">
          <iframe src="https://www.youtube.com/embed/2DkkdewrcX4?controls=0" title="" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
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
      <div class="modal-body">
        <table
          class="table"
          style="border: solid; border-color: #E6E8F0; font-size: small;"
        >
          <thead style="background-color:#FAFBFF;">
            <tr>
              <th scope="col">Rank</th>
              <th scope="col">User</th>
              <th scope="col">Likes</th>
              <th scope="col">Answers</th>
            </tr>
          </thead>
          <tbody>
            {#each leaderboard as lead, i}
              <tr>
                <td><b>{i + 1}</b></td>
                <td
                  ><img
                    src={profileUrl + 'pic' + lead.profilePictureCode + '.png'}
                    alt=""
                    height="30"
                  /> <b>{lead.uniqueAlias}</b></td
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

<div class="modal fade" id="suggest" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
  <div class="modal-dialog modal-fullscreen-sm-down">
      <div class="modal-content">
      <div class="modal-header">
          <h1 class="modal-title fs-5" id="exampleModalLabel">Share your thoughts</h1>
          <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
      </div>
      <div class="modal-body">
          <form>
          <div class="mb-3">
              <label for="recipient-name" class="col-form-label">What do you want to learn as five year old</label>
              <input type="text" class="form-control" id="recipient-name" disabled placeholder="Posting as: {firstName}">
          </div>
          <div class="mb-3">
          <label for="message-text" class="col-form-label">Question:</label>
              <textarea class="form-control" id="message-text"></textarea>
          </div>
          </form>
      </div>
      <div class="modal-footer">
          <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
          <button type="button" class="btn btn-primary">Send message</button>
      </div>
      </div>
  </div>
  </div>

  <div class="modal fade" id="explore" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
    <div class="modal-dialog modal-fullscreen-sm-down">
    <div class="modal-content">
        <div class="modal-header">
        <h5 class="modal-title" id="exampleModalLabel"><i class="bi bi-binoculars"></i>  Explore on Eli5</h5>
        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
        </div>
        <div class="modal-body">
            <div class="container">
                <div class="row text-center">
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Science</div>
                    <div class="col m-1 p-2 border border-1rounded shadow-sm">JEE</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">UPSC</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Life</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Health</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Sports</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Politics</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Technology</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Trending</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Economy</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Startup</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">VC&Funding</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">News</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Music</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Gaming</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">F1</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Football</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Comedy</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Fifa</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Bollywood</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Wedding</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Winters</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Delhi</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Mumbai</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Kolkata</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Cricket</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Christman</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">NewYear</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Business</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Finance</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Market</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Crypto</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Blockchain</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Biotechnology</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">Engineering</div>
                    <div class="col m-1 p-2 border border-1 rounded shadow-sm">MBBS</div>
                  </div>
                
            </div>

            <div class="container px-4 py-5 d-lg-none" id="custom-cards">
                <h2 class="pb-2 border-bottom">Trending on Eli5</h2>
            
                <div class="row row-cols-1 row-cols-lg-3 align-items-stretch g-4 py-5">
                  <div class="col">
                    <div class="card card-cover h-100 overflow-hidden text-bg-dark rounded-4 shadow-lg" style="background-image: linear-gradient(to right, #000046 , #1CB5E0);;">
                      <div class="d-flex flex-column h-100 p-5 pb-3 text-white text-shadow-1">
                        <h3 class="pt-2 mt-2 mb-2 display-6 lh-1 fw-bold">Can there be life on mars?</h3>
                        <ul class="d-flex list-unstyled mt-auto">
                          <li class="d-flex align-items-center me-3">
                            <i class="bi bi-heart-fill"></i>
                            <small>2K Likes</small>
                          </li>
                        </ul>
                      </div>
                    </div>
                  </div>

                  <div class="col">
                    <div class="card card-cover h-100 overflow-hidden text-bg-dark rounded-4 shadow-lg" style="background-image: linear-gradient(to right, #36D1DC , #5B86E5);;">
                      <div class="d-flex flex-column h-100 p-5 pb-3 text-white text-shadow-1">
                        <h3 class="pt-2 mt-2 mb-2 display-6 lh-1 fw-bold">Why basics of computer software works with binary number?</h3>
                        <ul class="d-flex list-unstyled mt-auto">
                          <li class="d-flex align-items-center me-3">
                            <i class="bi bi-heart-fill"></i>
                            <small>2K Likes</small>
                          </li>
                        </ul>
                      </div>
                    </div>
                  </div>

                  <div class="col">
                    <div class="card card-cover h-100 overflow-hidden text-bg-dark rounded-4 shadow-lg" style="background-image: linear-gradient(to right, #11998e , #38ef7d);;">
                      <div class="d-flex flex-column h-100 p-5 pb-3 text-white text-shadow-1">
                        <h3 class="pt-2 mt-2 mb-2 display-6 lh-1 fw-bold">Why everyone is creating reels on instagram?</h3>
                        <ul class="d-flex list-unstyled mt-auto">
                          <li class="d-flex align-items-center me-3">
                            <i class="bi bi-heart-fill"></i>
                            <small>2K Likes</small>
                          </li>
                        </ul>
                      </div>
                    </div>
                  </div>
            
                  


                  </div>
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
        <h6 class="modal-title" id="exampleModalLabel">
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
        <div class="form-floating">
          <textarea
            class="form-control"
            placeholder="Leave a comment here"
            id="floatingTextarea2"
            bind:value={textAreaAnswer}
            style="height: 200px"
            maxlength="2000"
          />
          <label for="floatingTextarea2">Explain like I’m five</label>
        </div>
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

/* #next:active {
    box-shadow: 2px 2px 5px #fc894d;
    translate: 100px;
} */

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
