<script>
    import {onMount} from "svelte";
    import { words } from "../data/words.js";

    const MAX_LETTERS = 5;
    const MAX_GUESSES = 6;
    let WORD = $state("");
    let CAN_WRITE = $state(true);

    let board = $state(
        Array.from({ length: MAX_GUESSES }, () =>
            Array.from({ length: MAX_LETTERS }, () => "")
        )
    );

    let boardState = $state(
        Array.from({ length: MAX_GUESSES }, () =>
            Array.from({ length: MAX_LETTERS }, () => 0)
        )
    );

    let currentRow = $state(0);
    let currentCol = $state(0);
    let guess = $state("");

    const handler = (e) => {
        const key = e.key
        const numbers = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"];

        if (!CAN_WRITE) return;
        if (e.keyCode === 32) return;

        if (key === "Enter" && guess.length === MAX_LETTERS) {
            checkGuess();
            guess = "";
            currentCol = 0;
            if (currentRow < MAX_GUESSES) {
                currentRow++;
            }
            return;
        }

        if (key === "Backspace") {
            if (currentCol === 0) return;

            currentCol--;
            board[currentRow][currentCol] = "";

            guess = guess.slice(0, -1);
            return;
        }

        if (key.length > 1) return;
        if (numbers.includes(key[0])) return;
        if (currentCol === MAX_LETTERS) return;

        board[currentRow][currentCol] = key.toUpperCase();

        guess += key.toUpperCase();
        currentCol++;
    };

    let checkGuess = () => {
        let currentCheck = [0, 0, 0, 0, 0]
        let guessed = 0;

        for (let i = 0; i < MAX_LETTERS; i++) {
            if (guess[i] === WORD[i]) {
                currentCheck[i] = 1;
            }
        }

        for (let j = 0; j < MAX_LETTERS; j++) {
            if (currentCheck[j] === 1) continue;

            if (WORD.includes(guess[j])) {
                currentCheck[j] = 2;
            } else {
                currentCheck[j] = 3;
            }
        }

        for (let k = 0; k < MAX_GUESSES; k++) {
            if (currentCheck[k] === 1) guessed++;
        }

        if (guessed === MAX_LETTERS) win();

        boardState[currentRow] = currentCheck;
    }

    const win = () => {
        CAN_WRITE = false;
    }

    let getRandomWord = () => {
        WORD = words[Math.floor(Math.random() * words.length)].toUpperCase();
        console.log(WORD);
    }

    let getStatusClass = (statusNumber) => {
        switch(statusNumber) {
            case 1: return "bg-pink-300 text-white";
            case 2: return "bg-yellow-200 text-white";
            case 3: return "bg-gray-500 text-white";
            default: return "bg-gray-200 text-black";
        }
    };

    onMount(() => {
        document.addEventListener("keydown", (e) => handler(e));
        getRandomWord();

        return () => {
            document.removeEventListener("keydown", handler);
        };
    });
</script>

<main class="container min-h-screen mx-auto max-w-1/2 p-4 flex flex-col gap-4 items-center justify-center">
    <div class="flex flex-col items-center justify-center mb-3">
        <h1 class="font-bold text-xl">Benjadle</h1>
        <p>Adivina la palabra de 5 letras.</p>

        <div class="flex flex-col items-start gap-1 mt-3">
            <div class="flex flex-row items-center gap-2">
                <div class="bg-pink-300 w-5 aspect-square rounded-3xl"></div>
                <p class="font-light text-sm">Representa a una letra correcta en su lugar correcto.</p>
            </div>

            <div class="flex flex-row items-center gap-2">
                <div class="bg-yellow-200 w-5 aspect-square rounded-3xl"></div>
                <p class="font-light text-sm">Representa a una letra correcta en un lugar incorrecto.</p>
            </div>

            <div class="flex flex-row items-center gap-2">
                <div class="bg-gray-500 w-5 aspect-square rounded-3xl"></div>
                <p class="font-light text-sm">Representa a una letra incorrecta.</p>
            </div>
        </div>
    </div>
    {#each board as row, rowIndex}
        <div class="flex flex-row gap-2 items-center justify-center">
            {#each row as col, colIndex}
                {@const status = boardState[rowIndex][colIndex]}
                <div class="flex w-16 h-16 items-center justify-center {getStatusClass(status)} transition-all duration-100 ease-in-out rounded-3xl">
                    <p class="text-4xl">{col}</p>
                </div>
            {/each}
        </div>
    {/each}
</main>
