<script>
  import axios from 'axios';
  import { onMount } from 'svelte';
  import { push } from 'svelte-spa-router';
  import NavBar from './components/NavBar.svelte';
  import { userChoosenQuestionState, userChoosenTagState } from './store';

  let questions = [];
  let loadMore = true;
  let page = 1;

  let userChoosenTag;

  userChoosenTagState.subscribe((value) => {
    userChoosenTag = value;
    if (value == null) {
      userChoosenTag = localStorage.getItem('userChoosenExploreTag');
    }
  });

  async function getQuestionsOfTag() {
    try {
      const response = await axios.get(`/questions/${userChoosenTag}`);
      // console.log(response);
      if (response.data.length < 10) {
        loadMore = false;
      }

      questions = [...response.data];
    } catch (error) {
      console.log(error);
    }
  }

  async function chooseQuestion(event) {
    let questionId = event.currentTarget.id;
    let index = questions.findIndex((question) => questionId == question.id);
    let question = questions[index];
    userChoosenQuestionState.set(question);
    push('/feed');
  }

  async function loadMoreQuestions() {
    try {
      if (loadMore == false) {
        return;
      }
      page++;

      const response = await axios.get(
        `/questions/${userChoosenTag}?page=${page}`
      );
      // console.log(response);

      if (response.status == 204) {
        loadMore = false;
        return;
      }
      if (response.data.length < 10) {
        loadMore = false;
      }

      questions = [...questions, ...response.data];
    } catch (error) {
      console.log(error);
    }
  }

  onMount(async () => {
    await getQuestionsOfTag();
  });
</script>

<NavBar />

<div class="container mt-3 mb-3">
  <div class="list-group w-auto">
    <button type="button" class="btn btn-primary mb-2"
      ># {userChoosenTag}</button
    >
    <!-- <p class="lead">
      
    </p> -->
    {#each questions as question}
      <div
        id={question.id}
        on:click={chooseQuestion}
        on:keypress={null}
        class="list-group-item list-group-item-action d-flex gap-3 py-3 border rounded my-1"
        aria-current="true"
      >
        <div class="d-flex gap-2 w-100 justify-content-between">
          <div>
            <p class="mb-0 opacity-75">
              {question.question}
            </p>
          </div>
          <small class="text-nowrap"
            ><i
              class="bi bi-chevron-right"
              style="font-size: 1.5rem; color: #3366FF;"
            /></small
          >
        </div>
      </div>
    {/each}
  </div>
  {#if loadMore != false}
    <div class="text-center mt-3">
      <button
        on:click={loadMoreQuestions}
        type="button"
        class="btn btn-outline-primary mx-2 rounded-pill"
        ><i class="bi bi-plus" />See More</button
      >
    </div>
  {/if}
</div>

<div class="container mb-2 bottom-0">
  <img src="/assets/images/ask.png" class="img-fluid" alt="noanswer" />
</div>
