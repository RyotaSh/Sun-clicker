<template>
  <div class="lvl0">
    <h2>Mon Soleil</h2>
    <img
      :class="{ pulse: isPulsing }"
      @click="clickSoleil"
      :src="currentImage"
      alt="Soleil"
    />
    <p>Score total : {{ Math.floor(score) }}</p>
    <p>Argent : {{ Math.floor(argent) }}</p>
    <button @click="resetGame">Reset</button>

    <p class="info-text">
      Heat up the celestial body!<br>
      And discover the mysteries of the universe!
    </p>
    <div class="shop">
      <h3>Boutique</h3>

      <!-- Esprit -->
      <div class="item">
        <h4>Esprit</h4>
        <p>Production passive : {{ espritPower.toFixed(2) }}/s</p>
        <p>Prix : {{ espritPrice.toFixed(0) }} argent</p>
        <button :disabled="argent < espritPrice" @click="acheterEsprit">Acheter</button>
        <p>Possédés : {{ espritCount }}</p>
      </div>

      <!-- Boost de click -->
      <div class="item">
        <h4>Boost de click</h4>
        <p>+1 point par click</p>
        <p>Prix : {{ boostPriceDisplay }} argent</p>
        <button :disabled="argent < boostPriceDisplay" @click="acheterBoost">Acheter</button>
        <p>Possédés : {{ boostCount }}</p>
      </div>
    </div>
  </div>
</template>


<script setup>
import { ref, onMounted, watch, computed } from 'vue'

const score = ref(0)
const argent = ref(0)
const isPulsing = ref(false)
const clickPower = ref(1)

// reset le jeu
function resetGame() {
  score.value = 0
  argent.value = 0
  espritCount.value = 0
  espritPower.value = 0
  espritPrice.value = espritBasePrice
  boostCount.value = 0
  clickPower.value = 1

  // also clear localStorage
  localStorage.removeItem('soleilScore')
  localStorage.removeItem('soleilArgent')
  localStorage.removeItem('soleilEspritCount')
  localStorage.removeItem('soleilEspritPrice')
  localStorage.removeItem('soleilBoostCount')
}

// Bonus passif : esprit 
const espritCount = ref(0)
const espritBasePower = 0.1
const espritPower = ref(0)
const espritBasePrice = 50
const espritPrice = ref(espritBasePrice)
const espritEfficiencyMultiplier = 1.3
const espritPriceIncrease = 1.1

//  Boost de click 
const boostCount = ref(0)
const boostPriceDisplay = computed(() => 100 + espritCount.value * 200)


const last10Bonus = ref(0)
const last100Bonus = ref(0)

//  Paliers d'image 
const imageLevels = [
  { score: 0, src: '/lvl0.png' },
  { score: 1000, src: '/lvl1.jpg' },
  { score: 2000, src: '/lvl2.jpg' },
  { score: 5000, src: '/lvl3.png' },
  { score: 10000, src: '/lvl4.jpg' },
  { score: 25000, src: '/lvl5.jpg' },
  { score: 70000, src: '/lvl6.jpeg' },
  { score: 160000, src: '/lvl7.png' },
  { score: 500000, src: '/lvl8.png' },
  { score: 1000000, src: '/lvlmax.png' },
]

// image actuelle selon le score 
const currentImage = computed(() => {
  let img = imageLevels[0].src
  for (const level of imageLevels) {
    if (score.value >= level.score) img = level.src
  }
  return img
})

//localStorage 

onMounted(() => {
  const savedScore = localStorage.getItem('soleilScore')
  const savedArgent = localStorage.getItem('soleilArgent')
  const savedEsprit = localStorage.getItem('soleilEspritCount')
  const savedEspritPrice = localStorage.getItem('soleilEspritPrice')
  const savedBoost = localStorage.getItem('soleilBoostCount')

  if (savedScore) score.value = parseFloat(savedScore)
  if (savedArgent) argent.value = parseFloat(savedArgent)
  if (savedEsprit) espritCount.value = parseInt(savedEsprit)
  if (savedEspritPrice) espritPrice.value = parseFloat(savedEspritPrice)
  if (savedBoost) boostCount.value = parseInt(savedBoost)

  espritPower.value = espritCount.value > 0 
    ? espritBasePower * Math.pow(espritEfficiencyMultiplier, espritCount.value - 1)
    : 0

  last10Bonus.value = Math.floor(score.value / 10)
  last100Bonus.value = Math.floor(score.value / 100)
  clickPower.value = 1 + boostCount.value
})

watch([score, argent, espritCount, espritPrice, boostCount], () => {
  localStorage.setItem('soleilScore', score.value)
  localStorage.setItem('soleilArgent', argent.value)
  localStorage.setItem('soleilEspritCount', espritCount.value)
  localStorage.setItem('soleilEspritPrice', espritPrice.value)
  localStorage.setItem('soleilBoostCount', boostCount.value)
})

//  Vérifie argent 
function checkBonusArgent() {
  const tens = Math.floor(score.value / 10)
  if (tens > last10Bonus.value) {
    argent.value += (tens - last10Bonus.value) * 5
    last10Bonus.value = tens
  }

  const hundreds = Math.floor(score.value / 100)
  if (hundreds > last100Bonus.value) {
    argent.value += (hundreds - last100Bonus.value) * 100
    last100Bonus.value = hundreds
  }
}

function clickSoleil() {
  score.value += clickPower.value
  isPulsing.value = true
  setTimeout(() => isPulsing.value = false, 300)
  checkBonusArgent()
}

//  Acheter esprit 
function acheterEsprit() {
  if (argent.value >= espritPrice.value) {
    argent.value -= espritPrice.value
    espritCount.value++
    espritPower.value = espritBasePower * Math.pow(espritEfficiencyMultiplier, espritCount.value -1)
    espritPrice.value = Math.round(espritPrice.value * espritPriceIncrease)
  }
}

//  Acheter boost de click 
function acheterBoost() {
  const prixBoost = 100 + espritCount.value * 200
  if (argent.value >= prixBoost) {
    argent.value -= prixBoost
    boostCount.value++
    clickPower.value = 1 + boostCount.value
  }
}

//  Production passive 
setInterval(() => {
  score.value += espritPower.value
  checkBonusArgent()
}, 1000)
</script>

<style scoped>
.info-text {
  font-family: 'Arial', sans-serif;      
  font-size: 1.5rem;
  font-weight: 600;
  text-align: center;
  background: linear-gradient(90deg, #ff8c00, #ffd700); 
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 8px rgba(255, 140, 0, 0.5), 0 0 12px rgba(255, 215, 0, 0.4);
  margin: 15px 0;
  animation: glow 2s ease-in-out infinite alternate;
}
.lvl0 {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
}

img {
  width: 150px;
  height: auto;
  transition: transform 0.2s ease, filter 0.2s ease;
  cursor: pointer;
}

img.pulse {
  transform: scale(1.2);
  filter: brightness(1.5);
}

.shop {
  margin-top: 20px;
  width: 320px;
  padding: 12px;
  border-radius: 8px;
  background: #f5f5f5;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.item {
  display: flex;
  flex-direction: column;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 6px;
  background: white;
}

button {
  padding: 6px 10px;
  border-radius: 5px;
  cursor: pointer;
}

button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>

