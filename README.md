<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Guerre de Territoire</title>
    <style>
        body { text-align: center; font-family: Arial, sans-serif; background-color: black; color: white; }
        .map { display: grid; grid-template-columns: repeat(10, 40px); gap: 2px; justify-content: center; margin-top: 10px; border: 3px solid white; padding: 5px; border-radius: 10px; }
        .territory { width: 40px; height: 40px; display: flex; align-items: center; justify-content: center; font-size: 10px; cursor: pointer; transition: background-color 0.5s ease, transform 0.3s ease; position: relative; border: 1px solid white; }
        .territory img { width: 80%; height: 80%; border-radius: 50%; position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); }
        .territory:hover { transform: scale(1.1); }
        .territory-animation { animation: conquer 1s ease-out; }
        .territory.conquered { box-shadow: 0 0 10px white; }
        @keyframes conquer { from { transform: scale(1.5); opacity: 0; } to { transform: scale(1); opacity: 1; } }
        .countdown { font-size: 24px; margin-top: 10px; font-weight: bold; color: yellow; }
        .leaderboard, .teams, .hall-of-fame, .top-players { margin-top: 10px; text-align: left; font-size: 16px; background: rgba(255, 255, 255, 0.1); padding: 10px; border-radius: 10px; }
        .config { margin: 20px 0; }
        .config input, .config button { margin: 5px; padding: 10px; }
    </style>
</head>
<body>
    <h2 id="rivalry-title">Guerre de Territoire</h2>
    <div class="config">
        <label for="team-number">Nombre d'équipes:</label>
        <input type="number" id="team-number" value="4" min="2" max="5">
        <label for="rivalry">Rivalité:</label>
        <input type="text" id="rivalry" placeholder="Ex: Paris vs Marseille">
        <button onclick="setupGame()">Lancer la partie</button>
    </div>
    <div class="countdown" id="countdown">Temps restant : 30:00</div>
    <div class="map" id="map"></div>
    <div class="leaderboard" id="leaderboard"></div>
    <div class="top-players" id="top-players"></div>
    <div class="teams" id="teams"></div>
    <div class="hall-of-fame" id="hall-of-fame"></div>
    
    <script>
        let teams = {};
        let territories = Array(100).fill(null);
        let customTeams = {};
        let avatars = {};
        let gameTime = 30 * 60;
        let topPlayers = [];
        let audioCapture = new Audio('https://www.fesliyanstudios.com/play-mp3/4381');

        function setupGame() {
            let teamNumber = document.getElementById("team-number").value;
            let rivalry = document.getElementById("rivalry").value;
            document.getElementById("rivalry-title").textContent = rivalry || "Guerre de Territoire";
            
            let colors = ["gold", "orange", "cyan", "purple", "red"];
            customTeams = {};
            teams = {};

            for (let i = 0; i < teamNumber; i++) {
                customTeams[`Équipe ${i+1}`] = colors[i];
                teams[`Équipe ${i+1}`] = 0;
            }
            
            territories.fill(null);
            generateMap();
        }

        function startCountdown() {
            setInterval(() => {
                if (gameTime > 0) {
                    gameTime--;
                    updateCountdown();
                } else {
                    endGame();
                }
            }, 1000);
        }

        function updateCountdown() {
            let minutes = Math.floor(gameTime / 60);
            let seconds = gameTime % 60;
            document.getElementById("countdown").textContent = `Temps restant : ${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
        }

        function registerPlayer(player, amount, team, avatarUrl) {
            if (!teams[team]) teams[team] = 0;
            teams[team] += amount;
            avatars[player] = avatarUrl;
            topPlayers.push({ player, amount });
            topPlayers.sort((a, b) => b.amount - a.amount);
            claimTerritory(team, player);
            updateLeaderboard();
            updateTopPlayers();
        }

        function claimTerritory(team, player) {
            let available = territories.map((t, i) => t === null ? i : null).filter(i => i !== null);
            if (available.length > 0) {
                let chosenIndex = available[Math.floor(Math.random() * available.length)];
                territories[chosenIndex] = { team, player };
                animateTerritory(chosenIndex);
                audioCapture.play();
            }
            generateMap();
        }

        function generateMap() {
            const map = document.getElementById("map");
            map.innerHTML = "";
            
            territories.forEach((cell, index) => {
                const div = document.createElement("div");
                div.className = "territory";
                div.style.backgroundColor = cell ? customTeams[cell.team] : "lightgray";
                
                if (cell && avatars[cell.player]) {
                    let img = document.createElement("img");
                    img.src = avatars[cell.player];
                    div.appendChild(img);
                }
                
                if (cell) div.classList.add("conquered");
                map.appendChild(div);
            });
        }

        function animateTerritory(index) {
            const map = document.getElementById("map").children[index];
            map.classList.add("territory-animation");
            setTimeout(() => map.classList.remove("territory-animation"), 1000);
        }

        setupGame();
        startCountdown();
    </script>
</body>
</html>

