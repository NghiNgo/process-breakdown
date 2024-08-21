## Step 1: Set up the development environment

1. Install Node.js and npm (Node Package Manager)

2. Set up the development environment:

```bash
npm install -g ganache
```
This command installs Ganache, a tool for creating a local Ethereum blockchain for development and testing purposes.

```bash
mkdir music-rights-blockchain
```
This command creates a new directory for the project.

```bash
cd music-rights-blockchain
```
This command changes the current directory to the newly created project directory.

## Step 2: Install the smart contract directory

1. Initialize the smart contract directory in the main music-rights-blockchain directory

```bash
mkdir smartcontract
```
This command creates a new directory for the smart contract.

```bash
cd smartcontract
```
This command changes the current directory to the smart contract directory. - TypeScript

```bash
npx hardhat
```
This command initializes a new Hardhat project for smart contract development.

2. Write Smart Contract: (smart-contract directory)

Create a file `MusicRights.sol` and remove the file `Lock.sol` from the `contracts` directory:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract MusicRights {
    struct Song {
        string title;
        address artist;
        uint256 playCount;
        uint256 royaltyPerPlay;
    }

    mapping(uint256 => Song) public songs;
    uint256 public songCount;

    event SongRegistered(uint256 indexed id, string title, address artist);
    event SongPlayed(uint256 indexed id, address listener);

    function registerSong(string memory _title, uint256 _royaltyPerPlay) public {
        songCount++;
        songs[songCount] = Song(_title, msg.sender, 0, _royaltyPerPlay);
        emit SongRegistered(songCount, _title, msg.sender);
    }

    function playSong(uint256 _id) public payable {
        Song storage song = songs[_id];
        require(msg.value >= song.royaltyPerPlay, "Insufficient payment");
        
        song.playCount++;
        payable(song.artist).transfer(song.royaltyPerPlay);
        emit SongPlayed(_id, msg.sender);
        
        if (msg.value > song.royaltyPerPlay) {
            payable(msg.sender).transfer(msg.value - song.royaltyPerPlay);
        }
    }

    function getSong(uint256 _id) public view returns (string memory, address, uint256, uint256) {
        Song memory song = songs[_id];
        return (song.title, song.artist, song.playCount, song.royaltyPerPlay);
    }
}
```

3. Modify the Hardhat configuration file (`hardhat.config.ts`) to use the local network:

```typescript
import { HardhatUserConfig } from "hardhat/config";
import "@nomicfoundation/hardhat-toolbox";

const config: HardhatUserConfig = {
  solidity: "0.8.19",
  networks: {
    hardhat: {
      chainId: 1337,
    },
  },
};

export default config;
```

4. Create a deployment script in `scripts/deploy.ts`:

```typescript
import { ethers } from "hardhat";

async function main() {
  const MusicRights = await ethers.getContractFactory("MusicRights");
  const musicRights = await MusicRights.deploy();

  await musicRights.waitForDeployment();

  console.log("Copyright contract deployed to:", await musicRights.getAddress());
}

main()
  .then(() => process.exit(0))
  .catch((error) => {
    console.error(error);
    process.exit(1);
  });
```

5. Run the local Ethereum network:
```bash
npx hardhat node
```
This command starts a local Ethereum network for testing and development.

6. Save the Account and Private Key
```bash
Account #0: 0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266 (10000 ETH)
Private Key: 0xac0974bec39a17e36ba4a6b4d238ff944bacb478cbed5efcae784d7bf4f2ff80

Account #1: 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (10000 ETH)
Private Key: 0x59c6995e998f97a5a0044966f0945389dc9e86dae88c7a8412f4603b6b78690d

Account #2: 0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC (10000 ETH)
Private Key: 0x5de4111afa1a4b94908f83103eb1f1706367c2e68ca870fc3fb9a804cdab365a

Account #3: 0x90F79bf6EB2c4f870365E785982E1f101E93b906 (10000 ETH)
Private Key: 0x7c852118294e51e653712a81e05800f419141751be58f605c371e15141b007a6

Account #4: 0x15d34AAf54267DB7D7c367839AAf71A00a2C6A65 (10000 ETH)
Private Key: 0x47e179ec197488593b187f80a00eb0da91f1b9d0b13f8733639f19c30a34926a

Account #5: 0x9965507D1a55bcC2695C58ba16FB37d819B0A4dc (10000 ETH)
Private Key: 0x8b3a350cf5c34c9194ca85829a2df0ec3153be0318b5e2d3348e872092edffba

Account #6: 0x976EA74026E726554dB657fA54763abd0C3a0aa9 (10000 ETH)
Private Key: 0x92db14e403b83dfe3df233f83dfa3a0d7096f21ca9b0d6d6b8d88b2b4ec1564e

Account #7: 0x14dC79964da2C08b23698B3D3cc7Ca32193d9955 (10000 ETH)
Private Key: 0x4bbbf85ce3377467afe5d46f804f221813b2bb87f24d81f60f1fcdbf7cbf4356

Account #8: 0x23618e81E3f5cdF7f54C3d65f7FBc0aBf5B21E8f (10000 ETH)
Private Key: 0xdbda1821b80551c9d65939329250298aa3472ba22feea921c0cf5d620ea67b97

Account #9: 0xa0Ee7A142d267C1f36714E4a8F75612F20a79720 (10000 ETH)
Private Key: 0x2a871d0798f97d79848a013d4936a73bf4cc922c825d33c1cf7073dff6d409c6

Account #10: 0xBcd4042DE499D14e55001CcbB24a551F3b954096 (10000 ETH)
Private Key: 0xf214f2b2cd398c806f84e317254e0f0b801d0643303237d97a22a48e01628897

Account #11: 0x71bE63f3384f5fb98995898A86B02Fb2426c5788 (10000 ETH)
Private Key: 0x701b615bbdfb9de65240bc28bd21bbc0d996645a3dd57e7b12bc2bdf6f192c82

Account #12: 0xFABB0ac9d68B0B445fB7357272Ff202C5651694a (10000 ETH)
Private Key: 0xa267530f49f8280200edf313ee7af6b827f2a8bce2897751d06a843f644967b1

Account #13: 0x1CBd3b2770909D4e10f157cABC84C7264073C9Ec (10000 ETH)
Private Key: 0x47c99abed3324a2707c28affff1267e45918ec8c3f20b8aa892e8b065d2942dd

Account #14: 0xdF3e18d64BC6A983f673Ab319CCaE4f1a57C7097 (10000 ETH)
Private Key: 0xc526ee95bf44d8fc405a158bb884d9d1238d99f0612e9f33d006bb0789009aaa

Account #15: 0xcd3B766CCDd6AE721141F452C550Ca635964ce71 (10000 ETH)
Private Key: 0x8166f546bab6da521a8369cab06c5d2b9e46670292d85c875ee9ec20e84ffb61

Account #16: 0x2546BcD3c84621e976D8185a91A922aE77ECEc30 (10000 ETH)
Private Key: 0xea6c44ac03bff858b476bba40716402b03e41b8e97e276d1baec7c37d42484a0

Account #17: 0xbDA5747bFD65F08deb54cb465eB87D40e51B197E (10000 ETH)
Private Key: 0x689af8efa8c651a91ad287602527f3af2fe9f6501a7ac4b061667b5a93e037fd

Account #18: 0xdD2FD4581271e230360230F9337D5c0430Bf44C0 (10000 ETH)
Private Key: 0xde9be858da4a475276426320d5e9262ecfc3ba460bfac56360bfa6c4c28b4ee0

Account #19: 0x8626f6940E2eb28930eFb4CeF49B2d1F2C9C1199 (10000 ETH)
Private Key: 0xdf57089febbacf7ba0bc227dafbffa9fc08a93fdc68e1e42411a14efcf23656e
```

7. Deploy the contract:
```bash
npx hardhat run scripts/deploy.ts --network localhost
```
This command deploys the MusicRights contract to the local Ethereum network.

8. Save the deployed contract address:
```bash
0x5FbDB2315678afecb367f032d93F642f64180aa3
```
This is the address of the deployed MusicRights contract on the local Ethereum network.

## Step 3: Setting up the backend directory (in the main music-rights-blockchain directory)

1. Creating an Express.js backend directory

```bash
mkdir backend
```
```bash
cd backend
```
```bash
npm init -y
```
```bash
npm install express cors body-parser web3
```

2. Building the Express.js backend: (in the backend directory) -> Remember to replace the deployed contract address from Step 2.8

Create a file `app.js` in the `backend` directory:

```javascript
const express = require("express");
const cors = require("cors");
const bodyParser = require("body-parser");
const { Web3 } = require("web3");

const app = express();
app.use(cors());
app.use(bodyParser.json());

const web3 = new Web3("http://127.0.0.1:8545");
const contractABI = require("../smartcontract/artifacts/contracts/MusicRights.sol/MusicRights.json").abi;
const contractAddress = "0x5FbDB2315678afecb367f032d93F642f64180aa3"; // The address of the deployed contract

const contract = new web3.eth.Contract(contractABI, contractAddress);

app.post("/register-song", async (req, res) => {
  const { title, royaltyPerPlay, from } = req.body;
  try {
    const gasEstimate = await contract.methods
      .registerSong(title, royaltyPerPlay)
      .estimateGas({ from });

    const result = await contract.methods
      .registerSong(title, royaltyPerPlay)
      .send({ from, gas: gasEstimate });

    // Extract the song ID from the SongRegistered event
    const songId = result.events.SongRegistered.returnValues.songId;

    // Convert BigInt values to strings
    const serializedResult = JSON.parse(
      JSON.stringify(result, (key, value) =>
        typeof value === "bigint" ? value.toString() : value
      )
    );

    res.json({ ...serializedResult, songId });
  } catch (error) {
    console.error("Error details:", error);
    res.status(500).json({
      error: error.message,
      details: error.data || "No additional details available",
    });
  }
});

app.post("/play-song", async (req, res) => {
  const { id, from } = req.body;
  try {
    // First, get the song details to check if it exists and its royalty
    const song = await contract.methods.getSong(id).call();

    if (!song[0]) {
      return res.status(404).json({ error: "Song not found" });
    }

    const royaltyPerPlay = song[3];
    const value = royaltyPerPlay;

    // Convert value to BigInt before comparison
    if (BigInt(value) < BigInt(royaltyPerPlay)) {
      return res.status(400).json({
        error: "Insufficient payment",
        message: `Required: ${royaltyPerPlay}, Provided: ${value}`,
      });
    }

    const gasEstimate = await contract.methods
      .playSong(id)
      .estimateGas({ from, value });
    console.log("Estimated gas:", gasEstimate);

    const result = await contract.methods
      .playSong(id)
      .send({ from, value, gas: gasEstimate });

    const serializedResult = JSON.parse(
      JSON.stringify(result, (key, value) =>
        typeof value === "bigint" ? value.toString() : value
      )
    );

    res.json(serializedResult);
  } catch (error) {
    console.error("Error details:", error);
    res.status(500).json({
      error: "Transaction failed",
      message: error.message,
      details: error.data
        ? JSON.stringify(error.data)
        : "No additional details available",
      params: { id, value: royaltyPerPlay, from },
    });
  }
});

app.get("/song/:id", async (req, res) => {
  const { id } = req.params;
  try {
    const song = await contract.methods.getSong(id).call();
    
    // Check if the song exists (assuming an empty title means the song doesn't exist)
    if (!song[0]) {
      return res.status(404).json({ error: "Song not found" });
    }

    const serializedSong = {
      id: id,
      title: song[0],
      artist: song[1],
      playCount: BigInt(song[2] || 0).toString(),
      royaltyPerPlay: BigInt(song[3] || 0).toString(),
    };
    res.json(serializedSong);
  } catch (error) {
    console.error("Error fetching song:", error);
    res.status(500).json({ error: "Failed to fetch song details" });
  }
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

3. Starting the backend application: (in the backend directory)

```bash
node app.js
```

## Step 4: Setting up the Frontend Directory (in the main music-rights-blockchain directory)

1. Create a frontend directory using Next.js - Comments: `TypeScript` - No, `ESLint` - No, `Tailwind CSS` - Yes, `src/` - No, `App Router` - Yes, `import alias` - No

```bash
npx create-next-app@latest frontend 
```
```bash
cd frontend
```
```bash
npm install -D tailwindcss postcss autoprefixer
```

2. Delete the frontend/app/page.js file

3. Create a new index.js file in the new pages directory <- setAccount is obtained from Step 2, Section 4

```jsx
// pages/index.js
// pages/index.js
import { useState, useEffect } from "react";

export default function Home() {
  const [account, setAccount] = useState("");
  const [songTitle, setSongTitle] = useState("");
  const [royaltyPerPlay, setRoyaltyPerPlay] = useState("");
  const [songId, setSongId] = useState("");
  const [songDetails, setSongDetails] = useState(null);
  const [registrationStatus, setRegistrationStatus] = useState("");
  const [errorMessage, setErrorMessage] = useState("");
  const [playError, setPlayError] = useState("");

  useEffect(() => {
    setAccount("account_Step_2_6");
  }, []);

  const fetchSongDetails = async () => {
    try {
      const response = await fetch(`http://localhost:3001/song/${songId}`);
      const result = await response.json();
      if (response.ok) {
        setSongDetails(result);
        setErrorMessage("");
      } else {
        setSongDetails(null);
        setErrorMessage(result.error || "Failed to fetch song details");
      }
    } catch (error) {
      console.error("Error fetching song details:", error);
      setSongDetails(null);
      setErrorMessage("An error occurred while fetching song details");
    }
  };

  const registerSong = async () => {
    try {
      const response = await fetch("http://localhost:3001/register-song", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          title: songTitle,
          royaltyPerPlay,
          from: account,
        }),
      });
      const result = await response.json();
      console.log("Song registered:", result);
      setRegistrationStatus(
        `Song "${songTitle}" (ID: ${result.events.SongRegistered.returnValues.id}) has been registered successfully!
      Artist: ${result.events.SongRegistered.returnValues.artist}`
      );
      setSongTitle("");
      setRoyaltyPerPlay("");
    } catch (error) {
      console.error("Error registering song:", error);
    }
  };

  const playSong = async () => {
    try {
      const response = await fetch("http://localhost:3001/play-song", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ id: songId, from: account }),
      });
      const result = await response.json();

      if (response.ok) {
        console.log("Song played:", result);
        setSongId("");
        setPlayError("");
      } else {
        setPlayError(result.error || result.message || "Failed to play song");
      }
    } catch (error) {
      console.error("Error playing song:", error);
      setPlayError("An error occurred while playing the song");
    }
  };

  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-6">Music Rights Blockchain</h1>
      <p className="mb-8">
        Your account: <span className="font-mono">{account}</span>
      </p>

      <div className="mb-8">
        <h2 className="text-2xl font-semibold mb-4">Register a Song</h2>
        <div className="flex space-x-4">
          <input
            type="text"
            placeholder="Song Title"
            value={songTitle}
            onChange={(e) => setSongTitle(e.target.value)}
            className="flex-grow px-3 py-2 border rounded"
          />
          <input
            type="number"
            placeholder="Royalty per Play (in Wei)"
            value={royaltyPerPlay}
            onChange={(e) => setRoyaltyPerPlay(e.target.value)}
            className="flex-grow px-3 py-2 border rounded"
          />
          <button
            onClick={registerSong}
            className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
          >
            Register Song
          </button>
        </div>
        {registrationStatus && (
          <p className="mt-2 text-green-600">{registrationStatus}</p>
        )}
      </div>

      <div className="mb-8">
        <h2 className="text-2xl font-semibold mb-4">Play a Song</h2>
        <div className="flex space-x-4">
          <input
            type="number"
            placeholder="Song ID"
            value={songId}
            onChange={(e) => setSongId(e.target.value)}
            className="flex-grow px-3 py-2 border rounded"
          />
          <button
            onClick={playSong}
            className="px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600"
          >
            Play Song
          </button>
        </div>
        {playError && <p className="mt-2 text-red-600">{playError}</p>}
      </div>

      <div>
        <h2 className="text-2xl font-semibold mb-4">Song Details</h2>
        <div className="flex space-x-4 mb-4">
          <input
            type="number"
            placeholder="Song ID"
            value={songId}
            onChange={(e) => setSongId(e.target.value)}
            className="flex-grow px-3 py-2 border rounded"
          />
          <button
            onClick={fetchSongDetails}
            className="px-4 py-2 bg-purple-500 text-white rounded hover:bg-purple-600"
          >
            Fetch Song Details
          </button>
        </div>
        {errorMessage && <p className="text-red-600 mb-4">{errorMessage}</p>}
        {songDetails && (
          <div className="bg-gray-100 p-4 rounded">
            <p>
              <span className="font-semibold">Title:</span> {songDetails.title}
            </p>
            <p>
              <span className="font-semibold">Artist:</span>{" "}
              {songDetails.artist}
            </p>
            <p>
              <span className="font-semibold">Royalty per Play:</span>{" "}
              {songDetails.royaltyPerPlay} Wei
            </p>
            <p>
              <span className="font-semibold">Total Plays:</span>{" "}
              {songDetails.playCount}
            </p>
            <p>
              <span className="font-semibold">Total Royalties:</span>{" "}
              {songDetails.royaltyPerPlay * songDetails.playCount} Wei
            </p>
          </div>
        )}
      </div>
    </div>
  );
}
```

4. Create a new _app.js file in the pages directory:
```js
import "../app/globals.css";

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}

export default MyApp;
```

5. Update the postcss.config.mjs file:
```mjs
/** @type {import('postcss-load-config').Config} */
const config = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

export default config;
```

6. Update the next.config.mjs file:

```mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
};

export default nextConfig;
```

7. Update the app/globals.css file:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --foreground-rgb: 0, 0, 0;
  --background-start-rgb: 214, 219, 220;
  --background-end-rgb: 255, 255, 255;
}

@media (prefers-color-scheme: dark) {
  :root {
    --foreground-rgb: 255, 255, 255;
    --background-start-rgb: 0, 0, 0;
    --background-end-rgb: 0, 0, 0;
  }
}

body {
  @apply bg-white text-gray-900;
}

@layer utilities {
  .text-balance {
    text-wrap: balance;
  }
}
```

8. Start the frontend application

```bash
npm run dev
```

9. Open your browser and navigate to `http://localhost:3000`

## Step 5: Run your application (If you restart and the previous commands have closed)

1. Start your local Ethereum network:
```bash
cd smartcontract
```
```bash
npx hardhat node
```
```bash
npx hardhat run scripts/deploy.ts --network localhost
```

2. Start your backend server:
```bash
cd backend
```
```bash
node app.js
```

3. Start your Next.js frontend:
```bash
cd frontend
```
```bash
npm run dev
```

4. Open your browser and navigate to `http://localhost:3000`

## Advanced - Start all scripts

1. Create `start_all.bat` in the folder containing `frontend`, `backend`, and `smartcontract`

```bat
@echo off

:: Start smartcontract
start cmd /k "cd /d %~dp0smartcontract && npx hardhat node"
timeout /t 15
start cmd /k "cd /d %~dp0smartcontract && npx hardhat run scripts/deploy.ts --network localhost"

:: Start frontend
start cmd /k "cd /d %~dp0frontend && npm run dev"

:: Start backend
start cmd /k "cd /d %~dp0backend && node server.js"

echo All projects started.
```

2. Create `stop_all.bat` in the same folder
```bat
@echo off

:: Kill all node processes
taskkill /F /IM node.exe

:: Kill all Windows Terminal instances except the current one
powershell -Command "Get-Process WindowsTerminal | Where-Object { $_.Id -ne $PID } | Stop-Process -Force"

echo All projects stopped.
```

3. Double click `start_all.bat` to start and double click `stop_all.bat` to stop.