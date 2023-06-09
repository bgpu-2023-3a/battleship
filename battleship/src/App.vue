<template>
  <v-app>
    <TheNavBar @show-game-menu="handleShowGameMenu" />
    <v-content>
      <TheGame
        ref="game"
        @round="handleRound"
      />
      <TheGameMenu
        :isOpen="isGameMenuOpen"
        :options="gameMenuOptions"
        @start-new-game="handleNewGame"
        @resume-game="handleResumeGame"
      />
      <TheGameBoardRedactor :isOpen="isGameBoardRedactorOpen" @start-game="handleStartGame"/>
    </v-content>
  </v-app>
</template>

<script>
import TheNavBar from './components/TheNavBar.vue';
import TheGameMenu from './components/TheGameMenu.vue';
import TheGameBoardRedactor from './components/TheGameBoardRedactor.vue';
import TheGame from './components/TheGame.vue';

import createPlayer from './scripts/factories/createPlayer';

export default {
  name: 'App',

  components: {
    TheGame,
    TheNavBar,
    TheGameMenu,
    TheGameBoardRedactor,
  },

  data: () => ({
    isGameMenuOpen: true,
    isGameBoardRedactorOpen: false,
    gameMenuOptions: {
      resume: {
        isDisabled: true,
      },
    },
    pl: null,
    pc: null,
    plHasDamaged: false,
    pcHasDamaged: false,
    gameHasAwinner: false,
  }),

  methods: {
    hideGameMenu() {
      this.isGameMenuOpen = false;
    },

    handleShowGameMenu() {
      this.isGameMenuOpen = true;
    },

    openGameBoardRedactor() {
      this.isGameBoardRedactorOpen = true;
    },

    closeGameBoardRedactor() {
      this.isGameBoardRedactorOpen = false;
    },

    handleNewGame() {
      this.gameHasAwinner = false;
      this.gameMenuOptions.resume.isDisabled = false;

      this.hideGameMenu();
      this.openGameBoardRedactor();
      this.$refs.game.resetTheGame();
    },

    handleResumeGame() {
      this.hideGameMenu();
    },

    handleStartGame(plBoard, plBoardElement, pcBoard, pcBoardElement) {
      this.closeGameBoardRedactor();

      this.pl = createPlayer({ board: plBoard });
      this.pc = createPlayer({ board: pcBoard, isPc: true });

      this.$refs.game.initTheGame(plBoardElement, pcBoardElement, this.pl, this.pc);
    },

    handleRound(pcCordAttack) {
      if (!this.gameHasAwinner) {
        this.plHasDamaged = this.makePlTurn(pcCordAttack);
        this.$refs.game.updateTheBoardsInfo();

        if (this.pc.getBoard().isAllShipsSunk()) {
          this.gameHasAwinner = true;
          this.gameMenuOptions.resume.isDisabled = true;
          this.$refs.game.updateGameInfo('Вы победитель!', 'rgb(43, 197, 87)');
          setTimeout(this.handleShowGameMenu, 3000);

          return;
        }

        if (this.plHasDamaged) return;

        this.$refs.game.updateGameInfo('Ход PC!', 'rgb(226, 54, 54)');
        this.$refs.game.disablePcBoard();

        const delayPcTurn = (ms) => {
          setTimeout(() => {
            this.pcHasDamaged = this.makePcTurn();
            this.$refs.game.updateTheBoardsInfo();

            if (this.pl.getBoard().isAllShipsSunk()) {
              this.gameHasAwinner = true;
              this.gameMenuOptions.resume.isDisabled = true;
              this.$refs.game.updateGameInfo('Выйграл PC!', 'rgb(226, 54, 54)');
              setTimeout(this.handleShowGameMenu, 3000);

              return;
            }

            if (this.pcHasDamaged) {
              delayPcTurn(ms);

              return;
            }

            this.$refs.game.updateGameInfo('Твой ход!', 'rgb(43, 197, 87)');
            this.$refs.game.enablePcBoard();
          }, ms);
        };

        delayPcTurn(500);
      }
    },

    makePlTurn(pcCordAttack) {
      watch: {
    // для изменения статусов сообщения и цвета
    gameStatus () {
        if (this.gameStatus === 'win') {
            this.gameStatusColor = 'statusWin'

            this.gameStatusMessage = `${this.activePlayer} Wins !`

            return
        } else if (this.gameStatus === 'draw') {
            this.gameStatusColor = 'statusDraw'

            this.gameStatusMessage = 'Draw !'

            return
        }

        this.gameStatusMessage = `${this.activePlayer}'s turn`
    }
}
      const { x, y } = JSON.parse(pcCordAttack);
      const attackInfo = this.pl.attack({ player: this.pc, x, y });
      this.$refs.game.updatePcBoard(pcCordAttack, attackInfo);

      return attackInfo === true || attackInfo.damagedShipData;
    },

    makePcTurn() {
      const { attackInfo, cord } = this.pc.attack({ player: this.pl });
      const { x, y } = cord;
      this.$refs.game.updatePlBoard(JSON.stringify({ x, y }), attackInfo);

      return attackInfo === true || attackInfo.damagedShipData;
    },
  },
};
</script>

<style>
@font-face {
  font-family: bfont;
  /*src: url('./assets/fonts/04B_30__.TTF');*/
}

*,
*::before,
*::after {
  padding: 0;
  margin: 0;
  box-sizing: border-box;
  font-size: inherit;
  user-select: none !important;
}

html {
  font-size: 10px !important;
}

@media screen and (max-width: 860px) {
  html {
    font-size: 7px !important;
  }
}

@media screen and (max-width: 410px) {
  html {
    font-size: 6px !important;
  }
}

@media screen and (max-width: 359) {
  html {
    font-size: 5px !important;
  }
}

@media screen and (max-width: 350px) {
  html {
    font-size: 4px !important;
  }
}
</style>
