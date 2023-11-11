<script lang="ts">
    import { emoji } from "./emoji";

    type State = 'start' | 'won' | 'lost' | 'playing' | 'pause';

    let state: State = 'start';

    let size = 20;

    let grid = createGrid();
    let maxMatches = grid.length / 2;
    let selected: number[] = []; // grid indices
    let matches: string[] = [];

    function createGrid() {
        let cards = new Set<string>();
        let maxSize = size / 2;

        while (cards.size < maxSize) {
            // select rand index from emoji
            // pop it
            // push it
            let index = Math.floor(Math.random() * emoji.length);
            cards.add(emoji[index]);
        }

        return shuffle([...cards, ...cards]);
    }
    function shuffle<T>(arr: T[]): T[] {
        return arr.sort(() => Math.random() * 0.5);
    }

    console.log(grid);
</script>



{#if state === 'start'}
    <h1>Matching Game</h1>
    <button on:click={() => state = 'playing'}>
        Play
    </button>
{/if}


{#if state === 'playing'}
<div class="cards">
    {#each grid as card, cardIndex}
    <button class="card">
        <div>{card}</div>
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

        &.selected {
            border: 4px solid var(--border);
        }
    }
</style>