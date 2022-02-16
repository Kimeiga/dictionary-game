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

	let time = new Date();
	$: hours = 23 - time.getHours();
	$: minutes = 59 - time.getMinutes();
	$: seconds = 59 - time.getSeconds();

	let isCopied = false;

	const onKeyPress = (e) => {
		validationError = "";
		if (!guess) return;
		if (guess == rhymeHint) {
			validationError = "You can't use the rhyme hint.";
			return;
		}
		if (e.charCode === 13) guessed();
	};

	onMount(async () => {
		let goodWord = true;
		do {
			word = wordList[Math.floor(Math.random() * wordList.length)];
			console.log(word);

			await getRhymeForWord(word);

			let wordLower = word.toLowerCase();
			let hintLower = rhymeHint.toLowerCase();

			// make sure its a good word
			if (
				hintLower.includes(wordLower) ||
				wordLower.includes(hintLower)
			) {
				goodWord = false;
			} else if (wordLower.length < 5) {
				goodWord = false;
			} else {
				goodWord = true;
			}
		} while (!goodWord);

		const interval = setInterval(() => {
			time = new Date();
		}, 1000);

		return () => {
			clearInterval(interval);
		};
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

		let wordLower = word.toLowerCase();
		let guessLower = guess.toLowerCase();

		if (!isWord) {
			validationError = `${guess} is not a word`;
			return;
		}

		// if (wordList.includes(guess)) {
		// green letters
		for (let i = 0; i < guessLower.length; i++) {
			// check each guess letter against the corresponding word letter
			if (i > wordLower.length) {
				break;
			}
			if (guessLower[i] == wordLower[i]) {
				guessArray[i].color = "green";
			}
		}

		// yellow letters
		for (let i = 0; i < guessLower.length; i++) {
			// check each guess letter against each word letter
			for (let j = 0; j < wordLower.length; j++) {
				if (
					guessLower[i] == wordLower[j] &&
					guessArray[i].color != "green"
				) {
					guessArray[i].color = "orange";
				}
			}
		}

		guesses = [...guesses, guessArray];
		// } else {
		// 	console.error("not in word list");
		// }

		if (guessLower == wordLower) {
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

	async function getRhymeForWord(word) {
		await fetch(`https://api.datamuse.com/words?rel_rhy=${word}`)
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

	function generateSharableText() {
		if (guesses.length == 0) {
			return;
		}
		let guessTextArray = [];
		for (let i = 0; i < guesses.length; i++) {
			guessTextArray[i] = "";
			for (let j = 0; j < guesses[i].length; j++) {
				if (guesses[i][j].color == "green") {
					guessTextArray[i] += "🟩";
				} else if (guesses[i][j].color == "orange") {
					guessTextArray[i] += "🟨";
				} else {
					guessTextArray[i] += "⬛";
				}
			}
		}
		console.log(guessTextArray);

		let guessesText = guessTextArray.join("\n");

		let text =
			`Rhymedime 0 ${guesses.length}/6
	
` + guessesText;
		console.log(text);

		navigator.clipboard.writeText(text).then(
			function () {
				console.log("Async: Copying to clipboard was successful!");
			},
			function (err) {
				console.error("Async: Could not copy text: ", err);
			}
		);

		isCopied = true;
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

	{#if validationError}
		<p style="color: red;">{validationError}</p>
	{/if}

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

	{#if won || lost}
		<div class="container">
			<div class="footer">
				<div class="countdown">
					<p class="timer">
						Next word in {hours}:{minutes}:{seconds}
					</p>
				</div>
				<div class="share">
					<button id="share-button" on:click={generateSharableText}>
						Share <svg
							xmlns="http://www.w3.org/2000/svg"
							height="12"
							viewBox="0 0 24 24"
							width="12"
						>
							<path
								fill="var(--white)"
								d="M18 16.08c-.76 0-1.44.3-1.96.77L8.91 12.7c.05-.23.09-.46.09-.7s-.04-.47-.09-.7l7.05-4.11c.54.5 1.25.81 2.04.81 1.66 0 3-1.34 3-3s-1.34-3-3-3-3 1.34-3 3c0 .24.04.47.09.7L8.04 9.81C7.5 9.31 6.79 9 6 9c-1.66 0-3 1.34-3 3s1.34 3 3 3c.79 0 1.5-.31 2.04-.81l7.12 4.16c-.05.21-.08.43-.08.65 0 1.61 1.31 2.92 2.92 2.92s2.92-1.31 2.92-2.92c0-1.61-1.31-2.92-2.92-2.92zM18 4c.55 0 1 .45 1 1s-.45 1-1 1-1-.45-1-1 .45-1 1-1zM6 13c-.55 0-1-.45-1-1s.45-1 1-1 1 .45 1 1-.45 1-1 1zm12 7.02c-.55 0-1-.45-1-1s.45-1 1-1 1 .45 1 1-.45 1-1 1z"
							/>
						</svg>
						{#if isCopied}
							<p>Copied performance to clipboard!</p>
						{/if}
					</button>
				</div>
			</div>
		</div>
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
