<script setup lang="ts">
import { ref } from 'vue'
import lotoData from '../assets/loto-data-since-2008-10.json';
import Balls from '../components/ui/Balls.vue'
console.log('lotoData', lotoData)
type Lotery = 'loto' | 'euromillion';
type LotterySelection = {
  type: Lotery;
  selectedNumbers: number[];
  selectedStars: number[];
}[];

const selectedNumbers = ref<number[]>([])
const selectedStars = ref<number[]>([])
const selectedTab = ref<Lotery>('loto')
const savedSelection = ref<LotterySelection>([
  { type: 'loto', selectedNumbers: [], selectedStars: []},
  { type: 'euromillion', selectedNumbers: [], selectedStars: []}
]);
const winData = ref<string[]>([]);
const hasPlay = ref<boolean>(false);

function toggleSelected(num: number, limit: number, type: string = 'numbers'): void {
  const array = type === 'numbers' ? selectedNumbers.value : selectedStars.value
  const index = array.indexOf(num)
  if (index > -1) { // Delete
    array.splice(index, 1)
  } else if (array.length < limit) { // Add
    array.push(num)
  } else {
    array.pop();
    array.push(num)
  }
}

function isSelected(num: number, type: string = 'numbers'): boolean {
  const array = type === 'numbers' ? selectedNumbers.value : selectedStars.value
  return array.includes(num)
}

function selectTab(tabName: Lotery): void {
  // Save previous selection values
  const previousSelection = savedSelection.value.find(l => l.type === selectedTab.value);

  if (previousSelection) {
    previousSelection.selectedNumbers = selectedNumbers.value
    previousSelection.selectedStars = selectedStars.value
  }
  
  selectedTab.value = tabName;

  // Get new selection values
  const newSelection = savedSelection.value.find(l => l.type === selectedTab.value);

  if (newSelection) {
    selectedNumbers.value = newSelection.selectedNumbers
    selectedStars.value = newSelection.selectedStars
  }
}

function clearSelection() {
  const selectionToClear = savedSelection.value.find(l => l.type === selectedTab.value)
  if (selectionToClear) {
    selectionToClear.selectedNumbers = []
    selectionToClear.selectedStars = []
  }
  selectedNumbers.value = []
  selectedStars.value = []
}

function checkSelection() {
  let finalSelection = selectedNumbers.value.sort((a, b) => a - b).join("-") + "+"
  console.log("selectedNumbers.value" , selectedNumbers.value.sort((a, b) => a - b).join("-"))
  console.log("selectedStars.value" , selectedStars.value)
  finalSelection += selectedStars.value.join("+")

  console.log("finalSelection", finalSelection)
  const winFind = lotoData.filter(l => l["combinaison_gagnante_en_ordre_croissant"] === finalSelection)
  console.log("winFind", winFind)
  if (winFind) {
    winFind.forEach(w => {
      winData.value.push(w.date_de_tirage);
    });
  }
  
  hasPlay.value = true;
  console.log("winData", winData.value)
}

function resetGame() {
  hasPlay.value = false;
  winData.value = [];
}
</script>

<template>
  <main>
    <div>
      <h1>LotoChecker</h1>
      <p>Ne manquez jamais une occasion de gagner! Entrez votre grille de loto ou d'Euromillions et découvrez instantanément si vous êtes un gagnant.</p>

    
    
  <div>
    <div>
      <button :class="{ 'active': selectedTab === 'loto' }" @click="selectTab('loto')">Loto</button>
      <button :class="{ 'active': selectedTab === 'euromillion' }" @click="selectTab('euromillion')">Euromillion</button>
    </div>
    <div v-if="selectedTab === 'loto'">
      <h2>Grille de Loto</h2>
      <div>
        <p>Sélectionnez 5 numéros :</p>
        <div>
          <span v-for="num in 49" :key="num" @click="toggleSelected(num, 5)"
                :class="{ 'selected': isSelected(num) }">
            {{ num }}
          </span>
        </div>
        <p>Sélectionnez 1 étoile :</p>
        <div>
          <span v-for="star in 10" :key="star" @click="toggleSelected(star, 1, 'stars')"
                :class="{ 'selected': isSelected(star, 'stars') }">
            {{ star }}
          </span>
        </div>
      </div>
    </div>
    <div v-else-if="selectedTab === 'euromillion'">
      <h2>Grille d'Euromillion</h2>
      <div>
        <p>Sélectionnez 5 numéros :</p>
        <div>
          <span v-for="num in 50" :key="num" @click="toggleSelected(num, 5)"
                :class="{ 'selected': isSelected(num) }">
            {{ num }}
          </span>
        </div>
        <p>Sélectionnez 2 étoiles :</p>
        <div>
          <span v-for="star in 12" :key="star" @click="toggleSelected(star, 2, 'stars')"
                :class="{ 'selected': isSelected(star, 'stars') }">
            {{ star }}
          </span>
        </div>
      </div>
    </div>
    <button @click="clearSelection()">Effacer</button>
    <button @click="checkSelection()">Valider</button>

    <Balls></Balls>
  </div>

  <div v-if="hasPlay">
    <div v-if="winData.length > 0">
      <h2>Félicitations</h2>
      <p>
        Avec ce tirage, vous auriez gagné le: <br>
        <span v-for="(win, index) in winData" :key="index">{{ win }}<br></span>
      </p>
    </div>
    <div v-else>
      <h2>Dommage</h2>
      <p>Ce tirage n'est pas gagnant</p>
    </div>
    <button @click="resetGame()">Recommencer</button>
  </div>
    </div>
  </main>
</template>

<style scoped>
.selected {
  background-color: yellow;
  font-weight: bold;
  color: black;
}
button {
  padding: 5px 10px;
  margin-right: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  outline: none;
}
button.active {
  background-color: lightblue;
}
</style>