<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import { ChevronLeft } from 'lucide-svelte';

	type Tile = 'red' | 'blue' | 'none';
	type Board = Array<Array<Tile>>;

	function newBoard() {
		let board: Array<Array<Tile>> = [];

		for (let i = 0; i < 7; i++) {
			// 7 columns
			let column: Array<Tile> = [];
			for (let j = 0; j < 6; j++) {
				// 6 rows
				column.push('none');
			}
			board.push(column);
		}

		return board;
	}

	let player: boolean = true;
	let board: Board = newBoard();
	let highlightedColumn = -1;
	let hilightedRow = -1;
	let won = false;

	function getLowestEmptyCell(columnIndex: number) {
		let column = board[columnIndex];

		for (let i = 0; i < column.length; i++) {
			let piece = column[i];

			if (piece !== 'none') {
				return i - 1; // Return the index of the last non-empty cell
			}
		}

		return 5; // Return -1 if the column is full
	}

	function highlightColumn(columnIndex: number) {
		highlightedColumn = columnIndex;
		hilightedRow = getLowestEmptyCell(columnIndex);
	}

	function clearHighlight() {
		highlightedColumn = hilightedRow = -1;
	}

	let winningPieces: Array<Array<number>> = [];
	function checkWin(): boolean {
		// horizontal
		for (let row = 0; row < 6; row++) {
			for (let col = 0; col < 4; col++) {
				let p = board[col][row];
				if (
					p !== 'none' &&
					p === board[col + 1][row] &&
					p === board[col + 2][row] &&
					p === board[col + 3][row]
				) {
					winningPieces = [
						[col, row],
						[col + 1, row],
						[col + 2, row],
						[col + 3, row]
					];
					console.log(winningPieces);

					return true;
				}
			}
		}

		// vertical
		for (let col = 0; col < 7; col++) {
			for (let row = 0; row < 3; row++) {
				// Ensure it checks the correct number of rows
				let p = board[col][row];
				if (
					p !== 'none' &&
					p === board[col][row + 1] &&
					p === board[col][row + 2] &&
					p === board[col][row + 3]
				) {
					winningPieces = [
						[col, row],
						[col, row + 1],
						[col, row + 2],
						[col, row + 3]
					];
					return true;
				}
			}
		}

		// diagonal tl -> br  \
		for (let row = 0; row < 3; row++) {
			for (let col = 3; col < 7; col++) {
				if (
					board[col][row] !== 'none' &&
					board[col][row] === board[col - 1][row + 1] &&
					board[col][row] === board[col - 2][row + 2] &&
					board[col][row] === board[col - 3][row + 3]
				) {
					winningPieces = [
						[col, row],
						[col - 1, row + 1],
						[col - 2, row + 2],
						[col - 3, row + 3]
					];
					return true;
				}
			}
		}

		// diagonal tr -> bl  /
		for (let row = 0; row < 3; row++) {
			for (let col = 0; col < 4; col++) {
				if (
					board[col][row] !== 'none' &&
					board[col][row] === board[col + 1][row + 1] &&
					board[col][row] === board[col + 2][row + 2] &&
					board[col][row] === board[col + 3][row + 3]
				) {
					winningPieces = [
						[col, row],
						[col + 1, row + 1],
						[col + 2, row + 2],
						[col + 3, row + 3]
					];
					return true;
				}
			}
		}

		return false;
	}

	function play(columnIndex: number) {
		if (getLowestEmptyCell(columnIndex) >= 0 && !won) {
			board[columnIndex][getLowestEmptyCell(columnIndex)] = player ? 'blue' : 'red';
			highlightColumn(columnIndex);
			console.log(board);

			if (checkWin()) {
				won = true;
				// Handle win (e.g., display a win message)
				console.log(`Player ${player ? 'Blue' : 'Red'} wins!`);
				// Optionally, clear the board or end the game
			} else {
				player = !player;
			}
		}
	}

	function reset() {
		board = newBoard();
		won = false;
		player = true;
	}
</script>

<div
	class="h-screen w-screen overflow-hidden
	{won && player ? 'bg-sky-400/20 dark:bg-sky-700/20' : ''} 
  {won && !player ? 'bg-rose-400/20 dark:bg-rose-700/20' : ''}"
>
	<a href="/" class="absolute">
		<ChevronLeft size="64" class="p-3 opacity-90" />
	</a>
	<div
		class="
	mx-auto my-auto flex h-screen max-w-[800px] flex-col justify-center align-middle transition-all"
	>
		<div class="flex-1" />
		<div
			class="mx-3 my-auto grid grid-cols-7 rounded-3xl bg-zinc-200/80 p-3 dark:bg-zinc-500/10 sm:mx-10 sm:p-5"
		>
			{#each board as column, columnIndex}
				<div class="flex flex-col">
					{#each column as piece, rowIndex}
						<button
							class="flex aspect-square p-2 transition-all sm:p-3 md:p-4
							{highlightedColumn === columnIndex && !won && player ? 'bg-sky-500/10' : ''}
							{highlightedColumn === columnIndex && !won && !player ? 'bg-rose-500/10' : ''}
							{rowIndex === 0 ? 'rounded-t-full' : ''}
							{rowIndex === 5 ? 'rounded-b-full' : ''}"
							on:mouseover={() => highlightColumn(columnIndex)}
							on:mouseout={clearHighlight}
							on:click={() => play(columnIndex)}
						>
							<div
								class="h-full w-full rounded-full bg-background transition-all
								{hilightedRow === rowIndex && !won && highlightedColumn === columnIndex && player
									? 'bg-blue-500 opacity-30 ring-8 ring-blue-500/50'
									: ''}
              	{hilightedRow === rowIndex && !won && highlightedColumn === columnIndex && !player
									? 'bg-red-500 opacity-30 ring-8 ring-red-500/50'
									: ''}
              	{piece === 'red' ? 'bg-red-500 ring-8 ring-red-500/50' : ''}
              	{piece === 'blue' ? 'bg-blue-500 ring-8 ring-blue-500/50' : ''}
								{won == true &&
								!winningPieces.some(
									(winningPiece) => winningPiece[0] === columnIndex && winningPiece[1] === rowIndex
								)
									? 'opacity-30'
									: ''}"
							/>
							<!-- <span class="absolute">
								{JSON.stringify([columnIndex, rowIndex])}
							</span> -->
						</button>
					{/each}
				</div>
			{/each}
		</div>
		<div class="flex-1">
			<div class="flex justify-center">
				<div>
					<br />
					<button
						on:click={reset}
						class="text {won
							? ''
							: 'opacity-50 hover:opacity-100'} split-light dark:split-dark rounded-xl bg-sky-800/50 px-20 py-2 transition-all hover:scale-105 hover:bg-sky-700/50 active:scale-95"
						>Reset</button
					>
				</div>
			</div>
		</div>
	</div>
</div>
