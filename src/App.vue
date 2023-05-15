<script setup>
import { onMounted, ref } from "vue";

const ranImgs = ref([]);
const length = ref(15);
const firstNum = ref(0);
const secondNum = ref(0);
const count = ref(0);
const shuffledPair = ref([]);

const shuffleDeck = () => {
  const shuffled = [...ranImgs.value];
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }
  shuffledPair.value = shuffled;
};

const createGame = () => {
  for (let i = 1; i <= length.value; i++) {
    const url = `https://picsum.photos/200?random=${i}`;
    ranImgs.value.push({ id: i, url: url, checked: true, matched: false });
    ranImgs.value.push({ id: i, url: url, checked: true, matched: false });
  }
  shuffleDeck();
};

const resetGame = () => {
  ranImgs.value = [];
  shuffledPair.value = [];
  createGame();
}

const flipFront = { transform: "rotateY(0)" };
const flipBack = { transform: "rotateY(180deg)" };

const checkCard = (img) => {
  if (img.checked || img.matched) return;
  count.value < 2 ? (img.checked = !img.checked) : false;
  if (firstNum.value === 0) {
    count.value++;
    firstNum.value = img.id;
  } else {
    count.value++;
    secondNum.value = img.id;
    if (firstNum.value === secondNum.value) {
      shuffledPair.forEach((card) => (card.id === img.id ? (card.matched = true) : false));
      firstNum.value = 0;
      secondNum.value = 0;
      count.value = 0;
    } else {
      setTimeout(() => {
        shuffledPair.forEach((card) => (card.checked = false));
        firstNum.value = 0;
        secondNum.value = 0;
        count.value = 0;
      }, 500);
    }
  }
};

onMounted(() => {
  createGame();
  setTimeout(() => {
    shuffledPair.value.forEach((card) => (card.checked = false));
  }, 3000);
});
</script>

<template>
  <header>
    <h1>MEMORY GAME</h1>
  </header>

  <main>
    <ul class="mainBoard">
      <li v-for="img in shuffledPair" :key="img.id" @click="checkCard(img)">
        <img :src="img.url" :alt="`랜덤이미지+${img.id}`" :style="(img.checked || img.matched) && flipFront" />
        <div class="backFace" :style="(img.checked || img.matched) && flipBack">카드 뒷면</div>
      </li>
    </ul>
    <button @click="resetGame">다시하기</button>
    <button>난이도 변경</button>
  </main>
</template>

<style scoped>
header h1 {
  text-align: center;
}
main {
  overflow: hidden;
}
.mainBoard {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(6, 150px);
  grid-auto-rows: repeat(6, 150px);
  place-content: center;
  list-style: none;
  padding-left: 0;
}
li {
  width: 150px;
  height: 150px;
  perspective: 300px;
  position: relative;
}
li img,
li .backFace {
  width: 150px;
  height: 150px;
  cursor: pointer;
  transition: 0.6s;
  backface-visibility: hidden;
  border: 1px solid #000;
}
li img {
  object-fit: cover;
  position: absolute;
  transform: rotateY(-180deg);
}
li .backFace {
  text-indent: -9999px;
  background-color: orange;
}
</style>
