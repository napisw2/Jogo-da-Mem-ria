<template>
  <div >

    <img src="./../public/images/peek-a-vue-title.png" class="title1" alt="Peek-a-Vue" style="padding-inline: 20px;">
    <section>
      <p style="padding: 30px;">Bem Vindo ao meu Jogo da Memória!! </p>
    </section>
    <transition-group tag="section" class="game-board"   name="shuffle-card">
      <CardView 
        v-for="card in cardList" 
        :key="`${card.value}-${card.variant}`" 
        :value="card.value" 
        :visible="card.visible"
        :position="card.position"
        :matched="card.matched"
        @select-card="flipCard"
      />                  <!--  chamando a componente CardView -->
    </transition-group>
    <br>
    <h2 class="status">{{ status }}</h2>
    <br>
    <button v-if="newPlayer" @click="startGame" class="restartButton" alt="Start Icon"> 
      <img src="/images/play.svg">
      Start Game
    </button>
    <button v-else @click="restartGame" class="restartButton" alt="Restart Icon"> 
      <img src="/images/restart.svg">
       Restart Game
    </button>
  </div>
  
</template>


<script>

import { launchConfetti } from './utilities/confetti';
import _ from 'lodash';
import CardView from './components/CardView.vue';
import {ref, watch, computed} from 'vue'; // aqui contém métodos que nos permitem dizer o que é ou não reativo
export default {

  name:'App',
  components: {
    CardView
  },
  setup() {
    
    const cardList = ref([]);    // ref torna reativo?
    const userSelection = ref([]);
    const newPlayer = ref(true) ;

    const startGame = () => {
      newPlayer.value = false;

      restartGame();
    }

    const parRestante = computed(() => {
      const cardRestante = cardList.value.filter(card => card.matched === false).length  
      //aqui iremos percorrer nossa cardList e veremos quais ainda não possuem matched,e a partir disso dividiremos por 2
      //pois queremos achar os pares
      return cardRestante/2 ;
    });

    const status = computed(() => {
      if(parRestante.value === 0){
        return ' Você Venceu !! ' ;
      }else{
        return ` Pares Restantes : ${parRestante.value}` ;
      }
    });
    
    const cardItems = [
      'bat',
      'candy',
      'cauldron',
      'cupcake',
      'ghost',
      'moon',
      'pumpkin',
      'witch-hat'
    ];
    
    cardItems.forEach(item => {
      cardList.value.push({
        value: item,
        visible: false,
        variant: 1,
        position: null,
        matched: false
      });

      cardList.value.push({
        value: item,
        variant: 2,
        visible: true,
        position: null,
        matched: false
      });

    });

    cardList.value = cardList.value.map((card,index) => {
      return {
        ...card,
        position: index
      }
    });
    
    const restartGame = () => {
      
      cardList.value = _.shuffle(cardList.value);  //funcao importada para embaralhar as cartas

      cardList.value = cardList.value.map((card, index) => {    
                                                                
        return {
          ...card,
          matched: false,
          position: index,
          visible: false
        }
      });
    }

    const flipCard = (posicao) => {

      let isVisible = cardList.value[posicao.position].visible;  
      
      if(isVisible !== false){   //nao permite clicar no que já foi resolvido
          return ;
      }
      console.log(isVisible);

      cardList.value[posicao.position].visible = true;  

      var selectedCard = userSelection.value[0];

      if(selectedCard){        //aqui estamos impedindo o bug de escolher a mesma carta
        if(selectedCard.position === posicao.position && selectedCard.faceValue === posicao.faceValue ){
          return ;
        }else{
          userSelection.value[1] = posicao;
        } 
      }else{
        userSelection.value[0] = posicao;
      }
    }

    watch (parRestante, currentValue => {
      if(currentValue === 0) {
        launchConfetti();
      }
    })

    watch(userSelection, currentValue => {

      if(currentValue.length === 2){
        const cardOne = currentValue[0];
        const cardTwo = currentValue[1];

        if(cardOne.faceValue === cardTwo.faceValue){ 
          cardList.value[cardOne.position].matched = true;
          cardList.value[cardTwo.position].matched = true;
        }else{
          setTimeout(() => {
            cardList.value[cardOne.position].visible = false;
            cardList.value[cardTwo.position].visible = false;   //torna as cartas invisiveis
          }, 2000);
        }

        userSelection.value.length = 0 //fazendo a length voltar a 0, permite que o usuario volte a clicar
      
      }
    },{deep: true})
    
    return {
      cardList,
      flipCard,
      userSelection,
      status,
      parRestante,
      restartGame,
      startGame,
      newPlayer
    }
  }
 
}

</script>

<style>

html,body{
  margin: 0;
  padding: 0;
  padding-top: 60px;
  height: 100%;
  /* max-height:100%;
  overflow:hidden; */
}

h1{
  margin-top:0;
}

html{
background-color: #000000;
background-image: linear-gradient(132deg, #000000 0%, #16A085 100%);   

}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  /* background-image: url('.././public/images/24.jpg');
  background-repeat: no-repeat;
  background-size: cover;
  background-position: center;
  height: 100vh;
  width: 100%; */
  color: #2c3e50;
  color: #fff;
  padding-top: 60px ;
  
 
}

.restartButton {
  background-color: orange;
  color: black;
  padding: 0.75rem 0.75rem;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  font-weight: bold;
  font-family: 'Titillium Web',sans-serif;
  font-size: 1.1rem;
  border: 3px rgb(0, 0, 0) solid;
  border-radius: 10px;
}

.restartButton img {
  padding-right: 7px;
 
}

.status {
  font-family: "Titillium Web", sans-serif;
  font-size: 18px;
  text-transform: uppercase;
}

.game-board {
  /* display: grid;
  grid-template-columns: 120px 120px 120px 120px;
  grid-template-rows: 120px 120px 120px 120px;

  grid-column-gap: 40px;
  grid-row-gap: 40px;
  justify-content: center; */
  display: grid;
  grid-template-columns: repeat(4, 60px);
  grid-template-rows: repeat(4, 60px);
  grid-column-gap: 12px;
  grid-row-gap: 12px;
  justify-content: center;
  
}
.title1{
  padding-bottom: 30px;
}
.shuffle-card-move {
  transition: transform 0.8s ease-in;
}

@media screen and (min-width: 500px) {
  .game-board {
    grid-template-columns: repeat(4, 90px);
    grid-template-rows: repeat(4, 90px);
  }
}

@media screen and (min-width: 600px) {
  .game-board {
    grid-template-columns: repeat(4, 120px);
    grid-template-rows: repeat(4, 120px);
  }
}

</style>
