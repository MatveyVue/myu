<template>
<div style="margin-top: 20vw;" id="likes-container">
  <div style="text-align: center;">
    <p id="username" style="color: white; font-family: Geologica;">@{{ user }}</p>
      <span class="score" id="count"><b>{{ likeCount }}</b></span><br>
      <button class="like" id="like" @click="tap"><img class="tap" src="https://i.postimg.cc/xCj2Qkp5/2587-D1-A6-37-E0-45-B0-89-E4-3-F4-D29-E83130.png"></button>
  </div>
        <div class="bar">
          <RouterLink to="/task">
            <img src="https://www.svgrepo.com/show/311307/task-list.svg" width="40px" style="position: absolute; margin-left: 49vw; margin-top: 1vw; color: white;">
               <button class="task">Task</button>
          </RouterLink>
          <RouterLink to="/leader">
            <img src="https://i.postimg.cc/xCj2Qkp5/2587-D1-A6-37-E0-45-B0-89-E4-3-F4-D29-E83130.png" width="40px" style="position: absolute; margin-left: 9.5vw;">
              <button class="game"><b>Game</b></button>
          </RouterLink>
          <RouterLink to="/leader">
              <button class="soon">Leader</button>
          </RouterLink>
        </div>
</div>
</template>

<style scoped>
  body {
      background-color: rgb(0, 0, 0);
      touch-action: none;
      overflow: none;
  }
  .score {
      font-family: Geologica;
      color: white;
      font-size: 36px;
  }
  .like {
      margin-top: 20%;
      background-color: black;
      border: none;
  }
  .tap:active {
      transform: translateY(5px);
  }
  .tap {
      width: 320px;
  }
  .bar {
      background-color: rgb(0, 0, 0);
      bottom: 0;
      width: 150%;
      height: 10%;
      border-radius: 25px;
      margin-left: -40vw;
      position: fixed;
  }
  .task {
      background-color: rgb(0, 0, 0);
      width: 33vw;
      height: 105px;
      border: none;
      font-family: Geologica;
      color: white;
      margin-left: 38vw;
  }
  .game {
      background-color: rgb(0, 0, 0);
      width: 33vw;
      height: 100px;
      border: none;
      font-family: Geologica;
      color: white;
      margin-left: -2vw;
      font-size: 18px;
  }
  .soon{
      background-color: rgb(0, 0, 0);
      width: 38vw;
      height: 105px;
      border: none;
      font-family: Geologica;
      color: white;
      margin-left: -1vw;
  }
</style>

<script setup>
import { ref, onMounted } from 'vue';
import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-app.js";
import { getDatabase, ref as dbRef, set, get, onValue } from "https://www.gstatic.com/firebasejs/11.0.1/firebase-database.js";

const firebaseConfig = {
  apiKey: "AIzaSyBhTL7JcmGtdwPpJyKkwMRoIEOIU17ZxUk",
  authDomain: "tapalka-c9556.firebaseapp.com",
  databaseURL: "https://tapalka-c9556-default-rtdb.firebaseio.com",
  projectId: "tapalka-c9556",
  storageBucket: "tapalka-c9556.firebasestorage.app",
  messagingSenderId: "386428968826",
  appId: "1:386428968826:web:de56837a0bbc2cf27940c3",
  measurementId: "G-YENX5XCT8J"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);
const likeCount = ref(0);
const username = ref("");

// Функция для получения или установки имени пользователя
const getOrSetUsername = async () => {
  const usernameRef = dbRef(db, `users/${username.value}`);
  try {
    const snapshot = await get(usernameRef);
    if (snapshot.exists()) {
      // Если имя пользователя уже существует, используем его
      username.value = snapshot.val().username;
    } else {
      // Если имя пользователя не существует, запрашиваем его у пользователя
      username.value = prompt("Введите ваше имя пользователя:");
      if (username.value) {
        // Сохраняем имя пользователя в базе данных
        await set(usernameRef, { username: username.value });
      }
    }
  } catch (error) {
    console.error("Error fetching or setting username:", error);
  }
};

const tap = async () => {
  if (!username.value) {
    await getOrSetUsername();
  }
  const countsRef = dbRef(db, `counts/${username.value}`);
  try {
    const snapshot = await get(countsRef);
    let tap = snapshot.exists() ? snapshot.val().count : 0;
    tap++;
    likeCount.value = tap;
    await set(countsRef, { count: tap });
  } catch (error) {
    console.error("Error updating taps:", error);
  }
};

const fetchInitialLikeCount = async () => {
  if (!username.value) {
    await getOrSetUsername();
  }
  const countsRef = dbRef(db, `counts/${username.value}`);
  try {
    const snapshot = await get(countsRef);
    if (snapshot.exists()) {
      likeCount.value = snapshot.val().count;
    }
  } catch (error) {
    console.error("Error fetching initial taps count:", error);
  }
};

onMounted(fetchInitialLikeCount);
</script>

<script>
export default {
   data() {
      return {
          user: window.Telegram.WebApp.initDataUnsafe.user.username
       }
   }
}
</script>
