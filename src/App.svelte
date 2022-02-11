<script>
	import { onMount } from "svelte";
	import wordList from "./words";

	let definition;
	let thesaurusData;
	let synonymsList;
	let synonym;
	let synonym2;
	let partOfSpeech;

	let guesses = [];
	const thesaurusKey = "process.env.THESAURUS_KEY";
	let guess;
	const onKeyPress = (e) => {
		if (e.charCode === 13) guessed();
	};

	onMount(async () => {
		console.log("process.env.THESAURUS_KEY");
		console.log(wordList);
		let randomWord = wordList[Math.floor(Math.random() * wordList.length)];
		console.log(randomWord);
		getSynonymForWord(randomWord);
		// getDefinitionForWordOld();
	});

	function guessed() {
		console.log(guess);

		guesses = [...guesses, guess];
		console.log(guesses);
	}

	function getSynonymForWord(word) {
		fetch(
			`https://www.dictionaryapi.com/api/v3/references/thesaurus/json/${word}?key=${thesaurusKey}`
		)
			.then((response) => response.json())
			.then((data) => {
				console.log(data);

				if (data.title == "No Definitions Found") {
					getSynonymForWord();
				} else {
					thesaurusData = data;
					synonymsList = thesaurusData[0].meta.syns[0];

					console.log(synonymsList);
					let i1 = Math.floor(Math.random() * synonymsList.length);
					synonym = synonymsList[i1];
					let i2;

					i2 = Math.floor(Math.random() * synonymsList.length);
					while (i2 == i1) {
						i2 = Math.floor(Math.random() * synonymsList.length);
						if (i2 != i1) {
							break;
						}
					}

					synonym2 = synonymsList[i2];
					console.log(synonym);
					console.log(synonym2);
				}
			})
			.catch((error) => {
				console.log(error);
				getSynonymForWord();
			});
	}

	function getDefinitionForWord(word) {
		fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${word}`)
			.then((response) => response.json())
			.then((data) => {
				console.log(data);

				if (data.title == "No Definitions Found") {
					getDefinitionForWord();
				} else {
					definition = data;
				}
			})
			.catch((error) => {
				console.log(error);
				getDefinitionForWord();
			});
	}

	function getDefinitionForWordOld() {
		fetch("https://random-word-api.herokuapp.com/word")
			.then((response) => response.json())
			.then((data) => {
				console.log(data);
				word = data[0];

				fetch(`https://api.dictionaryapi.dev/api/v2/entries/en/${word}`)
					.then((response) => response.json())
					.then((data) => {
						console.log(data);

						if (data.title == "No Definitions Found") {
							getDefinitionForWordOld();
						} else {
							definition = data;
						}
					})
					.catch((error) => {
						console.log(error);
						getDefinitionForWordOld();
					});
			})
			.catch((error) => {
				console.log(error);
				return [];
			});
	}
</script>

<main>
	<h1>Synonym Game</h1>
	<p>{synonym}, {synonym2}</p>
	<br />
	<!-- {thesaurusData} -->
	<br />
	<!-- {synonymsList} -->
	<br />

	<input
		type="text"
		name="Guess"
		id="guess"
		bind:value={guess}
		on:keypress={onKeyPress}
	/>

	{#each guesses as guess}
		<p>{guess}</p>
	{/each}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
