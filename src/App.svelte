<script defer>
    import { onMount } from "svelte/internal";
    import startBoard from "./sudoku.json";

    onMount(() => {
        (function () {
            function onChange(event) {
                var reader = new FileReader();
                reader.onload = onReaderLoad;
                reader.readAsText(event.target.files[0]);
            }

            function onReaderLoad(event) {
                loadGame(JSON.parse(event.target.result)["board"]);
            }

            document
                .getElementById("sudoku")
                .addEventListener("change", onChange);
        })();
        displayedCells = addDisplayedCells();
    });

    let gameBoard = [];
    let solvedBoard = [];
    let displayedBoard = [];
    let displayedCells = [];
    let possibleInputsArr = [];
    let keyboardArr = [
        [1, false],
        [2, false],
        [3, false],
        [4, false],
        [5, false],
        [6, false],
        [7, false],
        [8, false],
        [9, false],
    ];
    let currentCell = [0, 0, 0];
    let isFinished = false;
    let solvable = true;
    let currentHint = false;

    function addDisplayedCells() {
        const board = document.getElementById("board");
        let arr = [];
        let group = board.firstElementChild;
        for (let i = 0; i < 9; i++) {
            let cell = group.firstElementChild;
            for (let j = 0; j < 9; j++) {
                if (displayedBoard[i][j]["num"] == null) {
                    arr.push(cell.firstElementChild);
                }
                cell = cell.nextElementSibling;
            }
            group = group.nextElementSibling;
        }
        return arr;
    }

    function loadGame(startBoard) {
        gameBoard = [];
        solvedBoard = [];
        displayedBoard = [];
        possibleInputsArr = [];
        displayedCells = [];
        currentCell = [0, 0, 0];
        isFinished = false;
        currentHint = false;

        startBoard.forEach((row, index) => {
            gameBoard.push([]);
            solvedBoard.push([]);
            row.forEach((el) => {
                gameBoard[index].push(el);
                solvedBoard[index].push(el);
            });
        });
        let temp = solveBoard(solvedBoard);
        if (temp) {
            solvable = true;
            isFinished = false;
            solvedBoard = solveBoard(solvedBoard); //solved board
            keyboardArr.forEach((key) => {
                key[1] = false;
            });
            keyboardArr[0][0] = 1; // force svelte to refresh displayed keyboard
        } else {
            solvable = false;
            isFinished = true;
            keyboardArr.forEach((key) => {
                key[1] = true;
            });
            keyboardArr[0][0] = 1; // force svelte to refresh displayed keyboard
        }
        console.log(solvable);

        displayedBoard = displayBoard(gameBoard); // displayed board
        possibleInputsArr = possibleInputs(gameBoard); //possible inputs for all cells
        console.log(solvedBoard);
    }

    function isValid(board, row, col, num) {
        for (let i = 0; i < 9; i++) {
            const m = 3 * Math.floor(row / 3) + Math.floor(i / 3);
            const n = 3 * Math.floor(col / 3) + (i % 3);
            if (
                board[row][i] == num ||
                board[i][col] == num ||
                board[m][n] == num
            ) {
                return false;
            }
        }
        return true;
    }

    function solveBoard(data) {
        for (let i = 0; i < 9; i++) {
            for (let j = 0; j < 9; j++) {
                if (data[i][j] == null) {
                    for (let k = 1; k <= 9; k++) {
                        if (isValid(data, i, j, k)) {
                            data[i][j] = k;
                            if (solveBoard(data)) {
                                return data;
                            } else {
                                data[i][j] = null;
                            }
                        }
                    }
                    return false;
                }
            }
        }
        return data;
    }

    function displayBoard(gameBoard) {
        let board = [[], [], [], [], [], [], [], [], []];
        let counter = 0;
        let cells = 0;
        let rows = 0;
        gameBoard.forEach((row) => {
            row.forEach((el) => {
                if (counter % 9 == 0) {
                    cells = Math.floor(rows / 3) * 3;
                } else if (counter % 3 == 0) {
                    cells += 1;
                }
                board[cells].push({
                    num: el,
                    row: rows,
                    col: counter % 9,
                });
                counter += 1;
            });
            rows += 1;
        });
        return board;
    }
    function handleInput(row, col) {
        if (JSON.stringify(gameBoard) == JSON.stringify(solvedBoard)) {
            isFinished = true;
            solvable = false;
        }
        checkKeyboardAvalibity();
        if (gameBoard[row][col] == solvedBoard[row][col]) {
            console.log(gameBoard);
        } else {
        }
    }

    function possibleInputs(board) {
        let possibleInputsArr = [];
        function cellPossibleInputs(row, col) {
            let possibilites = [];
            let valid = false;
            for (let num = 1; num <= 9; num++) {
                for (let i = 0; i < 9; i++) {
                    const m = 3 * Math.floor(row / 3) + Math.floor(i / 3);
                    const n = 3 * Math.floor(col / 3) + (i % 3);
                    if (
                        board[row][i] == num ||
                        board[i][col] == num ||
                        board[m][n] == num
                    ) {
                        valid = false;
                        break;
                    }
                    valid = true;
                }
                if (valid) {
                    possibilites.push(num);
                }
            }
            return possibilites;
        }
        board.forEach((row, i) => {
            possibleInputsArr.push([]);
            row.forEach((cell, j) => {
                if (cell == null) {
                    possibleInputsArr[i].push(cellPossibleInputs(i, j));
                } else {
                    possibleInputsArr[i].push([cell]);
                }
            });
        });
        return possibleInputsArr;
    }

    function requestSolve() {
        if (solvable) {
            displayedBoard = displayBoard(solvedBoard);
        } else {
            displayedBoard = displayBoard(startBoard);
        }
        isFinished = true;
    }

    function requestRestart() {
        document.location.reload(); // gdybym tylko zerował plansze funckja onMount nie wywoływałaby się
        // a przez to arr z wyświetlnymi komórkami nie mógłby się odświeżyć,
        // ponieważ jeżeli funckję wywoływałbym poza onMount, to elementy nie miałyby czasu
        // się załadować
    }

    function setCurrentCell() {
        let active = document.activeElement;
        let index = displayedCells.indexOf(active);
        currentCell = [index, 0, 0];
    }
    function updateCurrentCell(row, col) {
        currentCell[1] = row;
        currentCell[2] = col;
    }

    function checkKeyboardAvalibity() {
        keyboardArr.forEach((key) => {
            let counter = 0;
            gameBoard.forEach((row) => {
                row.forEach((cell) => {
                    if (cell == key[0]) {
                        counter += 1;
                    }
                });
            });
            if (counter >= 9) {
                key[1] = true;
            } else {
                key[1] = false;
            }
            keyboardArr[0][0] = 1; // force svelte to refresh displayed keyboard
        });
    }

    function movePointer(row, col, event) {
        event = event ? event : window.event;
        var keyCode = event.code;
        if (keyCode == "ArrowRight") {
            if (currentCell[0] == displayedCells.length - 1) {
                currentCell[0] = 0;
            } else {
                currentCell[0] += 1;
            }
            displayedCells[currentCell[0]].focus();
            currentHint = false;
        }
        if (keyCode == "ArrowLeft") {
            if (currentCell[0] == 0) {
                currentCell[0] = displayedCells.length - 1;
            } else {
                currentCell[0] -= 1;
            }
            displayedCells[currentCell[0]].focus();
            currentHint = false;
        }
        if (keyCode == "Backspace") {
            if (!currentHint) {
                displayedCells[currentCell[0]].value = null;
                gameBoard[row][col] = null;
                checkKeyboardAvalibity();
            }
        }
        if (keyCode == "Enter") {
            endHint();
        }
    }

    function handleKeyboardClick(num) {
        if (!currentHint) {
            displayedCells[currentCell[0]].focus();
            displayedCells[currentCell[0]].value = num;
            gameBoard[currentCell[1]][currentCell[2]] = num;
            checkKeyboardAvalibity();
            if (JSON.stringify(gameBoard) == JSON.stringify(solvedBoard)) {
                isFinished = true;
            }
        } else {
            displayedCells[currentCell[0]].previousSibling.focus();
            displayedCells[currentCell[0]].previousSibling.value += num + " ";
        }
    }

    function requestDelete() {
        if (!currentHint) {
            displayedCells[currentCell[0]].focus();
            displayedCells[currentCell[0]].value = null;
            gameBoard[currentCell[1]][currentCell[2]] = null;
            checkKeyboardAvalibity();
            if (JSON.stringify(gameBoard) == JSON.stringify(solvedBoard)) {
                isFinished = true;
            }
        } else {
            displayedCells[currentCell[0]].previousSibling.focus();
            let temp = displayedCells[currentCell[0]].previousSibling.value;
            displayedCells[currentCell[0]].previousSibling.value = temp.slice(
                0,
                temp.length - 2
            );
        }
    }

    function requestHint() {
        if (!currentHint) {
            displayedCells[currentCell[0]].focus();
            let hint = document.createElement("input");
            hint.setAttribute(
                "style",
                "width: 100%; border: none; font-size: .5em; outline: none; position: absolute; top: -4px; text-align: center; z-index: -1;"
            );
            displayedCells[currentCell[0]].before(hint);
            displayedCells[currentCell[0]].previousSibling.focus();
            currentHint = true;
        } else {
            endHint();
            currentHint = false;
        }
    }
    function endHint() {
        displayedCells[currentCell[0]].focus();
    }

    loadGame(startBoard["board"]);
</script>

<h1 class="title">Sudoku</h1>
<input type="file" id="sudoku" />
<div id="board" class="grid">
    {#each displayedBoard as group}
        <div class="group grid border">
            {#each group as cell}
                <div
                    class="cell border"
                    on:keydown={() => movePointer(cell.row, cell.col)}
                >
                    {#if cell.num != null}
                        <input type="number" bind:value={cell.num} disabled />
                    {/if}
                    {#if cell.num == null}
                        <input
                            type="number"
                            disabled={solvable ? false : true}
                            bind:value={gameBoard[cell.row][cell.col]}
                            on:click={setCurrentCell}
                            on:input={() => handleInput(cell.row, cell.col)}
                            class={solvedBoard[cell.row][cell.col] ==
                            gameBoard[cell.row][cell.col]
                                ? "correct"
                                : "wrong"}
                            on:blur={() =>
                                updateCurrentCell(cell.row, cell.col)}
                        />
                    {/if}
                </div>
            {/each}
        </div>
    {/each}
</div>
<div class="solved">
    {#if isFinished}
        {#if solvable}
            <h1>Solved!</h1>
        {:else}
            <h1>Unsolvable!</h1>
        {/if}
    {/if}
</div>
<button
    disabled={isFinished ? true : false}
    class={isFinished ? "hint used" : "hint"}
    on:click={requestHint}>Hint</button
>
<div class="keyboard grid">
    {#each keyboardArr as key}
        <button
            class={key[1] || isFinished ? "used" : ""}
            on:click={() => handleKeyboardClick(key[0])}
            disabled={key[1] || isFinished ? true : false}>{key[0]}</button
        >
    {/each}
</div>
<button
    disabled={isFinished ? true : false}
    class={isFinished ? "delete used" : "delete"}
    on:click={requestDelete}>-</button
> <br />
<button
    disabled={solvable ? false : true}
    class={solvable ? "solve" : "solve used"}
    on:click={requestSolve}
>
    Solve
</button>
<button class="restart" on:click={requestRestart}> Restart </button>

<div class="instructions">
    <h2>Quick instructions</h2>
    <ul>
        <li>You can use both your own keyboard and screen keyboard.</li>
        <ul>
            <li>
                The color of the number will tell you whether it is correct or
                wrong.
            </li>
            <li>
                On the scrren keyboard, the button will grey out when its number
                is used.
            </li>
        </ul>
        <li>You are able to move your cursor with left and right arrow.</li>
        <ul>
            <li>
                To do so, you first need to click any editable cell on the
                board.
            </li>
            <li>After that you can feel free to use arrows to move.</li>
        </ul>
        <li>
            The 'Hint' button can make additional place in the top of the cell
            for you to write your predictions.
        </li>
        <ul>
            <li>
                Clicking 'Hint' button on the cell that already has it will
                cause to remove previous hint and create new one.
            </li>
        </ul>
        <li>
            To remove number in the cell you can use both Backspace on your
            keyboard and '-' button on the screen.
        </li>
        <li>You can use 'Solve' button to solve current board.</li>
        <li>'Restart' button will cause the game to reload.</li>
        <li>
            You can also load your own sudoku with JSON files with the following
            format: <br />
            <pre>
"board": [
  [null, 6, 3, null, 4, 7, 2, 9, 8],
  [8, null, null, null, 3, null, 4, null, null],
  [null, null, 2, null, null, null, 7, 1, null],
  [3, 4, null, 8, 6, 9, 1, 7, 2],
  [2, null, 6, null, 5, 3, 8, null, 9],
  [null, 8, null, 2, 1, null, 3, null, 6],
  [6, null, null, null, null, null, 9, 3, null],
  [7, null, 4, 3, null, null, null, 8, null],
  [5, null, 8, 9, null, null, 6, null, null]
]
          </pre>
        </li>
        <ul>
            <li>
                Please note that if you load unsolvable sudoku board you won't
                be able to type anything.
            </li>
        </ul>
    </ul>
</div>

<style>
  *{margin:0px;padding:0px;}
    .grid {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        grid-template-rows: repeat(3, 1fr);
    }
    .border {
        border: 1px solid black;
    }
    .title {
        font-size: 4em;
        margin-bottom: 30px;
        font-weight: 100;
    }
    div#board {
        margin: auto;
        width: 500px;
        height: 500px;
        gap: 4px;
        border-collapse: collapse;
    }
    div.cell {
        position: relative;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
    }
    div.cell input {
        background-color: transparent;
        border: none;
        width: 100%;
        text-align: center;
        outline: none;
        font-size: 1.2em;
        font-weight: 600;
    }
    div.cell input:disabled {
      color: black;
    }
    .correct {
        color: green;
    }
    .wrong {
        color: red;
    }
    div.keyboard {
        width: 162px;
        height: 162px;
        margin: auto;
    }
    div.keyboard button,
    button.delete,
    button.hint {
        background-color: #212121;
        color: #eee;
        font-weight: bold;
        font-size: 1.5em;
        margin: 1px;
        height: 1fr;
    }
    button.hint {
        font-size: 1em;
        width: 52px;
    }
    button.delete {
        height: 26px;
        width: 162px;
    }
    div.keyboard button.used,
    .used {
        opacity: 0.5;
    }
    div.solved h1{
        margin-top: 10px;
        font-size: 2em;
        color: #212121;
    }
    button.solve,
    button.restart {
        margin-top: 10px;
        background-color: #212121;
        color: #eee;
        padding: 3px;
        width: 60px;
    }
    #sudoku {
        margin-bottom: 10px;
    }

    input::-webkit-outer-spin-button,
    input::-webkit-inner-spin-button {
        -webkit-appearance: none;
        margin: 0;
    }
    .instructions {
        margin-top: 40px;
        text-align: initial;
    }
    .instructions h2 {
        font-size: 2em;
        margin-bottom: 10px;
    }
    .instructions ul {
        margin-left: 30px;
        list-style: inherit;
    }

    /* Firefox */
    input[type="number"] {
        -moz-appearance: textfield;
    }
    @media print {
        * {
            visibility: hidden;
        }
        #board,
        #board * {
            visibility: visible;
        }
    }
</style>
