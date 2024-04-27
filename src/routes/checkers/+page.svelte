<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import { ChevronLeft, Crown } from 'lucide-svelte';
	import { scale, fade } from 'svelte/transition';

	export type Tile = 'empty' | 'red' | 'blue' | 'red-king' | 'blue-king' | 'move-preview';
	type Board = Array<Array<Tile>>;

	let player: 'red' | 'blue' = 'red';

	function newBoard() {
		let board: Board = [];

		for (let i = 0; i < 8; i++) {
			let column: Array<Tile> = [];
			for (let j = 0; j < 8; j++) {
				column.push('empty');
			}
			board.push(column);
		}

		let blue: Array<Array<number>> = [
			[1, 0],
			[3, 0],
			[5, 0],
			[7, 0],
			[0, 1],
			[2, 1],
			[4, 1],
			[6, 1],
			[1, 2],
			[3, 2],
			[5, 2],
			[7, 2]
		];
		let red: Array<Array<number>> = [
			[0, 5],
			[2, 5],
			[4, 5],
			[6, 5],
			[1, 6],
			[3, 6],
			[5, 6],
			[7, 6],
			[0, 7],
			[2, 7],
			[4, 7],
			[6, 7]
		];

		for (const piece of blue) {
			board[piece[0]][piece[1]] = 'blue';
		}
		for (const piece of red) {
			board[piece[0]][piece[1]] = 'red';
		}

		return board;
	}

	export function isPiece(piece: string | Tile) {
		return ['red', 'blue', 'red-king', 'blue-king'].includes(`${piece}`);
	}

	export function isRed(piece: string | Tile) {
		return ['red', 'red-king'].includes(piece);
	}

	export function isBlue(piece: string | Tile) {
		return ['blue', 'blue-king'].includes(piece);
	}

	export function isKing(piece: string | Tile) {
		return ['blue-king', 'red-king'].includes(piece);
	}

	function clearPreviews() {
		for (let x = 0; x < 8; x++) {
			for (let y = 0; y < 8; y++) {
				if (board[x][y] == 'move-preview') {
					board[x][y] = 'empty';
				}
			}
		}
	}

	function compareArrays(array1: Array<any>, array2: Array<any>) {
		if (array1.length !== array2.length) {
			return false;
		}
		return array1.every((value, index) => value === array2[index]);
	}

	let lastClicked: any;
	function click(pos: any) {
		let { x, y } = pos;

		let piece: Tile = board[x][y];
		console.log(piece);

		if (
			(isBlue(piece) && player == 'blue') ||
			(isRed(piece) && player == 'red') ||
			isPiece(piece) == false
		) {
			if (isPiece(piece)) {
				clearPreviews();
				showMoves(pos);
			} else if (piece == 'move-preview') {
				move(pos);
				clearPreviews();
			} else {
				clearPreviews();
			}
			lastClicked = { x, y };
		} else {
			clearPreviews();
		}
	}

	function checkPromotion() {
		for (let x = 0; x < 8; x++) {
			for (let y = 0; y < 8; y++) {
				let piece = board[x][y];
				if (y === 0 && isRed(piece)) {
					board[x][y] = 'red-king';
				} else if (y === 7 && isBlue(piece)) {
					board[x][y] = 'blue-king';
				}
			}
		}
	}

	function showMoves(pos: any) {
		let { x, y } = pos;

		// Define the possible moves for a piece
		let redMoves = [
			[x + 1, y - 1], // Up-right
			[x - 1, y - 1] // Up-left
		];
		let blueMoves = [
			[x + 1, y + 1], // Down-right
			[x - 1, y + 1] // Down-left
		];
		let redJumpMoves = [
			[x + 2, y - 2], // Up-right
			[x - 2, y - 2] // Up-left
		];
		let blueJumpMoves = [
			[x + 2, y + 2], // Down-right
			[x - 2, y - 2] // Down-left
		];
		let allMoves = [
			[x + 1, y + 1], // Down-right
			[x + 1, y - 1], // Up-right
			[x - 1, y + 1], // Down-left
			[x - 1, y - 1] // Up-left
		];
		let allJumpMoves = [
			[x + 2, y + 2], // Down-right
			[x + 2, y - 2], // Up-right
			[x - 2, y + 2], // Down-left
			[x - 2, y - 2] // Up-left
		];

		let piece = board[pos.x][pos.y];
		let team: string,
			moves: Array<Array<number>> = allMoves,
			jumpMoves: Array<Array<number>> = allJumpMoves;

		if (isKing(piece)) {
			moves = allMoves;
			jumpMoves = allJumpMoves;
		} else {
			if (isRed(piece)) {
				team = 'red';
				jumpMoves = redJumpMoves;
				moves = redMoves;
			} else if (isBlue(piece)) {
				moves = blueMoves;
				jumpMoves = blueJumpMoves;
				team = 'blue';
			}
		}

		// Filter out moves that are out of bounds or not empty
		let validNormalMoves = moves.filter(([newX, newY]) => {
			return newX >= 0 && newX < 8 && newY >= 0 && newY < 8 && board[newX][newY] === 'empty';
		});

		// Filter out jump moves that are out of bounds, not empty, or do not capture an opponent's piece
		let validJumpMoves = jumpMoves.filter(([newX, newY]) => {
			let jump: Tile | string;
			if (board[x + (newX - x) / 2] == undefined) {
				jump = 'empty';
			} else {
				if (board[x + (newX - x) / 2][y + (newY - y) / 2] == undefined) {
					jump = 'empty';
				} else {
					jump = board[x + (newX - x) / 2][y + (newY - y) / 2];
				}
			}

			return (
				newX >= 0 &&
				newX < 8 &&
				newY >= 0 &&
				newY < 8 &&
				((team === 'blue' && isBlue(board[newX][newY]) === false) ||
					(team === 'red' && isRed(board[newX][newY]) === false)) &&
				((team === 'blue' && isRed(jump) === true) || (team === 'red' && isBlue(jump) === true)) &&
				isPiece(board[newX][newY]) == false
			);
		});

		for (const [newX, newY] of validNormalMoves) {
			board[newX][newY] = 'move-preview';
		}
		for (const [newX, newY] of validJumpMoves) {
			board[newX][newY] = 'move-preview';
		}
	}

	function switchPlayer() {
		if (player == 'red') {
			player = 'blue';
		} else {
			player = 'red';
		}
	}

	let won: false | 'red' | 'blue' = false;
	function checkWin() {
		let blues = 0,
			reds = 0;

		for (let x = 0; x < 8; x++) {
			for (let y = 0; y < 8; y++) {
				let piece = board[x][y];
				if (isBlue(piece)) {
					blues++;
				}
				if (isRed(piece)) {
					reds++;
				}
			}
		}

		if (reds == 0) {
			won = 'blue';
		} else if (blues == 0) {
			won = 'red';
		} else {
			won = false;
		}
	}

	function move(next: any) {
		let prev: any = lastClicked;
		let piece = board[prev.x][prev.y];

		// Check if the move is a jump move
		let isJumpMove = Math.abs(next.x - prev.x) === 2 && Math.abs(next.y - prev.y) === 2;

		// If it's a jump move, remove the opponent's piece from the board
		if (isJumpMove) {
			let midX = (prev.x + next.x) / 2;
			let midY = (prev.y + next.y) / 2;
			board[midX][midY] = 'empty';
		}

		// Move the piece to the new position
		board[prev.x][prev.y] = 'empty';
		board[next.x][next.y] = piece;

		checkPromotion();
		checkWin();
		console.log(prev, next, piece);
		switchPlayer();
	}

	function reset() {
		board = newBoard();
		player = 'red';
		won = false;
	}

	let board = newBoard();
</script>

<a href="/" class="absolute ml-0">
	<ChevronLeft size="64" class="p-3 opacity-90" />
</a>
<div
	class="flex h-screen w-screen justify-center transition-all
	{player == 'blue' ? 'bg-blue-900' : 'bg-red-900'} 
	{won == 'blue' ? 'bg-blue-900' : ''} {won == 'red' ? 'bg-red-900' : ''}"
>
	<div class="my-auto aspect-square max-w-screen-md flex-1 overflow-auto p-5">
		<!--how can i align this horizonally? -->
		<div class="grid grid-cols-8 rounded-2xl border-4 border-foreground/20">
			{#each board as row, rowIndex}
				<div class="flex flex-col">
					{#each row as piece, columnIndex}
						<button
							on:click={() => {
								click({
									x: rowIndex,
									y: columnIndex
								});
							}}
							class="aspect-square
              {(rowIndex % 2 === 0 && columnIndex % 2 === 0) ||
							(rowIndex % 2 !== 0 && columnIndex % 2 !== 0)
								? 'bg-background'
								: 'bg-foreground'}
                {compareArrays([0, 0], [rowIndex, columnIndex]) ? 'rounded-tl-xl' : ''}
                {compareArrays([0, 7], [rowIndex, columnIndex]) ? 'rounded-bl-xl' : ''}
                {compareArrays([7, 7], [rowIndex, columnIndex]) ? 'rounded-br-xl' : ''}
                {compareArrays([7, 0], [rowIndex, columnIndex]) ? 'rounded-tr-xl' : ''}"
						>
							<div
								class="piece aspect-square [&>*]:transition-all [&>*]:hover:ring-4 [&>*]:hover:ring-zinc-500/50"
							>
								{#if isRed(piece)}
									<div class="h-full w-full rounded-full bg-red-500">
										<div class="h-full w-full scale-75 rounded-full bg-red-600/60">
											{#if isKing(piece)}
												<Crown class="h-full w-full p-[1vw] text-background/20" />
											{/if}
										</div>
									</div>
								{/if}
								{#if isBlue(piece)}
									<div class="h-full w-full rounded-full bg-cyan-600">
										<div class="h-full w-full scale-75 rounded-full bg-cyan-700/60">
											{#if isKing(piece)}
												<Crown class="h-full w-full p-[1vw] text-background/20" />
											{/if}
										</div>
									</div>
								{/if}
								{#if piece == 'move-preview'}
									<button
										transition:scale
										class="h-full w-full scale-50 rounded-full bg-zinc-500/50"
									/>
								{/if}
							</div>
						</button>
					{/each}
				</div>
			{/each}
		</div>
	</div>
</div>

<!-- WON OVERLAY -->
{#if won}
	<div
		transition:fade
		class="align-center absolute top-0 z-10 flex h-screen w-screen justify-center bg-background/60 backdrop-blur
	{won ? '' : ''}"
	>
		<div
			class="prose prose-invert my-auto flex flex-col rounded-2xl bg-zinc-800 p-10 shadow-xl"
			transition:scale
		>
			<h1>{`${won}`.substring(0, 1).toUpperCase() + `${won}`.substring(1)} won!</h1>

			<Button on:click={reset}>Reset</Button>
		</div>
	</div>
{/if}

<style>
	.piece {
		padding: min(1vw, 5px);
	}
</style>
