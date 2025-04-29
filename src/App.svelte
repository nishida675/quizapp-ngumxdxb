<script>
  import { onMount } from "svelte";
  import { getQuizdata } from "./quizdataFactory";
  import AnswerButton from "./AnswerButton.svelte";
  import TitlePage from "./TitlePage.svelte";
  import Progressbar from "./Progressbar.svelte";
  import GameoverModal from "./GameoverModal.svelte";
  import FancyScore from "./FancyScore.svelte";

  let gameoverModal;
  const [
    InitialState,
    TitleState,
    QuestionState,
    AnswerState,
    GameoverState,
    QuizStart,
  ] = [0, 1, 2, 3, 4, 5];
  let state = InitialState;

  const maxTime = 10;
  let time = maxTime;
  let quizdata;
  let fancyScore;

  onMount(changeToTitle);

  function changeToTitle() {
    if (gameoverModal && typeof gameoverModal.closeModal === "function") {
      gameoverModal.closeModal();
    }
    state = TitleState;
  }

  function changeToQuestion() {
    quizdata = getQuizdata();
    state = QuestionState;
  }

  function changeToAnswer() {
    state = AnswerState;
    setTimeout(() => {
      if (state === AnswerState) {
        changeToQuestion();
      }
    }, 1000);
  }

  function changeToGameover() {
    state = GameoverState;
    gameoverModal.showModal(fancyScore.getScore());
  }

  function changeToQuizstart() {
    state = QuizStart;
    time = maxTime;
    const timer = setInterval(() => {
      if (state === QuestionState && time < 0) {
        clearInterval(timer);
        changeToGameover();
      }
      if (state === QuestionState) {
        time -= 0.1;
      }
    }, 100);
    changeToQuestion();
  }

  function answerButtonClicked(isCorrect) {
    if (state !== QuestionState) return;
    fancyScore.updateScore(isCorrect);
    if (fancyScore.getRenzokuSeikai() >= 5) {
      time = Math.min(time + 1, maxTime);
    } else if (!isCorrect) {
      time -= 1;
    }
    changeToAnswer();
  }
</script>

<svelte:head>
  <title>QuizApp</title>
</svelte:head>

<main
  class="flex flex-col h-svh bg-[url('/src/assets/haikei.jpeg')] bg-[size:100%] bg-repeat"
>
  {#if state === TitleState}
    <TitlePage on:click={changeToQuizstart}></TitlePage>
  {:else if state === QuestionState || state === AnswerState || state === GameoverState}
    <!-- メニューバー -->
    <FancyScore bind:this={fancyScore} />

    <!--プログレスバー-->
    <Progressbar {maxTime} {time} />

    <!-- 問題 -->
    <div class="bg-white/60 font-extrabold text-center text-4xl py-4">
      {quizdata.mondai}
    </div>

    <!-- 選択肢 -->
    <div
      class="bg-white/30 flex flex-col justify-around flex-grow items-center"
    >
      {#each quizdata.taku as t}
        <AnswerButton
          isGrayout={state === AnswerState && quizdata.seikai !== t}
          on:click={() => answerButtonClicked(quizdata.seikai === t)}
          >{t}
        </AnswerButton>
      {/each}
    </div>
  {/if}
</main>

<GameoverModal bind:this={gameoverModal} on:click={changeToTitle} />
