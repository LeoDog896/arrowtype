<script lang="ts">
	import { onMount } from "svelte";
  import { styles } from "svelte-styling"
  import { fade } from "svelte/transition"

  let playing = true

  /** synchronization between the player playing and the game playing */
  let correct: boolean
  let markTime = 0;

  // bpm
  const bpm = 80
  $: metronome = 1000 / (bpm / 60);
  let width = 0
  $: {
    if (cursor < arrows.length) {
      let newWidth = ((currentTime - markTime) % metronome) / (metronome / 100)
      if (newWidth < width && !correct) {
        arrows[cursor] = {
          ...arrows[cursor],
          state: TypeState.FAIL
        }
        cursor++;
      }
      correct = false
      width = newWidth
    } else { 
      playing = false
    }
  }

  let currentTime: number;

  onMount(() => {
    currentTime = Date.now()
    time()
  })

  enum Arrow {
    LEFT = 0,
    UP = 1,
    RIGHT = 2,
    DOWN = 3
  }

  enum TypeState {
    UNTYPED = 0,
    SUCCESS = 1,
    FAIL = 2
  }

  const arrowToSymbol: { [key in Arrow]: string } = { 
    0: "\u2190",
    1: "\u2191",
    2: "\u2192",
    3: "\u2193",
  }

  const nameToArrow: { [key: string]: Arrow } = {
    "ArrowLeft": Arrow.LEFT,
    "ArrowUp": Arrow.UP,
    "ArrowRight": Arrow.RIGHT,
    "ArrowDown": Arrow.DOWN
  }

  const stateToColor: { [key in TypeState]: string } = {
    0: "untyped",
    1: "success",
    2: "fail"
  }

  function time() {
    currentTime = Date.now()
    requestAnimationFrame(time)
  }
  
  function randomInt(min: number, max: number) { // inclusive on min & max
    return Math.floor(Math.random() * (max - min + 1) + min)
  }

  function randomArrows(length: number): { arrow: Arrow, state: TypeState }[] {
    return Array.from({ length }, () => ({
      arrow: randomInt(0, 3),
      state: TypeState.UNTYPED
    }))
  }

  let cursor = 0
  let arrows = randomArrows(200)
</script>

<svelte:window on:keydown={event => {
  if (nameToArrow[event.key] === undefined) return
  arrows[cursor] = {
    ...arrows[cursor],
    state: nameToArrow[event.key] == arrows[cursor].arrow ? TypeState.SUCCESS : TypeState.FAIL
  }
  cursor++;
  correct = true
  markTime = currentTime % metronome
}}></svelte:window>

{#if playing}
  <div class="bar" out:fade use:styles={{ width: width + "%" }}></div>
{:else}
  <h1 class="score">Score</h1>
{/if}
<div class="container">
  {#each arrows as { arrow, state }, i}
    {@const color = stateToColor[state] }
    <span class="{color} {i == cursor ? "active" : ""}">{arrowToSymbol[arrow]}</span>
  {/each}
</div>

<style>
  .untyped {
    color: darkgray
  }

  .success {
    color: white;
    animation-name: success;
    animation-timing-function: ease-out;
    animation-duration: 4s;
  }

  .fail {
    color: red
  }
  .container {
    word-wrap: break-word;
    margin: 4rem;
    margin-top: 12rem;
  }

  .active {
    animation: blink-animation 1.5s infinite;
    background-color: orange;
  }

  @keyframes blink-animation {
    0% { background-color: rgba(255, 166, 0, 0.564) }
    50% { background-color: rgba(255, 166, 0, 0.264) }
    100% { background-color: rgba(255, 166, 0, 0.564) }
  }

  @keyframes success {
    from { color: rgba(0, 128, 0, 0.438); background-color: rgba(144, 238, 144, 0.662); }
    to { color: white; background-color: white; }
  }

  span {
    font-size: 2rem;
    padding: 0.5rem;
  }

  .bar {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 4rem;
    background-color: green;
  }

  .score {
    position: absolute;
    top: 0;
    left: 0;
  }
</style>