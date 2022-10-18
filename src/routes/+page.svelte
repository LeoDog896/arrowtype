<script lang="ts">
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
}}></svelte:window>

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
    color: black;
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
  }

  .active {
    animation: blink-animation 1.5s infinite;
    background-color: orange;
  }

  @keyframes blink-animation {
    0% { background-color: rgba(255, 166, 0, 0.464) }
    50% { background-color: rgba(255, 166, 0, 0.264) }
    100% { background-color: rgba(255, 166, 0, 0.464) }
  }

  @keyframes success {
    from { color: green; background-color: lightgreen; }
    to { color: black; background-color: white; }
  }

  span {
    font-size: 2rem;
    padding: 0.5rem;
  }
</style>