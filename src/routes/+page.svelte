<script lang="ts">
	import { onMount } from 'svelte';
	import { styles } from 'svelte-styling';

	// was the last move correct? this stops false fails
	let correct = false;
	// the "cursor" -- what note is currently being played
	let cursor = 0;

	let markTime = 0;

	// bpm
	const bpm = 80;
	$: metronome = 1000 / (bpm / 60);
	// margin of error for human reaction speed
	let coyoteTime = 50;

  $: totalTime = metronome + coyoteTime
	let barPercentage = 0;
	$: {
		if (cursor < arrows.length) {
			let newHeight = (
        (currentTime - markTime) // the difference in time
        % totalTime
      ) / (totalTime / (100 + coyoteTime / 10));

			if (newHeight < barPercentage && !correct) {
				arrows[cursor].state = TypeState.FAIL;
				cursor++;
				markTime = currentTime % totalTime;
				correct = false;
			}
			barPercentage = newHeight;
		}
	}

	let currentTime: number;

	onMount(() => {
		currentTime = Date.now();
		markTime = currentTime % totalTime;
		time();
	});

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
		0: '\u2190',
		1: '\u2191',
		2: '\u2192',
		3: '\u2193'
	};

	const nameToArrow: { [key: string]: Arrow } = {
		ArrowLeft: Arrow.LEFT,
		ArrowUp: Arrow.UP,
		ArrowRight: Arrow.RIGHT,
		ArrowDown: Arrow.DOWN
	};

	function time() {
		currentTime = Date.now();
		requestAnimationFrame(time);
	}

	function randomInt(min: number, max: number) {
		// inclusive on min & max
		return Math.floor(Math.random() * (max - min + 1) + min);
	}

	function randomArrows(
		length: number
	): { arrow: Arrow; state: TypeState; arrowElement?: HTMLSpanElement }[] {
		return Array.from({ length }, () => ({
			arrow: randomInt(0, 3),
			state: TypeState.UNTYPED
		}));
	}

	let arrows = randomArrows(200);
</script>

<svelte:window
	on:keydown={(event) => {
		if (nameToArrow[event.key] === undefined) return;
		arrows[cursor] = {
			...arrows[cursor],
			state: nameToArrow[event.key] == arrows[cursor].arrow ? TypeState.SUCCESS : TypeState.FAIL
		};
		cursor++;
		correct = true;
		markTime = currentTime % totalTime;
	}}
/>

<div class="container">
	{#each arrows as { arrow, state, arrowElement }, i}
		{@const color = TypeState[state].toLowerCase()}
		{@const boundingBox = arrowElement?.getBoundingClientRect()}
		{@const top = boundingBox?.top ?? 0}
		{@const left = boundingBox?.left ?? 0}
		{@const right = boundingBox?.right ?? 0}
		{@const bottom = boundingBox?.bottom ?? 0}
		{@const height = bottom - top}
		{@const active = i == cursor}
		{#if active}
			<div
				class="bar"
				use:styles={{
					top: top - height * (Math.min(barPercentage, 100) / 100) + height + 'px',
					left: left + 'px',
					width: right - left + 'px',
					height: height * (Math.min(barPercentage, 100) / 100) + 'px'
				}}
			/>
		{/if}
		<span bind:this={arrowElement} class="{color} {active ? 'active' : ''}"
			>{arrowToSymbol[arrow]}</span
		>
	{/each}
</div>

<style>
	.untyped {
		color: darkgray;
	}

	.success {
		color: white;
		animation-name: success;
		animation-timing-function: ease-out;
		animation-duration: 4s;
	}

	.fail {
		color: red;
	}
	.container {
		margin: 4rem;
	}

	@keyframes success {
		from {
			color: rgba(0, 128, 0, 0.438);
			background-color: rgba(144, 238, 144, 0.662);
		}
		to {
			color: white;
			background-color: white;
		}
	}

	span {
		font-size: 2rem;
		display: inline-block;
		padding: 0.5rem;
	}

	.score {
		width: 100%;
		font-size: 2rem;
		font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
	}

	.active {
		border-top: 1px solid red;
	}

	.bar {
		background-color: lightcoral;
		position: absolute;
		height: 4rem;

		z-index: -1;
	}

	.score {
		position: absolute;
		top: 0;
		left: 0;
	}
</style>
