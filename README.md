
<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>CA SHOP - เว็บสุ่มไอดี Roblox</title>
<style>
  /* พื้นหลังดำ */
  body {
    margin: 0; padding: 0;
    background: #000;
    font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
    color: #eee;
    overflow-x: hidden;
  }

  /* container ใส่กรอบขาว พื้นหลังดำ */
  .container {
    position: relative;
    z-index: 1;
    max-width: 600px;
    margin: 2rem auto;
    padding: 2rem;
    background: #000; /* ดำ */
    border: 3px solid #fff; /* กรอบขาว */
    border-radius: 12px;
    color: #fff;
  }

  /* ตัวหนังสือและหัวข้ออยู่ในกรอบดำเล็กๆ */
  h1, h2, p, li {
    background: #000;
    display: inline;
    padding: 0 0.3rem;
    color: #fff;
  }

  ul {
    list-style-type: disc;
    margin-left: 1.2rem;
  }

  input, button {
    width: 100%;
    padding: 0.6rem;
    margin: 0.5rem 0;
    border-radius: 8px;
    border: 2px solid #fff;
    background: #000;
    color: #fff;
    font-size: 1rem;
    box-sizing: border-box;
  }

  button {
    cursor: pointer;
    font-weight: bold;
    transition: background 0.3s ease;
  }
  button:hover {
    background: #444;
  }
  #logoutBtn {
    background: #770000;
    color: #eee;
  }
  #logoutBtn:hover {
    background: #aa2222;
  }

  .credit {
    font-weight: 700;
  }

  .info-msg {
    color: #ccc;
    margin-top: 0.5rem;
  }

  /* EFFECT ดาวตกและฝนดาวตก */

  .starfield {
    position: fixed;
    width: 100%; height: 100%;
    top: 0; left: 0;
    pointer-events: none;
    overflow: hidden;
    z-index: 0;
    background: black;
  }

  /* ดาวตกเดี่ยว (ยาว) */
  .shooting-star {
    position: absolute;
    width: 2px;
    height: 80px;
    background: linear-gradient(-45deg, white, rgba(255,255,255,0));
    border-radius: 50%;
    opacity: 0.8;
    animation-timing-function: ease-in-out;
    animation-iteration-count: infinite;
    animation-name: shooting;
  }

  /* ฝนดาวตกหลายเส้น (สั้นกว่า) */
  .meteor {
    position: absolute;
    width: 1.5px;
    height: 40px;
    background: linear-gradient(-45deg, #ccc, rgba(204,204,204,0));
    border-radius: 50%;
    opacity: 0.6;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
    animation-name: meteorRain;
  }

  @keyframes shooting {
    0% {
      transform: translateX(0) translateY(0) rotate(45deg);
      opacity: 0;
    }
    10% {
      opacity: 1;
    }
    100% {
      transform: translateX(900px) translateY(450px) rotate(45deg);
      opacity: 0;
    }
  }

  @keyframes meteorRain {
    0% {
      transform: translateX(0) translateY(0) rotate(45deg);
      opacity: 0;
    }
    10% {
      opacity: 0.6;
    }
    100% {
      transform: translateX(600px) translateY(350px) rotate(45deg);
      opacity: 0;
    }
  }
</style>

<!-- Firebase CDN (compat) -->
<script defer src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
<script defer src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth-compat.js"></script>
<script defer src="https://www.gstatic.com/firebasejs/9.6.1/firebase-firestore-compat.js"></script>

</head>
<body>

<div class="starfield"></div>

<div class="container">
  <h1>🌟 CA SHOP – เว็บสุ่มไอดี Roblox 🌟</h1>

  <div id="auth-section">
    <input id="email" type="email" placeholder="อีเมล" />
    <input id="password" type="password" placeholder="รหัสผ่าน" />
    <button id="loginBtn">เข้าสู่ระบบ</button>
    <button id="registerBtn">สมัครสมาชิก</button>
  </div>

  <div id="user-section" style="display:none;">
    <p>👤 <span id="userEmail"></span></p>
    <p class="credit">💰 เครดิต: <span id="userCredit">0</span></p>
    <button id="logoutBtn">ออกจากระบบ</button>

    <div class="section">
      <h2>🎲 สุ่มไอดี Roblox (ใช้ 1 เครดิต)</h2>
      <button id="rollBtn">สุ่มไอดี</button>
    </div>

    <div class="section">
      <h2>💳 เติมเครดิตด้วยโค้ด</h2>
      <input id="codeInput" type="text" placeholder="กรอกโค้ดเครดิต" />
      <button id="redeemBtn">เติมโค้ด</button>
      <div class="info-msg" id="redeemMsg"></div>
    </div>

    <div class="section">
      <h2>📜 ประวัติการสุ่มไอดี</h2>
      <ul id="historyList"></ul>
    </div>
  </div>
</div>

<script>
  // สร้างดาวตกเดี่ยว (shooting star)
  function createShootingStar() {
    const star = document.createElement("div");
    star.classList.add("shooting-star");
    star.style.top = Math.random() * window.innerHeight * 0.5 + "px";
    star.style.left = Math.random() * window.innerWidth + "px";
    star.style.animationDuration = (Math.random() * 1 + 0.8) + "s";
    star.style.animationDelay = (Math.random() * 5) + "s";
    document.querySelector(".starfield").appendChild(star);

    star.addEventListener("animationend", () => {
      star.remove();
      createShootingStar();
    });
  }

  // สร้างฝนดาวตก (meteor rain)
  function createMeteor() {
    const meteor = document.createElement("div");
    meteor.classList.add("meteor");
    meteor.style.top = Math.random() * window.innerHeight * 0.5 + "px";
    meteor.style.left = Math.random() * window.innerWidth + "px";
    meteor.style.animationDuration = (Math.random() * 1 + 0.5) + "s";
    meteor.style.animationDelay = (Math.random() * 5) + "s";
    document.querySelector(".starfield").appendChild(meteor);

    meteor.addEventListener("animationend", () => {
      meteor.remove();
      createMeteor();
    });
  }

  // เริ่มสร้างดาวตกและฝนดาวตกหลายดวง
  for(let i=0; i<8; i++) createShootingStar();
  for(let i=0; i<20; i++) createMeteor();

  // --- Firebase setup and logic ---
  const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_AUTH_DOMAIN",
    projectId: "YOUR_PROJECT_ID",
    storageBucket: "YOUR_BUCKET",
    messagingSenderId: "YOUR_SENDER_ID",
    appId: "YOUR_APP_ID"
  };

  firebase.initializeApp(firebaseConfig);
  const auth = firebase.auth();
  const db = firebase.firestore();

  // DOM
  const authSection = document.getElementById("auth-section");
  const userSection = document.getElementById("user-section");
  const userEmailSpan = document.getElementById("userEmail");
  const userCreditSpan = document.getElementById("userCredit");
  const historyList = document.getElementById("historyList");
  const redeemMsg = document.getElementById("redeemMsg");

  const emailInput = document.getElementById("email");
  const passwordInput = document.getElementById("password");
  const codeInput = document.getElementById("codeInput");

  const loginBtn = document.getElementById("loginBtn");
  const registerBtn = document.getElementById("registerBtn");
  const logoutBtn = document.getElementById("logoutBtn");
  const rollBtn = document.getElementById("rollBtn");
  const redeemBtn = document.getElementById("redeemBtn");

  let currentUser = null;
  let userCredit = 0;
  let userHistory = [];

  // แสดง/ซ่อนส่วนตามสถานะล็อกอิน
  function showUser
