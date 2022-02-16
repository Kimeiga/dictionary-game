<script>
	import { onMount } from "svelte";
	import wordList from "./words";

	let word;
	let rhymeHint;
	let guesses = [];
	let guess;
	const maxGuesses = 6;
	let won = false;
	let lost = false;

	let validationError = "";

	const onKeyPress = (e) => {
		if (e.charCode === 13) guessed();
	};

	onMount(async () => {
		word = wordList[Math.floor(Math.random() * wordList.length)];
		console.log(word);

		getRhymeForWord(word);
	});

	function lose() {
		lost = true;
	}

	function win() {
		won = true;
	}

	async function guessed() {
		let guessArray = [];

		for (let letter of guess) {
			guessArray.push({ letter, color: "black" });
		}

		let isWord = await checkIfGuessIsEnglish(guess);

		if (!isWord) {
			validationError = `${guess} is not a word`;
			console.log(`${guess} is not a word`);
			return;
		}

		// if (wordList.includes(guess)) {
		// green letters
		for (let i = 0; i < guess.length; i++) {
			// check each guess letter against the corresponding word letter
			if (i > word.length) {
				break;
			}
			if (guess[i] == word[i]) {
				guessArray[i].color = "green";
			}
		}

		// yellow letters
		for (let i = 0; i < guess.length; i++) {
			// check each guess letter against each word letter
			for (let j = 0; j < word.length; j++) {
				if (guess[i] == word[j] && guessArray[i].color != "green") {
					console.log(
						"%c" + guess[i],
						"background: #222; color: #ff0000"
					);
					guessArray[i].color = "orange";
				}
			}
		}

		guesses = [...guesses, guessArray];
		// } else {
		// 	console.error("not in word list");
		// }

		if (guess == word) {
			// you win!
			win();
		}

		if (guesses.length == maxGuesses) {
			lose();
		}
	}

	async function checkIfGuessIsEnglish(word) {
		let isEnglish = true;
		// check if the guess is an english word
		// debugger;
		// fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${word}`)
		await fetch(`https://api.datamuse.com/words?ml=${word}`)
			.then((response) => response.json())
			.then((data) => {
				console.log(data);
				if (data.title == "No Definitions Found") {
					isEnglish = false;
				}
				if (data.length == 0) {
					isEnglish = false;
				}
			})
			.catch((e) => {
				console.error(e);
				isEnglish = false;
			});

		return isEnglish;
	}

	function getRhymeForWord(word) {
		fetch(`https://api.datamuse.com/words?rel_rhy=${word}`)
			.then((response) => response.json())
			.then((data) => {
				// TODO: if data array length is zero, find a different word
				console.log(data);
				// rhymeHint = data.reduce(function (prev, current) {
				// 	return prev.score > current.score ? prev : current;
				// }).word;
				rhymeHint = data[0].word;
				console.log(rhymeHint);
			})
			.catch((e) => console.error(e));
	}
</script>

<main>
	<h1>Rhymedime</h1>

	{#if word}
		<!-- <p>word: {word}</p> -->
		<p>letter count: {word.length}</p>
	{/if}

	{#if rhymeHint}<p>Rhymes with {rhymeHint}</p>{/if}

	{#each guesses as guess}
		<!-- <p>{guess}</p>
		{#each Array(guess.length) as _, g}
		<span>{guess.charAt(g)}</span>
		{/each} -->
		<p>
			{#each guess as g}
				<span style="color: {g.color}">{g.letter}</span>
			{/each}
		</p>
	{/each}

	{#if word}
		{#each Array(maxGuesses) as _, i}
			<div class="row">
				<!-- svelte-ignore empty-block -->
				{#if i == guesses.length}
					{#if !won}
						<!-- svelte-ignore a11y-autofocus -->
						<input
							type="text"
							name="Guess"
							id="guess"
							bind:value={guess}
							on:keypress={onKeyPress}
							autofocus
						/>
					{/if}
				{:else if i < guesses.length}{:else}
					{#if !won}
						<input type="text" disabled />
					{/if}
				{/if}
				<!-- {#each Array(word.length) as _, j}
					<div class="letterBox">{i}/{j}</div>
				{/each} -->
			</div>
		{/each}
	{/if}

	{#if won}
		<p>You won in {guesses.length} tries!</p>
	{/if}

	{#if lost}
		<p>Better luck next time! The word was {word}.</p>
	{/if}

	<br />
</main>

<style>
	.row {
		display: flex;
		justify-content: center;
	}

	.letterBox {
		height: 3rem;
		width: 3rem;
		display: flex;
		justify-content: center;
		align-items: center;
		border: 1px black solid;
	}
	main {
		text-align: center;
		padding: 1em;
		/* max-width: 240px; */
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	/* @media (min-width: 640px) {
		main {
			max-width: none;
		}
	} */
</style>
