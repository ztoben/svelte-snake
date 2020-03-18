<script>
	import Snek from './Snek.svelte';
	import Food from './Food.svelte';
    import {BOARD_SIZE, DOWN, EMPTY_CELL, FOOD_CELL, HIGH_SCORES, LEFT, RIGHT, SNEK_CELL, UP} from './constants';

	let highScores;
	let gameInterval;
	let score;
	let direction;
	let previousDirection;
	let board;
	let headPosition;

	let snek;
	try {
		highScores = JSON.parse(localStorage.getItem(HIGH_SCORES)) || [0];
	} catch (err) {
		highScores = [0];
	}

	function addSnek() {
		let pos = Math.floor(BOARD_SIZE / 2);

		board[pos][pos] = SNEK_CELL;
		headPosition = [pos, pos];
		snek.push([pos, pos]);
	}

	function addFood() {
		let possiblePositions = [];

		board.forEach((row, x) => {
			row.forEach((column, y) => {
				if (board[x][y] === EMPTY_CELL) possiblePositions.push([x, y]);
			});
		});

		const [x, y] = possiblePositions[Math.floor(Math.random()*possiblePositions.length)];

		board[x][y] = FOOD_CELL;
	}

	function eatFood(position) {
		const [x, y] = position;

		score += 1;
		headPosition = position;
		board[x][y] = SNEK_CELL;
		snek.push([x, y]);

		increaseIntervalSpeed();
		addFood();
	}

	function updatePosition(position) {
		const [x, y] = position;

		board[x][y] = SNEK_CELL;
		headPosition = position;
		snek.push([x, y]);

		const [removeX, removeY] = snek[0];
		board[removeX][removeY] = EMPTY_CELL;
		snek.splice(0, 1);
	}

	function newGame() {
		score = 0;
		direction = null;
		previousDirection = null;
		board = [...Array(BOARD_SIZE)].map(() => Array(BOARD_SIZE).fill(0));
		headPosition = [0, 0];
		snek = [];

		addSnek();
		addFood();

		gameInterval = setInterval(move, 250);
	}

	function increaseIntervalSpeed() {
		clearInterval(gameInterval);
		let speed = Math.max(250 - (score * 4), 50);

		gameInterval = setInterval(move, speed);
	}

	function gameOver() {
		highScores = highScores.concat(score);

		clearInterval(gameInterval);
		newGame();
	}

	function checkForCollision(position) {
		const [x, y] = position;

		if (x < 0 || x === BOARD_SIZE || y < 0 || y === BOARD_SIZE) return gameOver();
		if (board[x][y] === SNEK_CELL) return gameOver();
		if (board[x][y] === FOOD_CELL) return eatFood(position);

		return updatePosition(position);
	}

	function setDirection(newDirection) {
		if (
			direction === null
			|| (newDirection === RIGHT && previousDirection !== LEFT)
			|| (newDirection === LEFT && previousDirection !== RIGHT)
			|| (newDirection === UP && previousDirection !== DOWN)
			|| (newDirection === DOWN && previousDirection !== UP)
		) {
			direction = newDirection;
		}
	}

	function move() {
		const [currX, currY] = headPosition;
		previousDirection = direction;

		switch (direction) {
			case RIGHT:
				return checkForCollision([currX, currY + 1]);
			case LEFT:
				return checkForCollision([currX, currY - 1]);
			case UP:
				return checkForCollision([currX - 1, currY]);
			case DOWN:
				return checkForCollision([currX + 1, currY]);
		}
	}

	window.addEventListener('keydown', function (e) {
		if (e.key === 'ArrowRight' || e.key === 'd') {
			return setDirection(RIGHT);
		} else if (e.key === 'ArrowLeft' || e.key === 'a') {
			return setDirection(LEFT);
		} else if (e.key === 'ArrowUp' || e.key === 'w') {
			return setDirection(UP);
		} else if (e.key === 'ArrowDown' || e.key === 's') {
			return setDirection(DOWN);
		}
	});

	newGame();

	$: highScore = Math.max(...highScores);
	$: try {
		localStorage.setItem(HIGH_SCORES, JSON.stringify(highScores));
	} catch (err) {
		// noop
	}
</script>

<style>
	:root {
		--size: 650px;
		--noOfColumns: 15;
		--rowHeight: calc(var(--size) / var(--noOfColumns));
		--ratioA: 1;
		--ratioB: 1;
	}

	.container {
		width: 100%;
		height: 100%;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
	}

	.header-container {
		width: var(--size);
		display: inline-flex;
		justify-content: space-between;
		margin-bottom: 10px;
	}

	.score {
		width: 200px;
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: row;
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
	}

	.high-score {
		background-color: lightgray;
		color: red;
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
	}

	.game-container {
		padding: 5px;
		box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
		margin: 5px;
	}

	.grid-container {
		height: var(--size);
		width: var(--size);
		position: relative;
		display: grid;
		grid-template-columns: repeat(var(--noOfColumns), 1fr);
		grid-auto-rows: var(--rowHeight);
	}

	.cell {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;
	}
</style>

<div class="container">
	<div class="header-container">
		<wired-card class="score">
			Score:
			{score}
		</wired-card>
		<wired-card class="score high-score">
			High Score:
			{highScore}
		</wired-card>
	</div>

	<wired-card class="game-container" elevation="3">
		<div class="grid-container">
			{#each board as row, outerIndex}
				{#each row as cell, index}
					<div class="cell">
						{#if board[outerIndex][index] === 1}
							<Snek/>
						{:else if board[outerIndex][index] === 2}
							<Food/>
						{/if}
					</div>
				{/each}
			{/each}
		</div>
	</wired-card>
</div>
