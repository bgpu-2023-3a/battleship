<template>
  <v-dialog v-model="isOpen" persistent overlay-opacity="0.8">
    <div class="menu-container">
      <button class="btn" @click="$emit('start-new-game')">
        Новая игра
      </button>
      <button
        class="btn"
        :class="{ disable: options.resume.isDisabled }"
        @click="$emit('resume-game')"
      >
        Продолжить
      </button>    
    </div>
  </v-dialog>

  <button id="replay__btn" type="button" name="button">Play the game</button>

$(document).ready(function() {
    let app = {
        cards: ['&spades;', '&spades;', '&clubs;', '&clubs;', '&hearts;', '&hearts;', '&diams;', '&diams;', '&sung;', '&sung;', '&starf;', '&starf;'],

        init: function() {
            app.shuffle();
        }, //end jf init func

        shuffle: function() {
            let random = 0;
            let temp = 0;
            for (let i = 1; i < app.cards.length; i++) {
                random = Math.round(Math.random() * i);
                temp = app.cards[i];
                app.cards[i] = app.cards[random];
                app.cards[random] = temp;
            }
            console.log(app.cards);
            app.assignCards();
            console.log(`Shuffle arr ${app.cards}`);

        }, //end jf shuffle func

        assignCards: function() {
            $('.card').each(function(index) {
                $(this).data('cardValue', app.cards[index]).html('').addClass('unmatched').removeClass('selected').css({opacity: 1});
            });
            app.clickHandlers();
        }, //end jf assignCards func

        clickHandlers: function() {
            $('.card').on('click', function() {
                $(this).html('<p>' + $(this).data('cardValue') + '</p>').addClass('selected');
                app.checkMatch();
            });
        }, //end jf clickHandlers func

        checkMatch: function() {
            if ($('.selected').length === 2) {
                if ($('.selected').first().data('cardValue') == $('.selected').last().data('cardValue')) {
                    $('.selected').each(function() {
                        $(this).animate({
                            opacity: 0
                        }).removeClass('unmatched');
                    });
                    $('.selected').each(function() {
                        $(this).removeClass('selected');
                    });
                    app.win();
                } else {
                    setTimeout(function() {
                        // alert("jjj")
                        $('.selected').each(function() {
                            $(this).html('').removeClass('selected');
                        });
                    }, 2000);
                }
            }
        }, //end chek funk
        win: function() {
            if ($('.unmatched').length === 0) {
                $('#modal').modal();
            }
        }
    };

    console.log(app);
    app.init();

    function startPlay() {
        app.init();
    }
    
    $('#replay__btn').click(startPlay);

});
</template>

<script>
export default {
  name: 'TheGameMenu',

  props: {
    isOpen: Boolean,
    options: Object,
  },

  methods: {
   
  },
};
</script>

<style scoped>
.btn {
  margin: 2rem;
  font-size: 3rem;
  font-family: bfont;
  outline: none;
  text-shadow: 0 2px 2px black;
  text-align: center;
}

.btn:hover,
.btn:focus {
  color: rgb(128, 255, 0);
}

.menu-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background-color: rgba(34, 34, 34, 0.8);
  box-shadow: inset 0 0 2px 2px rgb(56, 56, 56);
}

.source {
 text-decoration: underline;
 color: rgb(255, 103, 103);
 text-shadow: 0 0px 2px black;
 font-size: 2rem;
}

.disable {
  pointer-events: none;
  color: rgb(117, 117, 117);
}

.author {
  font-family: bfont;
  text-shadow: 0 2px 2px black;
  font-size: 1.6rem;
}
</style>
