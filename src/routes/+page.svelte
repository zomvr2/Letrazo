<script>
    import {onMount} from "svelte";
    import { words } from "../data/words.js";
    import { Confetti } from "svelte-confetti";
    import pibble from "$lib/assets/pibble-rave.gif";
    import sadPibble from "$lib/assets/megabademo-emo-dog.gif";
    import {NUMBERS, KEYBOARD_ROWS} from "$lib/CONSTANTS.js";

    import SendSolidIcon from '@iconify-svelte/mynaui/send-solid';
    import DeleteSolidIcon from '@iconify-svelte/mynaui/delete-solid';

    const MAX_LETTERS = 5;
    const MAX_GUESSES = 6;
    let WORD = $state("");
    let CAN_WRITE = $state(true);
    let STATUS = $state("playing");

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
    let lockedLetters = $state([]);

    const handler = (e) => {
        const key = e.key

        if (!CAN_WRITE) return;
        if (key === " ") return;

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
        if (NUMBERS.includes(key[0])) return;
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
                lockedLetters.push(guess[j]);
            }
        }

        for (let k = 0; k < MAX_GUESSES; k++) {
            if (currentCheck[k] === 1) guessed++;
        }

        if (guessed === MAX_LETTERS) win();
        if (currentRow === MAX_GUESSES - 1) loss();

        boardState[currentRow] = currentCheck;
    }

    const win = () => {
        CAN_WRITE = false;
        STATUS = "win"
    }

    const loss = () => {
        CAN_WRITE = false;
        STATUS = "loss"
    }

    let getRandomWord = () => {
        resetGame()
        WORD = words[Math.floor(Math.random() * words.length)].toUpperCase();
        console.log(WORD);
    }

    let resetGame = () => {
        STATUS = "playing"
        CAN_WRITE = true;
        currentRow = 0;
        currentCol = 0;
        guess = "";
        board = Array.from({ length: MAX_GUESSES }, () =>
                Array.from({ length: MAX_LETTERS }, () => "")
            );
        boardState = Array.from({ length: MAX_GUESSES }, () =>
                Array.from({ length: MAX_LETTERS }, () => 0)
            );
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

<style>
    @keyframes pop {
        0% { transform: scale(1); }
        50% { transform: scale(1.1); }
        100% { transform: scale(1); }
    }

    .animate-pop {
        animation: pop 0.15s ease-in-out;
    }
</style>

<svelte:head>
    <title>Letrazo | Adivina la palabra oculta en español</title>
    <meta name="description" content="Juega Letrazo, el divertido reto de adivinar la palabra oculta de 5 letras en 6 intentos. ¡Ejercita tu mente y pon a prueba tu vocabulario gratis!" />
    <meta name="keywords" content="letrazo, juego de palabras, wordle español, adivinar palabras, rompecabezas, acertijo, juego mental" />
    <meta name="author" content="benjamonsh" />

    <meta name="theme-color" content="#f9a8d4" />

    <meta property="og:type" content="website" />
    <meta property="og:title" content="Letrazo - El reto de la palabra diaria" />
    <meta property="og:description" content="¿Podrás adivinar la palabra de 5 letras? Pon a prueba tu mente con Letrazo." />

    <meta name="twitter:title" content="Letrazo | Adivina la palabra oculta" />
    <meta name="twitter:description" content="Adivina la palabra de 5 letras en 6 intentos. ¡Juega gratis ahora!" />
</svelte:head>

<main class="container min-h-screen mx-auto md:max-w-1/2 py-4 px-1 md:p-4 flex flex-col gap-4 items-center justify-center">
    {#if STATUS === "win"}
        <div style="
             position: fixed;
             top: -50px;
             left: 0;
             height: 100vh;
             width: 100vw;
             display: flex;
             justify-content: center;
             overflow: hidden;
             pointer-events: none;
             z-index: 100;"
        >
            <Confetti x={[-5, 5]} y={[0, 0.1]} delay={[100, 2000]} infinite duration=5000 amount=200 fallDistance="100vh" />
        </div>

        <div
                class="
        fixed z-50
        left-5 right-5 bottom-5
        md:left-auto md:right-5 md:bottom-5

        w-auto md:w-80 lg:w-96

        flex flex-col
        bg-gray-100
        border border-gray-200
        rounded-3xl
        overflow-hidden

        shadow-xl shadow-black/10
        backdrop-blur-sm
    "
        >
            <img
                    src={pibble}
                    class="
            w-full
            h-72
            object-cover
        "
                    alt="Rave dancing pibble"
            />

            <div class="px-6 py-4">
                <p class="font-black text-xl text-gray-800">¡Acertaste! 🎉</p>

                <p class="text-gray-600">
                    La palabra es <span class="font-semibold text-gray-800">{WORD}</span>
                </p>

                <button
                        onclick={getRandomWord}
                        class="
                bg-pink-300
                text-white
                p-2.5
                rounded-xl
                mt-4
                cursor-pointer
                w-full
                font-semibold
                transition-all
                hover:bg-pink-400
                active:scale-95
            "
                >
                    Nueva palabra
                </button>
            </div>
        </div>
    {:else if STATUS === "loss"}
        <div
                class="
        fixed z-50
        left-5 right-5 bottom-5
        md:left-auto md:right-5 md:bottom-5

        w-auto md:w-80 lg:w-96

        flex flex-col
        bg-gray-100
        border border-gray-200
        rounded-3xl
        overflow-hidden

        shadow-xl shadow-black/10
        backdrop-blur-sm
    "
        >
            <img
                    src={sadPibble}
                    class="
            w-full
            h-72
            object-cover
            object-top
        "
                    alt="Rave dancing pibble"
            />

            <div class="px-6 py-4">
                <p class="font-black text-xl text-gray-800">¡Perdiste!</p>

                <p class="text-gray-600">
                    La palabra es <span class="font-semibold text-gray-800">{WORD}</span>
                </p>

                <button
                        onclick={getRandomWord}
                        class="
                bg-pink-300
                text-white
                p-2.5
                rounded-xl
                mt-4
                cursor-pointer
                w-full
                font-semibold
                transition-all
                hover:bg-pink-400
                active:scale-95
            "
                >
                    Nueva palabra
                </button>
            </div>
        </div>
    {/if}

    <div class="flex flex-col items-center justify-center mb-3">
        <h1 class="font-bold text-xl">LETRAZO</h1>
        <p>Adivina la palabra de 5 letras.</p>

        <div class="flex flex-col items-start gap-1 mt-3">
            <div class="flex flex-row items-center gap-2">
                <div class="bg-pink-300 w-5 aspect-square rounded-3xl"></div>
                <p class="font-light text-sm">Representa una letra correcta en su lugar correcto.</p>
            </div>

            <div class="flex flex-row items-center gap-2">
                <div class="bg-yellow-200 w-5 aspect-square rounded-3xl"></div>
                <p class="font-light text-sm">Representa una letra correcta en un lugar incorrecto.</p>
            </div>

            <div class="flex flex-row items-center gap-2">
                <div class="bg-gray-500 w-5 aspect-square rounded-3xl"></div>
                <p class="font-light text-sm">Representa una letra incorrecta.</p>
            </div>
        </div>
    </div>

    {#each board as row, rowIndex}
        <div class="flex flex-row gap-2 items-center justify-center">
            {#each row as col, colIndex}
                {@const status = boardState[rowIndex][colIndex]}
                <div class="flex w-16 h-16 items-center justify-center {getStatusClass(status)} transition-colors duration-100 ease-in-out rounded-3xl select-none {col !== '' && rowIndex === currentRow ? 'animate-pop' : ''}">
                    <p class="text-4xl leading-none {rowIndex === currentRow ? "font-normal" : "font-semibold"}">{col}</p>
                </div>
            {/each}
        </div>
    {/each}

    {#if CAN_WRITE}
        <div class="w-full flex justify-center px-0.5 select-none md:hidden">
            <div class="w-full max-w-2xl flex flex-col gap-2">
                {#each KEYBOARD_ROWS as row}
                    <div class="flex justify-center gap-1">
                        {#each row as key}
                            <button
                                    type="button"
                                    aria-label={key === "Backspace" ? "Borrar" : key}
                                    onclick={() => handler({key: key})}
                                    class="
                            h-11
                            rounded-xl sm:rounded-2xl
                            font-bold text-sm sm:text-base
                            shadow-sm shadow-blue-900/20
                            transition-all duration-150
                            active:scale-95
                            active:shadow-none
                            cursor-pointer
                            flex items-center justify-center
                            w-full
                            {lockedLetters.includes(key) ? "text-gray-300" : "text-black"}
                            bg-white
                            hover:bg-gray-200
                        "
                            >
                                {#if key === "Backspace"}
                                    <DeleteSolidIcon height="1.2rem" />
                                {:else if key === "Enter"}
                                    <SendSolidIcon height="1.2rem" />
                                {:else}
                                    {key}
                                {/if}
                            </button>
                        {/each}
                    </div>
                {/each}
            </div>
        </div>
    {/if}

</main>
