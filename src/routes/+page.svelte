<script lang="ts">
    import { emoji } from "./emoji";

    type State = 'start' | 'won' | 'lost' | 'playing' | 'paused';

    let state: State = 'start';

    let size = 20;
    let grid = createGrid();
    let maxMatches = grid.length / 2;
    let selected: number[] = []; // grid indices
    let matches: string[] = [];
    let timerId: number | null = null;
    let timeLeft = 60;


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
        resetGame();
    }

    function gameLost() {
        state = 'lost';
        resetGame();
    }

    function resetGame() {
        grid = createGrid();
        timeLeft = 60;
        timerId && clearInterval(timerId);
        timerId = null;
        maxMatches = grid.length/2;
        selected = [];
        matches = [];
    }
    
    function selectCard(cardIndex: number) {
        selected = selected.concat(cardIndex);
    }

    function canShowEmoji(emoji: string) {
        return matches.includes(emoji) || selected.some(i => grid[i] === emoji);
    }
    
    // Reactive stuff

    $: selected.length === 2 && matchCards();
    $: timeLeft === 0 && gameLost();
    $: if (state === 'playing') {
        // to check for paused
        !timerId && startTimer();
    }
    $: if (maxMatches === matches.length) {
        gameWon();
    }
    $: console.log({state, selected, matches, maxMatches});

</script>

{#if state === 'start'}
    <h1>Matching Game</h1>
    <button on:click={() => state = 'playing'}>
        Play
    </button>
{/if}


{#if state === 'playing'}
<h1 class="timer" class:pulse={timeLeft <= 10}>{timeLeft}</h1>
<div class="matches">
    {#each matches as match}
        <div>{match}</div>
    {/each}
</div>

<div class="cards">
    {#each grid as emoji, cardIndex}

    {@const isSelected = selected.includes(cardIndex)}
    {@const isSelectedOrMatched = selected.includes(cardIndex) || matches.includes(emoji)}
    {@const match = matches.includes(emoji)}
    

    <button 
        class:selected = {isSelected}
        class:match
        class:flip = {isSelectedOrMatched}
        disabled = {isSelectedOrMatched}
        on:click={() => selectCard(cardIndex)} class="card">
        <div class="back">{emoji}</div>
    </button>
    {/each}
</div>
{/if}

{#if state === 'lost'}
    <h1>Womp womp! Better luck next time</h1>
    <button on:click={() => state = 'playing'}>
        Play again
    </button>
{/if}

{#if state === 'won'}
    <h1>You win!</h1>
    <button on:click={() => state = 'playing'}>
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
        transition: rotate 0.3s ease-out;
        transform-style: preserve-3d;

        &.selected {
            border: 4px solid var(--border);
        }

        &.flip {
            rotate: y 180deg;
            pointer-events: none;
        }

        & .back {
            position: absolute;
			inset: 0;
			display: grid;
			place-content: center;
			backface-visibility: hidden;
			rotate: y 180deg;
        }

        &.match {
            transition: opacity 0.3s ease-out;
            opacity: 0.4;
        }
    }

    .timer {
        transition: color 0.3s ease;
    }

    .pulse {
        color: tomato;
        animation: pulse 1s infinite ease;
    }

    @keyframes pulse {
        to {
            scale: 1.4;
        }
    }

    .matches {
        display: flex;
        gap: 1rem;
        margin-block: 2rem;
        font-size: 3rem;
    }
</style>