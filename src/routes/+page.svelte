<script lang="ts">
  import { styles } from "svelte-styling"

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
    0: "black",
    1: "green",
    2: "red"
  }
  
  function randomInt(min: number, max: number) { // inclusive on min & max
    return Math.floor(Math.random() * (max - min + 1) + min)
  }

  function randomArrows(length: number): Arrow[] {
    return Array.from({ length }, () => randomInt(0, 3))
  }

  let arrowIndex = 0
  let arrowState: TypeState[] = Array.from({ length: 20 }, () => TypeState.UNTYPED)
  let arrows = randomArrows(20)
</script>

<svelte:window on:keydown={event => {
  arrowState[arrowIndex] = nameToArrow[event.key] == arrows[arrowIndex] ? 1 : 2
  arrowIndex++;
}}></svelte:window>

{#each arrows as arrow, i}
  <span use:styles={{ color: stateToColor[arrowState[i]] }}>{arrowToSymbol[arrow]}</span>
{/each}