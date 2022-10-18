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

  function randomArrows(length: number): Arrow[] {
    return Array.from({ length }, () => randomInt(0, 3))
  }

  let arrowIndex = 0
  let arrowState: TypeState[] = Array.from({ length: 200 }, () => TypeState.UNTYPED)
  let arrows = randomArrows(200)
</script>

<svelte:window on:keydown={event => {
  if (nameToArrow[event.key] === undefined) return
  arrowState[arrowIndex] = nameToArrow[event.key] == arrows[arrowIndex] ? 1 : 2
  arrowIndex++;
}}></svelte:window>

<div>
  {#each arrows as arrow, i}
    {@const color = stateToColor[arrowState[i]] }
    <span class="{color}">{arrowToSymbol[arrow]}</span>
  {/each}
</div>

<style>
  .untyped {
    color: darkgray
  }

  .success {
    color: green
  }

  .fail {
    color: red
  }
  div {
    word-wrap: break-word;
  }
  span {
    font-size: 2rem;
    padding: 0.5rem;
  }
</style>