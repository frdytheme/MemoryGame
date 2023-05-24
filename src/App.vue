<script setup>
import { onMounted, ref } from "vue";

const ranImgs = ref([]);
const length = ref(10);
const shuffledPair = ref([]);
const showGrade = ref(false);
const isGrade = ref("");
const clearGame = ref(false);

// 선오픈 카드 id
const firstNum = ref(0);
// 후오픈 카드 id
const secondNum = ref(0);
// 2장 오픈 제한용 count
const count = ref(0);
// 총 오픈된 카드 확인 count
const totalCount = ref(0);

/*
  게임 생성
  랜덤 이미지 배열 초기화 후 for반복문으로 객체 배열 생성
  한 쌍의 이미지가 나와야 하므로 객체도 쌍으로 저장
*/
const createGame = () => {
  ranImgs.value = [];
  totalCount.value = 0;
  clearGame.value = false;
  for (let i = 1; i <= length.value; i++) {
    const url = `https://picsum.photos/200?random=${i}`;
    ranImgs.value.push({ id: i, url: url, checked: true, matched: false });
    ranImgs.value.push({ id: i, url: url, checked: true, matched: false });
  }
};

/*
  생성된 배열 셔플
  스프레드연산자로 생성된 배열 복사 후 무작위 순서 섞기
  shuffledPair ref에 섞인 배열 할당 후 각 카드의 checked 속성 true로 변경 -> true일시 오픈
  * 3초 동안 오픈 후 전체 뒤집기 기능이나, 무지성으로 난이도 변경 시 로직이 꼬임. setTimeout의 비동기 동작이 문제로 보임. -> async await로 해결 안 됨.
*/
const shuffleDeck = async () => {
  const shuffled = [...ranImgs.value];
  for (let i = shuffled.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
  }
  shuffledPair.value = shuffled;
  shuffledPair.value.forEach((card) => (card.checked = true));
  await new Promise((res) => setTimeout(res, 3000));
  shuffledPair.value.forEach((card) => (card.checked = false));
};

// 게임 시작 메서드
const startGame = () => {
  createGame();
  shuffleDeck();
};

// 난이도 변경 버튼.
// 난이도에 따라 grid 레이아웃 변경을 위해 isGrade ref 활용
const changeLevel = (e) => {
  length.value = e.target.value;
  isGrade.value = e.target.textContent;
  startGame();
};

// 최초 렌더링 시 자동으로 게임 생성
onMounted(() => {
  createGame();
  shuffleDeck();
});

// 카드 뒤집는 style 객체
const flipFront = { transform: "rotateY(0)" };
const flipBack = { transform: "rotateY(180deg)" };

/*
  카드 클릭 메서드
  1. 오픈된 카드 클릭 시 return
  2. 2장만 오픈할 수 있으므로 count.value와 삼항연산자 활용.
  3. firstNum에 선오픈 카드 secondNum에 후오픈 카드 id 저장.
  4. 두 변수 비교 후 같으면 matched 속성 true -> 오픈 상태 유지 / 클릭 불가
  5. 짝을 찾으면 totalCount가 올라가며 총 배열 / 2와 값이 같아지면 게임 클리어.
  6. 짝이 아닐 시 setTimeout을 쓴 이유는 오픈한 카드를 확인할 시간이 필요하기 때문.
*/
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
      shuffledPair.value.forEach((card) => (card.id === img.id ? (card.matched = true) : false));
      firstNum.value = 0;
      secondNum.value = 0;
      count.value = 0;
      totalCount.value++;
      if (totalCount.value !== 0 && totalCount.value * 2 === shuffledPair.value.length) {
        clearGame.value = true;
      }
    } else {
      setTimeout(() => {
        shuffledPair.value.forEach((card) => (card.checked = false));
        firstNum.value = 0;
        secondNum.value = 0;
        count.value = 0;
      }, 500);
    }
  }
};
</script>

<template>
  <header>
    <h1>MEMORY GAME</h1>
  </header>

  <main>
    <article class="main_view">
      <ul :class="'main_board' + ' ' + isGrade">
        <li v-for="img in shuffledPair" @click="checkCard(img)">
          <img
            :src="img.url"
            :alt="`랜덤이미지${img.id}`"
            class="front_face"
            :style="(img.checked || img.matched) && flipFront" />
          <img src="./assets/images/pattern.png" class="back_face" :style="(img.checked || img.matched) && flipBack" />
        </li>
      </ul>
      <ul class="menu">
        <li @click="startGame">REPLAY</li>
        <li @click="showGrade = !showGrade">
          LEVEL
          <ul class="grade" v-show="showGrade" @click="changeLevel">
            <li value="10">easy</li>
            <li value="15">normal</li>
            <li value="20">hard</li>
          </ul>
        </li>
      </ul>
    </article>
    <div class="clear_msg" v-show="clearGame">
      <p>Congratulations!!</p>
      <p @click="startGame">Retry</p>
    </div>
  </main>
</template>

<style scoped>
header h1 {
  text-align: center;
  position: absolute;
  color: #fff;
  top: 30px;
  left: 50%;
  transform: translateX(-50%);
  color: #ffb2b2;
}
main {
  overflow: hidden;
}
ul {
  list-style: none;
  padding-left: 0;
  margin: 0;
}
.main_view {
  width: 100%;
  height: 100vh;
  background-color: #000;
}
.main_board {
  width: 100%;
  display: grid;
  grid-template-columns: repeat(5, 150px);
  grid-auto-rows: 150px;
  place-content: center;
  padding-top: 160px;
}
.main_board.easy {
  grid-template-columns: repeat(5, 150px);
  grid-auto-rows: 150px;
}
.main_board.normal {
  grid-template-columns: repeat(6, 140px);
  grid-auto-rows: 140px;
}
.main_board.hard {
  grid-template-columns: repeat(8, 130px);
  grid-auto-rows: 130px;
}
.main_board li {
  perspective: 300px;
  position: relative;
}
.main_board li img {
  width: 100%;
  height: 100%;
  cursor: pointer;
  transition: 0.6s;
  backface-visibility: hidden;
  border-radius: 10px;
  box-sizing: border-box;
}
.main_board li .front_face {
  object-fit: cover;
  position: absolute;
  transform: rotateY(-180deg);
}
.main_board li .back_face {
  border: 1px solid #ffb2b2;
}
.menu {
  color: #fff;
  font-size: 30px;
  font-weight: bold;
  position: absolute;
  top: 50%;
  left: 100px;
  transform: translateY(-50%);
}
.menu > li {
  margin: 100px 0;
  cursor: pointer;
}
.menu > li:hover {
  color: #ffb2b2;
}
.menu li:nth-child(2) {
  position: relative;
}
.menu li .grade {
  position: absolute;
  top: 100%;
  left: 0;
}
.menu li .grade li {
  opacity: 0.5;
}
.menu li .grade li:hover {
  opacity: 1;
}
.clear_msg {
  position: absolute;
  width: 100%;
  height: 100vh;
  background-color: #000000c5;
  top: 0;
  left: 0;
  display: flex;
  flex-flow: column;
  justify-content: center;
  align-items: center;
  text-align: center;
}
.clear_msg p {
  font-size: 100px;
  font-weight: bold;
  color: #fff;
  user-select: none;
}
.clear_msg p:nth-child(2) {
  font-size: 50px;
  user-select: all;
  cursor: pointer;
}
.clear_msg p:nth-child(2):hover {
  color: #e76161;
}
</style>
