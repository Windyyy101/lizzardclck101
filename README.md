<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Secure Clicker Game - Marketplace</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1-crypto-js.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 0;
            padding: 0;
            background-color: #f4f4f9;
        }
        #login-section, #game-section {
            margin-top: 20px;
        }
        #marketplace ul {
            list-style: none;
            padding: 0;
        }
        #marketplace li {
            margin: 10px 0;
        }
        button {
            padding: 10px 20px;
            font-size: 16px;
            cursor: pointer;
            margin: 5px;
        }
    </style>
</head>
<body>
    <h1>Clicker Game - Secure Marketplace</h1>

    <!-- Login Section -->
    <div id="login-section">
        <p>Enter your email to login:</p>
        <input type="email" id="email-input" placeholder="Enter your email" required>
        <button id="login-button">Login</button>
    </div>

    <!-- Game Section (Visible After Login) -->
    <div id="game-section" style="display: none;">
        <p>Tokens in your wallet: <span id="token-count">0</span></p>
        <p>Skins Available:</p>
        <div id="marketplace">
            <ul id="marketplace-list"></ul>
        </div>
    </div>

    <script>
        const totalTokens = 10000000000; // Total available tokens in the game
        let tokenCount = 0;
        let currentUser = null; // Current logged-in user

        // Szyfrowanie danych
        function encryptData(data, secretKey) {
            return CryptoJS.AES.encrypt(JSON.stringify(data), secretKey).toString();
        }

        // Deszyfrowanie danych
        function decryptData(encryptedData, secretKey) {
            const bytes = CryptoJS.AES.decrypt(encryptedData, secretKey);
            return JSON.parse(bytes.toString(CryptoJS.enc.Utf8));
        }

        // Save encrypted user data to localStorage
        function saveUserData(email, data) {
            const secretKey = "mySecretKey"; // Use a secure key for encryption
            const encryptedData = encryptData(data, secretKey);
            localStorage.setItem(email, encryptedData);
        }

        // Load encrypted user data from localStorage
        function loadUserData(email) {
            const secretKey = "mySecretKey"; // Use the same secure key for decryption
            const encryptedData = localStorage.getItem(email);
            if (encryptedData) {
                return decryptData(encryptedData, secretKey);
            } else {
                return { skins: [], tokens: 0 }; // Default user data if not found
            }
        }

        // Marketplace and auction data (example data for now)
        const marketplace = [
            { id: 1, rarity: 'rare', price: 200, seller: 'user1@example.com' },
            { id: 2, rarity: 'ultraRare', price: 500, seller: 'user2@example.com' }
        ];

        // Function to update marketplace display
        function updateMarketplace() {
            const marketplaceList = document.getElementById('marketplace-list');
            marketplaceList.innerHTML = marketplace.map(item => `
                <li>
                    <span>${item.rarity} Skin - ${item.price} Tokens</span>
                    <button onclick="buySkin(${item.id})">Buy</button>
                </li>
            `).join('');
        }

        // Function to handle skin purchase
        function buySkin(skinId) {
            const item = marketplace.find(skin => skin.id === skinId);
            if (tokenCount >= item.price) {
                tokenCount -= item.price;
                document.getElementById('token-count').textContent = tokenCount;
                alert(`You purchased a ${item.rarity} Skin!`);
            } else {
                alert('Not enough tokens!');
            }
        }

        // Handle user login
        document.getElementById('login-button').addEventListener('click', () => {
            const email = document.getElementById('email-input').value.trim().toLowerCase();
            if (!email) {
                alert('Please enter a valid email address');
                return;
            }
            currentUser = email;
            let userData = loadUserData(email);

            // If it's the first time user, give them tokens
            if (!userData.tokens) {
                userData.tokens = totalTokens * 0.01; // First user gets 0.01 of total tokens
                alert(`Welcome! You received 0.01 of the total tokens: ${userData.tokens}`);
            }

            tokenCount = userData.tokens;
            document.getElementById('token-count').textContent = tokenCount;

            // Save user data
            saveUserData(currentUser, userData);

            // Show game section and hide login section
            document.getElementById('login-section').style.display = 'none';
            document.getElementById('game-section').style.display = 'block';

            // Update marketplace display
            updateMarketplace();
        });
    </script>
</body>
</html>

