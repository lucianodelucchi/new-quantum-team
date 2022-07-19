<script>
  import { slide } from "svelte/transition";

  const api = '/api';

  let isQuantumSeed = false;

  // https://stackoverflow.com/questions/2450954/how-to-randomize-shuffle-a-javascript-array
  async function shuffle(array = [], seeds = []) {
    var currentIndex = array.length,
      temporaryValue,
      randomIndex;

    // While there remain elements to shuffle...
    while (0 !== currentIndex) {
      // Pick a remaining element...
      randomIndex = Math.floor(seeds[currentIndex-1] * currentIndex);
      currentIndex -= 1;

      // And swap it with the current element.
      temporaryValue = array[currentIndex];
      array[currentIndex] = array[randomIndex];
      array[randomIndex] = temporaryValue;
    }

    return array;
  }

  async function getSeeds(){
    try {
      const response = await fetch(api);
      const json = await response.json();
      console.log("Got:", json);

      isQuantumSeed = json.type == 'quantum';

      return json.numbers;
    } catch (error) {
      console.warn("Error while calling the API, fall back to Math.random()", error);
      return [Math.random(), Math.random(), Math.random(), Math.random()];
    }
  }

  let player1 = "Fredy";
  let player2 = "Javi";
  let player3 = "Luciano";
  let player4 = "Stefano";

  let team1 = [];
  let team2 = [];
  
  let buildingTeams = false;

  $: canBuildTeams = player1 !== "" && player2 !== "" && player3 !== "" && player4 !== "";

  $: teamsCompleted = team1.some(() => true) && team2.some(() => true);

  let history = [];

  $: shouldShowHistory = history.some(() => true);

  async function handleClick(event) {
    buildingTeams = true;
    const players = [player1, player2, player3, player4];
    const seeds = await getSeeds();
    const shuffledPlayers = await shuffle(players, seeds);

    team1 = shuffledPlayers.slice(0, 2);
    team2 = shuffledPlayers.slice(2, 4);
    history = [{ team1, team2 }, ...history];
    buildingTeams = false;
  }

  function reset() {
    team1 = [];
    team2 = [];
  }
</script>

<main>
  <h1 on:click={reset}>
    The Quantum RNG<span title="It may use JS Math.random() when API is not available 🤷‍♂️">*</span> Team builder
    <div class="disclaimer">*It may use JS Math.random() when API is not available 🤷‍♂️</div>
  </h1>
  
  {#if !teamsCompleted}
    <div transition:slide class="form">
      <input type="text" bind:value={player1} />
      <input type="text" bind:value={player2} />
      <input type="text" bind:value={player3} />
      <input type="text" bind:value={player4} />
      <br/>
      <button on:click={handleClick} disabled={!canBuildTeams || buildingTeams}>
        Build teams
      </button>
    </div>
  {/if}
  
  {#if buildingTeams}
    <div class="building-teams">⚙⚛⚙⚛</div>
    <h2>Crunching some quatum numbers and building the teams...</h2>
    <div class="building-teams">⚙⚛⚙⚛</div>
  {/if}

  {#if teamsCompleted}
    <div class="teams" transition:slide>
      <div>🎾🙆‍♂️🎾🙆‍♂️</div>
      <div>{team1}</div>
      <div>🆚</div>
      <div>{team2}</div>
      <div>🎾🙆‍♂️🎾🙆‍♂️</div>
      {#if !isQuantumSeed}
        <details>
          <summary>Pseudo random seeds used (click for more details)</summary>
          <p>The results were calculated using pseudo random seeds, for some reason the Quatum API didn't work.</p>
        </details>
      {/if}
    </div>
  {/if}
  {#if shouldShowHistory}
    <section class="history">
      <h3>Teams History</h3>
      {#each history as { team1, team2 }}
        <div class="entry">
          <div class="left">{team1}</div>
          <div class="vs">🆚</div>
          <div class="right">{team2}</div>
        </div>
      {/each}
    </section>
    {/if}
  <section class="credits">
    <h4>Credits</h4>
    <ul>
      <li>
        <a href="http://qrng.ethz.ch/">Quantum RNG API@ETH Zürich</a> (It wasn't working for a few weeks, so look out for quantum shortages)
      </li>
      <li>
        Visit the <a href="https://svelte.dev/tutorial">Svelte tutorial</a> to learn how to build Svelte apps.
      </li>
      <li>
        CI/CD & serverless functions by <a href="https://vercel.com/">Vercel</a>
      </li>
    </ul>
  </section>
</main>
<footer>
  <span class="version">
    version:
    {#if import.meta.env.deployedUsingGithub && import.meta.env.username}
      <a href="https://github.com/{import.meta.env.username}/{import.meta.env.repo}/commit/{import.meta.env.version}">{import.meta.env.version.slice(0,7)} ¯\_(ツ)_/¯ by {import.meta.env.author}✍</a>
    {:else}
      TBD
    {/if}
  </span>
</footer>

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

  div.teams {
    font-size: xx-large;
  }

  div.teams details {
    display: inline-block;
    font-size: small;
    margin: 1rem;
  }

  section.history div.entry {
    align-items: center;
    display: flex;
    justify-content: space-evenly;
  }

  section.history div.entry div.left {
    text-align: right;
  }

  section.history div.entry div.vs {
    margin: 0 2rem;
  }
  section.history div.entry div.right {
    text-align: left;
  }

  section.history div.entry div.left,
  section.history div.entry div.right {
    width: 10rem;
  }

  .building-teams {
        animation: blink 1s ease-out infinite;
        font-size: x-large;
  }
  @keyframes blink {
      50% {
          opacity: 0;
      }
      100% {
          opacity: 1;
      }
  }

  div.form{
    margin: 2rem 0;
  }

  footer{
    text-align: right;
  }

  .version{
    font-size: x-small;
  }

  h1 div.disclaimer{
    font-size: x-small;
  }

  section.credits {
    font-size: small;
    text-align: left;
  }
</style>