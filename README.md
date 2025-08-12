# jogo-da-memoria
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Jogo da Memória</title>
  <link rel="stylesheet" href="css/style.css" />
</head>
<body>
  <!-- Tela inicial com nome do jogador -->
  <div id="startScreen">
    <h1>Jogo da Memória</h1>
    <input type="text" id="playerNameInput" placeholder="Digite seu nome" />
    <select id="difficulty">
      <option value="4">Fácil (4x4)</option>
      <option value="6">Médio (6x6)</option>
      <option value="8">Difícil (8x8)</option>
    </select>
    <button onclick="startGame()">Iniciar Jogo</button>
  </div>

  <!-- Tabuleiro do jogo -->
  <div id="gameContainer" style="display: none;">
    <h2 id="welcomeMessage"></h2>
    <p id="score">Pontuação: 0</p>
    <div id="gameBoard"></div>
    <p id="message"></p>
  </div>

  <!-- Tela de vitória -->
  <div id="victoryScreen" style="display: none;">
    <h2>Parabéns! Você venceu! 🏆</h2>
    <p id="finalScore"></p>
    <h3>Top 5 Pontuações</h3>
    <ol id="highScoresList"></ol>
    <button onclick="location.reload()">Jogar Novamente</button>
  </div>

  <script src="js/script.js"></script>
</body>
</html>
body {
  font-family: 'Segoe UI', sans-serif;
  background: #f0f0f0;
  margin: 0;
  padding: 20px;
  text-align: center;
}

#gameBoard {
  display: grid;
  gap: 10px;
  justify-content: center;
  margin: 20px auto;
}

.card {
  width: 80px;
  height: 80px;
  background-color: #444;
  color: white;
  font-size: 2rem;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  border-radius: 8px;
  user-select: none;
  transition: background-color 0.3s ease;
}

.card.flipped {
  background-color: #fff;
  color: #000;
}

.card.matched {
  background-color: #4CAF50;
  color: white;
  cursor: default;
}

#startScreen,
#victoryScreen {
  max-width: 400px;
  margin: auto;
}

input, select, button {
  padding: 10px;
  margin-top: 10px;
  width: 100%;
  font-size: 1rem;
}
const gameBoard = document.getElementById('gameBoard');
const message = document.getElementById('message');
const scoreElement = document.getElementById('score');
const victoryScreen = document.getElementById('victoryScreen');
const finalScore = document.getElementById('finalScore');
const highScoresList = document.getElementById('highScoresList');

let playerName = '';
let gridSize = 4;
let firstCard = null;
let secondCard = null;
let lockBoard = false;
let matchedPairs = 0;
let score = 0;
let totalPairs = 0;
let cards = [];

function startGame() {
  playerName = document.getElementById('playerNameInput').value.trim();
  gridSize = parseInt(document.getElementById('difficulty').value);

  if (!playerName) {
    alert('Por favor, insira seu nome.');
    return;
  }

  totalPairs = (gridSize * gridSize) / 2;
  score = 0;
  matchedPairs = 0;
  firstCard = null;
  secondCard = null;
  lockBoard = false;

  document.getElementById('startScreen').style.display = 'none';
  document.getElementById('gameContainer').style.display = 'block';
  document.getElementById('welcomeMessage').textContent = `Boa sorte, ${playerName}!`;

  scoreElement.textContent = 'Pontuação: 0';
  message.textContent = '';
  setupBoard();
}

async function setupBoard() {
  gameBoard.innerHTML = '';
  gameBoard.style.gridTemplateColumns = `repeat(${gridSize}, 1fr)`;

  const uniqueImages = await fetchImages(totalPairs);
  const allImages = [...uniqueImages, ...uniqueImages]; // duplicar para pares
  shuffleArray(allImages);

  cards = allImages.map((img, index) => {
    const card = document.createElement('div');
    card.classList.add('card');
    card.dataset.image = img;
    card.dataset.index = index;
    card.textContent = '?';
    card.addEventListener('click', onCardClick);
    return card;
  });

  cards.forEach(card => gameBoard.appendChild(card));
}

async function fetchImages(count) {
  const imageUrls = [];
  while (imageUrls.length < count) {
    const id = Math.floor(Math.random() * 1000); // Random image
    const url = `https://picsum.photos/id/${id}/80`;
    if (!imageUrls.includes(url)) {
      imageUrls.push(url);
    }
  }
  return imageUrls;
}

function onCardClick(e) {
  const card = e.currentTarget;
  if (lockBoard || card.classList.contains('flipped') || card.classList.contains('matched')) return;

  flipCard(card);

  if (!firstCard) {
    firstCard = card;
  } else {
    secondCard = card;
    checkMatch();
  }
}

function flipCard(card) {
  const img = document.createElement('img');
  img.src = card.dataset.image;
  img.style.width = '100%';
  img.style.height = '100%';
  card.textContent = '';
  card.appendChild(img);
  card.classList.add('flipped');
}

function unflipCards(c1, c2) {
  setTimeout(() => {
    c1.textContent = '?';
    c2.textContent = '?';
    c1.classList.remove('flipped');
    c2.classList.remove('flipped');
    c1.innerHTML = '';
    c2.innerHTML = '';
    resetTurn();
  }, 1000);
}

function checkMatch() {
  lockBoard = true;
  if (firstCard.dataset.image === secondCard.dataset.image) {
    firstCard.classList.add('matched');
    secondCard.classList.add('matched');
    matchedPairs++;
    score += 5;
    scoreElement.textContent = `Pontuação: ${score}`;
    resetTurn();
    if (matchedPairs === totalPairs) {
      endGame();
    }
  } else {
    score -= 3;
    scoreElement.textContent = `Pontuação: ${score}`;
    unflipCards(firstCard, secondCard);
  }
}

function resetTurn() {
  [firstCard, secondCard] = [null, null];
  lockBoard = false;
}

function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
}

function endGame() {
  document.getElementById('gameContainer').style.display = 'none';
  victoryScreen.style.display = 'block';
  finalScore.textContent = `Sua pontuação: ${score}`;
  saveHighScore(playerName, score);
  showHighScores();
}

function saveHighScore(name, score) {
  let scores = JSON.parse(localStorage.getItem('highScores')) || [];
  scores.push({ name, score });
  scores.sort((a, b) => b.score - a.score);
  scores = scores.slice(0, 5);
  localStorage.setItem('highScores', JSON.stringify(scores));
}

function showHighScores() {
  const scores = JSON.parse(localStorage.getItem('highScores')) || [];
  highScoresList.innerHTML = '';
  scores.forEach(s => {
    const li = document.createElement('li');
    li.textContent = `${s.name} - ${s.score} pts`;
    highScoresList.appendChild(li);
  });
}
