<script lang="ts">
    import { emoji } from "./emoji";

    type State = 'start' | 'won' | 'lost' | 'playing' | 'paused';

    let state: State = 'start';

    let size = 20;

    let grid = createGrid();
    let maxMatches = 4 //grid.length / 2;
    let selected: number[] = []; // grid indices
    let matches: string[] = [];
    let timerId: number | null = null;
    let timeLeft = 20;


    function startTimer() {
        function countDown() {
            state !== 'paused' && (timeLeft -= 1);
        }

        timerId = setInterval(countDown, 1000);
    }

    function createGrid() {
        let cards = new Set<string>();
        let maxSize = size / 2;

        while (cards.size < maxSize) {
            let index = Math.floor(Math.random() * emoji.length);
            cards.add(emoji[index]);
        }

        return shuffle([...cards, ...cards]);

    }
    function shuffle<T>(arr: T[]): T[] {
        return arr.sort(() => Math.random() * 0.5);
    }

    function matchCards() {
        const [i, j] = selected;
        if (grid[i] === grid[j]) {
            matches = matches.concat(grid[i]);
            console.log("direct hit");
        }

        setTimeout(() => {
            selected = [];
        }, 300);
    }

    function gameWon() {
        state = 'won';
    }

    function gameLost() {
        state = 'lost';
    }

    function resetGame() {
        grid = createGrid();
        state = 'playing';
        timeLeft = 20;
        startTimer();
    }
    
    function selectCard(cardIndex: number) {
        selected = selected.concat(cardIndex);
    }

    function canShowEmoji(emoji: string) {
        return matches.includes(emoji) || selected.some(i => grid[i] === emoji);
    }
    
    // Reactive stuff
    $: maxMatches === matches.length && gameWon();

    $: selected.length === 2 && matchCards();
    $: timeLeft === 0 && gameLost();

    $: console.log({state, selected, matches});

</script>

{#if state === 'start'}
    <h1>Matching Game</h1>
    <button on:click={() => resetGame()}>
        Play
    </button>
{/if}


{#if state === 'playing'}
<h2>{timeLeft}</h2>
<div class="cards">
    {#each grid as emoji, cardIndex}

    {@const isSelected = selected.includes(cardIndex)}
    {@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(emoji)}
    {@const match = matches.includes(emoji)}
    

    <button 
        class:selected = {isSelected}
        class:match
        disabled = {isSelectedOrMatched}
        on:click={() => selectCard(cardIndex)} class="card">
        <div>{emoji}</div>
    </button>
    {/each}
</div>
{/if}

{#if state === 'lost'}
    <h2>Womp womp! Better luck next time</h2>
    <button on:click={() => resetGame()}>
        Play again
    </button>
{/if}

{#if state === 'won'}
    <h1>You win!</h1>
    <button on:click={() => resetGame()}>
        Play again
    </button>
{/if}

<style>
    .cards {
        display: grid;
        grid-template-columns: repeat(5, 1fr);
        gap: 0.4rem;
    }   

    .card {
        height: 120px;
        width: 120px;
        font-size: 4rem;
        background-color: var(--bg-2);

        &.selected {
            border: 4px solid var(--border);
        }

        &.match {
            transition: opacity 0.3s ease-out;
            opacity: 0.4;
        }
    }

    .matches {
        display: flex;
        gap: 1rem;
        margin-block: 2rem;
        font-size: 3rem;
    }
</style>