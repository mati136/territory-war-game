<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Guerre de Territoire</title>
    <style>
        body { text-align: center; font-family: Arial, sans-serif; background-color: black; color: white; }
        .map { display: grid; grid-template-columns: repeat(5, 60px); gap: 5px; justify-content: center; margin-top: 10px; }
        .territory { width: 60px; height: 60px; background-color: lightgray; display: flex; align-items: center; justify-content: center; font-size: 12px; cursor: pointer; transition: background-color 0.5s ease, transform 0.3s ease; }
        .territory:hover { transform: scale(1.1); }
        .countdown { font-size: 20px; margin-top: 10px; }
        .leaderboard, .teams, .hall-of-fame { margin-top: 10px; text-align: left; font-size: 14px; }
        .top-donors { margin-top: 10px; }
        .territory-animation { animation: conquer 1s ease-out; }
        @keyframes conquer { from { transform: scale(1.5); opacity: 0; } to { transform: scale(1); opacity: 1; } }
    </style>
</head>
<body>
    <h2>Guerre de Territoire</h2>
    <div class="countdown" id="countdown">Temps restant : 30:00</div>
    <div class="map" id="map"></div>
    <div class="leaderboard" id="leaderboard"></div>
    <div class="top-donors" id="top-donors"></div>
    <div class="teams" id="teams"></div>
    <div class="hall-of-fame" id="hall-of-fame"></div>
    
    <script>
        let teams = {};
        let territories = Array(5).fill(null);
        let teamColors = {};
        let gameTime = 30 * 60;
        let finalPhase = false;
        let hallOfFame = JSON.parse(localStorage.getItem("hallOfFame")) || [];
        let topDonors = [];
        let audioCapture = new Audio('https://www.fesliyanstudios.com/play-mp3/4381');

        function startCountdown() {
            setInterval(() => {
                if (gameTime > 0) {
                    gameTime--;
                    updateCountdown();
                } else {
                    endGame();
                }
                if (gameTime <= 300 && !finalPhase) {
                    finalPhase = true;
                    alert("Phase finale ! Tous les dons comptent double !");
                }
            }, 1000);
        }

        function updateCountdown() {
            let minutes = Math.floor(gameTime / 60);
            let seconds = gameTime % 60;
            document.getElementById("countdown").textContent = `Temps restant : ${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
        }
        
        function registerDonation(donor, amount, color) {
            if (!teams[donor] || teams[donor] < amount) {
                teams[donor] = amount;
                if (!teamColors[donor] && !Object.values(teamColors).includes(color)) {
                    teamColors[donor] = color;
                }
            }
            
            topDonors.push({ donor, amount });
            topDonors.sort((a, b) => b.amount - a.amount);
            
            claimTerritory(donor, amount);
            updateLeaderboard();
            updateTeams();
            updateTopDonors();
        }
        
        function claimTerritory(donor, amount) {
            let numTerritories = Math.min(Math.ceil(amount / 5000), 5);
            let claimed = 0;
            let sortedTerritories = territories.map((owner, index) => ({ index, owner, donation: owner ? teams[owner] : 0 }))
                                             .sort((a, b) => a.donation - b.donation);
            for (let i = 0; i < sortedTerritories.length && claimed < numTerritories; i++) {
                if (!sortedTerritories[i].owner || teams[sortedTerritories[i].owner] < amount) {
                    territories[sortedTerritories[i].index] = donor;
                    animateTerritory(sortedTerritories[i].index);
                    audioCapture.play();
                    claimed++;
                }
            }
            generateMap();
        }
        
        function generateMap() {
            const map = document.getElementById("map");
            map.innerHTML = "";
            
            territories.forEach((team, index) => {
                const div = document.createElement("div");
                div.className = "territory";
                div.style.backgroundColor = team && teamColors[team] ? teamColors[team] : "lightgray";
                div.textContent = team ? team[0] : "⬜"; 
                map.appendChild(div);
            });
        }
        
        function animateTerritory(index) {
            const map = document.getElementById("map").children[index];
            map.classList.add("territory-animation");
            setTimeout(() => map.classList.remove("territory-animation"), 1000);
        }
        
        function updateLeaderboard() {
            const leaderboard = document.getElementById("leaderboard");
            leaderboard.innerHTML = "<h3>Classement</h3>";
            let sortedTeams = Object.entries(teams).sort((a, b) => b[1] - a[1]);
            sortedTeams.forEach(([name, amount]) => {
                let entry = document.createElement("div");
                entry.textContent = `${name}: ${amount} coins`;
                leaderboard.appendChild(entry);
            });
        }
        
        function updateTopDonors() {
            const topDonorsDisplay = document.getElementById("top-donors");
            topDonorsDisplay.innerHTML = "<h3>Top Donateurs</h3>";
            topDonors.slice(0, 3).forEach(({ donor, amount }) => {
                let entry = document.createElement("div");
                entry.textContent = `${donor} : ${amount} coins`;
                topDonorsDisplay.appendChild(entry);
            });
        }
        
        generateMap();
        updateCountdown();
        startCountdown();
        updateTopDonors();
    </script>
</body>
</html>
