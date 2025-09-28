<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Bodycam Bingo</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
      <!-- Load Vue.js -->
      <script src="https://unpkg.com/vue@3/dist/vue.global.prod.js"></script>
    <script>
      tailwind.config = {
        theme: {
          extend: {
            colors: {
              "primary-dark": "#1e293b", // Slate-800
              "accent-red": "#ef4444", // Red-500
              "accent-blue": "#3b82f6", // Blue-500
              "marked-green": "#10b981", // Emerald-500
            },
            fontFamily: {
              sans: ["Inter", "sans-serif"],
            },
          },
        },
      };
    </script>
    <style>
      /* Custom font import */
      @import url("https://fonts.googleapis.com/css2?family=Bungee+Inline&family=Inter:wght@400;700;900&display=swap");

      .title-font {
        font-family: "Bungee Inline", cursive;
        letter-spacing: 2px;
      }

      /* --- BACKGROUND AND OVERLAY STYLES --- */
      body {
        /* Using the Mount Everest image provided by the user */
        background-image: url("https://cdn.britannica.com/17/83817-050-67C814CD/Mount-Everest.jpg");
        background-size: cover;
        background-position: center;
        background-attachment: fixed;

        min-height: 100vh;
        display: flex;
        flex-direction: column;
        align-items: center;
        padding: 1rem;
      }

      /* Container for the Bingo board and header, giving it a translucent background for readability */
      #main-app-container {
        background-color: rgba(
          0,
          0,
          0,
          0.85
        ); /* Highly transparent dark background */
        border-radius: 1.5rem;
        padding: 1.5rem;
        max-width: 600px;
        width: 100%;
        box-shadow: 0 10px 40px rgba(0, 0, 0, 0.8);
        backdrop-filter: blur(
          4px
        ); /* Slight blur effect on content behind the container */
      }

      /* Bingo Grid layout */
      .bingo-grid {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 4px; /* Small gap between cells */
      }

      /* Ensure the entire grid container scales correctly on mobile */
      #bingo-container {
        width: 100%;
      }
    </style>
  </head>
  <body class="text-white font-sans">
    <div id="app">
      <div id="main-app-container">
        <!-- Header & Title -->
        <header class="text-center mb-6 w-full">
          <h1 class="title-font text-5xl md:text-6xl tracking-wider mb-2">
            <span class="text-accent-red">BODYCAM</span>
            <span class="text-accent-blue">BINGO</span>
          </h1>
          <p
            class="text-xl font-bold transition-all duration-500 h-6"
            :class="win ? 'text-marked-green animate-pulse' : 'text-transparent'"
          >
            {{ win ? '🔥 BINGO! WE HAVE A WINNER! 🔥' : '' }}
          </p>
        </header>

        <!-- Bingo Grid Container -->
        <main id="bingo-container" class="flex-grow flex justify-center">
          <div
            class="bingo-grid border-4 border-primary-dark shadow-2xl rounded-xl aspect-square w-full"
          >
            <div
              v-for="(card, idx) in board"
              :key="idx"
              :class="[
                'flex items-center justify-center text-center p-1 md:p-2 rounded-md transition-all duration-200 cursor-pointer select-none border-2 border-primary-dark shadow-inner',
                card.marked ? 'bg-marked-green text-gray-900 font-extrabold transform scale-95' : 'bg-primary-dark hover:bg-slate-700 text-white font-medium',
                card.text === 'FREE SPACE' ? 'flex-col gap-1' : ''
              ]"
              @click="mark(idx)"
            >
              <template v-if="card.text === 'FREE SPACE'">
                <img
                  :src="freeSpaceImg"
                  alt="Fridays with Frank Sheriff Patch / Free Space"
                  class="w-3/4 h-auto max-w-[80px] rounded-lg shadow-lg border-2 border-white mb-1 object-contain"
                  @error="imgError = true"
                  v-if="!imgError"
                />
                <p v-else class="text-xs text-red-400">Error loading patch image. (Check URL)</p>
                <p class="text-xs md:text-sm lg:text-base leading-snug font-black">FREE SPACE</p>
              </template>
              <template v-else>
                <p class="text-xs md:text-sm lg:text-base leading-snug">{{ card.text }}</p>
              </template>
            </div>
          </div>
        </main>

        <!-- Controls -->
        <footer class="mt-8 w-full flex flex-col items-center">
          <button
            class="w-full max-w-xs px-6 py-3 bg-accent-blue hover:bg-blue-600 active:bg-blue-700 text-white font-extrabold rounded-lg shadow-xl transition-all transform hover:scale-[1.02] disabled:opacity-50"
            @click="reset"
          >
            Reset Card
          </button>
          <p class="text-sm text-gray-400 mt-4">Tap any square to mark it.</p>
        </footer>
      </div>
    </div>

    <script type="module">
    const { createApp } = Vue;
    createApp({
      data() {
        return {
          phrases: [
            "I know my rights",
            "Name & badge number?",
            "Call your supervisor",
            "I know a cop",
            "FREE SPACE",
            "I pay your salary",
            "I have a lawyer",
            'Saying "not resisting" while resisting',
            "Ignorant comment about cops shooting people",
          ],
          freeSpaceImg: "https://fridayswithfrank.com/wp-content/uploads/2023/12/Patch.jpg",
          winCombos: [
            [0, 1, 2], [3, 4, 5], [6, 7, 8],
            [0, 3, 6], [1, 4, 7], [2, 5, 8],
            [0, 4, 8], [2, 4, 6],
          ],
          board: [],
          win: false,
          imgError: false,
        };
      },
      methods: {
        reset() {
          this.win = false;
          this.imgError = false;
          this.board = this.phrases.map(text => ({
            text,
            marked: text === "FREE SPACE"
          }));
        },
        mark(idx) {
          if (this.win) return;
          if (this.board[idx].text === "FREE SPACE") return;
          this.board[idx].marked = !this.board[idx].marked;
          if (this.checkWin()) this.win = true;
        },
        checkWin() {
          return this.winCombos.some(combo =>
            combo.every(i => this.board[i].marked)
          );
        },
      },
      mounted() {
        this.reset();
      }
    }).mount('#app');
    </script>
  </body>
</html>
