<script>
  import { onMount } from 'svelte';
  import axios from 'axios';
  import { inview } from 'svelte-inview/dist/index';

  let profileUrl = '/assets/images/profile/';

  export let sortType;
  export let selectedQuestionId;

  let page = 1;
  let loadMore = true;
  let startLoadMore = false;
  $: answers = [];

  // let answers = [
  //   {
  //     uniqueAlias: 'rider_ritik',
  //     answeredBy: '1',
  //     answer: `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
  //       tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
  //       veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
  //       commodo consequat. Duis aute irure dolor in reprehenderit in voluptate
  //       velit esse cillum dolore eu fugia`,
  //     id: '1111',
  //     likeNumber: 100,
  //     liked: true,
  //     profilePictureCode: 1,
  //   },
  //   {
  //     uniqueAlias: 'rider_ritik',
  //     answeredBy: '2',
  //     answer: `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
  //       tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
  //       veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
  //       commodo consequat. Duis aute irure dolor in reprehenderit in voluptate
  //       velit esse cillum dolore eu fugia`,
  //     id: '2222',
  //     likeNumber: 100,
  //     liked: false,
  //     profilePictureCode: 2,
  //   },
  //   {
  //     uniqueAlias: 'rider_ritik',
  //     answeredBy: '33333',
  //     answer: `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
  //       tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim
  //       veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea
  //       commodo consequat. Duis aute irure dolor in reprehenderit in voluptate
  //       velit esse cillum dolore eu fugia`,
  //     id: '3',
  //     likeNumber: 100,
  //     liked: true,
  //     profilePictureCode: 3,
  //   },
  // ];

  function findLikeClassAndTime() {
    answers.forEach((answer) => {
      if (answer.liked == true) {
        answer.likeClass = 'bi bi-heart-fill';
      } else {
        answer.likeClass = 'bi bi-heart';
      }
      let time = Math.floor(Date.now() / 1000);
      let newTime = time - answer.createdAt;
      let timehrs = Math.floor(newTime / 3600);
      let timemin = Math.floor(newTime / 60);
      if (timemin <= 59) {
        answer.createdAt = timemin + ' min';
      } else if (timehrs >= 1) {
        answer.createdAt = timehrs + ' hrs';
      }
    });
  }

  async function getAnswers() {
    try {
      const response = await axios.get(
        `/answers/${selectedQuestionId}?sort=${sortType}`
      );
      console.log(response);
      if (response.status == 204) {
        loadMore = false;
        return;
      }

      answers = [...response.data];
      findLikeClassAndTime();
    } catch (error) {
      console.log(error);
    }
  }
  async function loadMoreAnswers() {
    try {
      if (loadMore == false) {
        return;
      }
      page++;

      const response = await axios.get(
        `/answers/${selectedQuestionId}?sort=${sortType}&page=${page}`
      );
      console.log(response);

      if (response.status == 204) {
        loadMore = false;
        return;
      }

      answers = [...answers, ...response.data];
      findLikeClassAndTime();
    } catch (error) {
      console.log(error);
    }
  }

  if (sortType != undefined && selectedQuestionId != undefined) {
    onMount(async () => {
      console.log(sortType, selectedQuestionId);
      await getAnswers();
      // startLoadMore = true;
    });
  }

  async function likeAnswer(answerId, answeredBy) {
    try {
      const response = await axios.put(`/like/${answerId}/${answeredBy}`);
      console.log(response);
    } catch (error) {
      console.log(error);
    }
  }
  async function cancellikeAnswer(answerId, answeredBy) {
    try {
      const response = await axios.put(`/cancelLike/${answerId}/${answeredBy}`);
      console.log(response);
    } catch (error) {
      console.log(error);
    }
  }

  function toggleLike(event) {
    //get answer id from event
    let answerId = event.currentTarget.id;

    let index = answers.findIndex((answer) => answer.id == answerId);

    if (answers[index].liked == true) {
      answers[index].liked = false;
      answers[index].likeNumber -= 1;
      answers[index].likeClass = 'bi bi-heart';
      cancellikeAnswer(answerId, answers[index].answeredBy);
    } else {
      answers[index].liked = true;
      answers[index].likeNumber += 1;
      answers[index].likeClass = 'bi bi-heart-fill';
      likeAnswer(answerId, answers[index].answeredBy);
    }
  }
  let isInView;
  const options = {};
</script>

<div>
  {#each answers as answer}
    <div class="card border-light mt-4 shadow-sm rounded">
      <div class="card-header bg-white border-light">
        <img
          src={profileUrl + 'pic' + answer.profilePictureCode + '.png'}
          alt=""
          height="30"
        />
        <b>{answer.answeredBy}</b> &emsp;
        <small class="text-muted">{answer.createdAt}</small>
      </div>
      <div class="card-body text-secondary">
        <p class="card-text">
          {answer.answer}
        </p>
      </div>
      <div class="card-body text-secondary">
        <input type="button" class="btn-check" id={answer.id} />
        <label
          style="border-radius: 100%;"
          class="btn btn-outline-primary"
          for={answer.id}
          id={answer.id}
          on:click={toggleLike}
          on:keydown={null}><i class={answer.likeClass} /></label
        > <small class="text-muted">{answer.likeNumber} likes</small> &ensp;
      </div>
    </div>
  {/each}
  {#if answers.length >= 1}
    <div use:inview={{}} on:enter={loadMoreAnswers} />
  {/if}
</div>
