# Ticket Verification Blockchain Project Setup Guide

## Step 1: Set up the development environment

1. Install Node.js and npm (Node Package Manager)

2. Set up the development environment:

```bash
npm install -g ganache
```
This command installs Ganache, a tool for creating a local Ethereum blockchain for development and testing purposes.

```bash
mkdir ticket-verification-blockchain
```
This command creates a new directory for the project.

```bash
cd ticket-verification-blockchain
```
This command changes the current directory to the newly created project directory.

## Step 2: Install the smart contract directory

1. Initialize the smart contract directory in the main `ticket-verification-blockchain` directory

```bash
mkdir smartcontract
```
This command creates a new directory for the smart contract.

```bash
cd smartcontract
```
This command changes the current directory to the smart contract directory. 

```bash
npx hardhat
```
This command initializes a new Hardhat project for smart contract development. You'll be prompted with the following options:

- Choose "Create a TypeScript project"
- Confirm the Hardhat project root (it should default to your current directory)
- Choose to add a .gitignore file (recommended)
- Choose to install the sample project's dependencies (recommended)

1. Write Smart Contract: (smart-contract directory)

Create a file `TicketVerification.sol` and remove the file `Lock.sol` from the `contracts` directory:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

contract TicketVerification {
    struct Event {
        string name;
        uint256 date;
        uint256 ticketPrice;
        uint256 totalTickets;
        uint256 soldTickets;
        address organizer;
    }

    struct Ticket {
        uint256 eventId;
        address owner;
        bool used;
    }

    mapping(uint256 => Event) public events;
    mapping(uint256 => Ticket) public tickets;
    uint256 public eventCount;
    uint256 public ticketCount;

    event EventCreated(uint256 indexed eventId, string name, uint256 date, uint256 ticketPrice, uint256 totalTickets);
    event TicketPurchased(uint256 indexed ticketId, uint256 indexed eventId, address buyer);
    event TicketUsed(uint256 indexed ticketId, uint256 indexed eventId);

    function createEvent(string memory _name, uint256 _date, uint256 _ticketPrice, uint256 _totalTickets) public {
        eventCount++;
        events[eventCount] = Event(_name, _date, _ticketPrice, _totalTickets, 0, msg.sender);
        emit EventCreated(eventCount, _name, _date, _ticketPrice, _totalTickets);
    }

    function purchaseTicket(uint256 _eventId) public payable {
        Event storage eventDetails = events[_eventId];
        require(eventDetails.date > block.timestamp, "Event has already occurred");
        require(eventDetails.soldTickets < eventDetails.totalTickets, "Event is sold out");
        require(msg.value >= eventDetails.ticketPrice, "Insufficient payment");

        ticketCount++;
        tickets[ticketCount] = Ticket(_eventId, msg.sender, false);
        eventDetails.soldTickets++;

        emit TicketPurchased(ticketCount, _eventId, msg.sender);

        if (msg.value > eventDetails.ticketPrice) {
            payable(msg.sender).transfer(msg.value - eventDetails.ticketPrice);
        }
    }

    function verifyTicket(uint256 _ticketId) public {
        Ticket storage ticket = tickets[_ticketId];
        Event storage eventDetails = events[ticket.eventId];
        
        require(msg.sender == eventDetails.organizer, "Only event organizer can verify tickets");
        require(!ticket.used, "Ticket has already been used");
        require(eventDetails.date <= block.timestamp, "Event has not started yet");

        ticket.used = true;
        emit TicketUsed(_ticketId, ticket.eventId);
    }

    function getEvent(uint256 _eventId) public view returns (string memory, uint256, uint256, uint256, uint256, address) {
        Event memory eventDetails = events[_eventId];
        return (eventDetails.name, eventDetails.date, eventDetails.ticketPrice, eventDetails.totalTickets, eventDetails.soldTickets, eventDetails.organizer);
    }

    function getTicket(uint256 _ticketId) public view returns (uint256, address, bool) {
        Ticket memory ticket = tickets[_ticketId];
        return (ticket.eventId, ticket.owner, ticket.used);
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
  const TicketVerification = await ethers.getContractFactory("TicketVerification");
  const ticketVerification = await TicketVerification.deploy();

  await ticketVerification.waitForDeployment();

  console.log("TicketVerification contract deployed to:", await ticketVerification.getAddress());
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
This command deploys the TicketVerification contract to the local Ethereum network.

8. Save the deployed contract address:
```bash
0x5FbDB2315678afecb367f032d93F642f64180aa3
```
This is the address of the deployed TicketVerification contract on the local Ethereum network.

## Step 3: Setting up the backend directory (in the main ticket-verification-blockchain directory)

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
const contractABI = require("../smartcontract/artifacts/contracts/TicketVerification.sol/TicketVerification.json").abi;
const contractAddress = "deployed_contract_address_Step_2_8"; // The address of the deployed contract

const contract = new web3.eth.Contract(contractABI, contractAddress);

app.post("/create-event", async (req, res) => {
  const { name, date, ticketPrice, totalTickets, from } = req.body;
  try {
    const dateValue = BigInt(date);
    const ticketPriceValue = BigInt(ticketPrice);
    const totalTicketsValue = BigInt(totalTickets);

    const gasEstimate = await contract.methods
      .createEvent(
        name,
        dateValue.toString(),
        ticketPriceValue.toString(),
        totalTicketsValue.toString()
      )
      .estimateGas({ from });

    const result = await contract.methods
      .createEvent(
        name,
        dateValue.toString(),
        ticketPriceValue.toString(),
        totalTicketsValue.toString()
      )
      .send({ from, gas: gasEstimate });

    const eventId = result.events.EventCreated.returnValues.eventId;

    const serializedResult = JSON.parse(
      JSON.stringify(result, (key, value) =>
        typeof value === "bigint" ? value.toString() : value
      )
    );

    res.json({ ...serializedResult, eventId: `EV${eventId}` });
  } catch (error) {
    console.error("Error details:", error);
    res.status(500).json({
      error: "Failed to create event",
      message: error.message,
      details: error.data
        ? JSON.stringify(error.data, (key, value) =>
            typeof value === "bigint" ? value.toString() : value
          )
        : "No additional details available",
    });
  }
});

app.post("/purchase-ticket", async (req, res) => {
  const { eventId, from } = req.body;
  try {
    const cleanEventId = eventId.startsWith("EV") ? eventId.slice(2) : eventId;
    const event = await contract.methods.getEvent(cleanEventId).call();
    const ticketPrice = BigInt(event[2]);

    const gasEstimate = await contract.methods
      .purchaseTicket(cleanEventId)
      .estimateGas({ from, value: ticketPrice.toString() });

    const result = await contract.methods
      .purchaseTicket(cleanEventId)
      .send({ from, value: ticketPrice.toString(), gas: gasEstimate });

    const ticketId = result.events.TicketPurchased.returnValues.ticketId;

    const serializedResult = JSON.parse(
      JSON.stringify(result, (key, value) =>
        typeof value === "bigint" ? value.toString() : value
      )
    );

    res.json({ ...serializedResult, ticketId: `TI${ticketId}` });
  } catch (error) {
    console.error("Error details:", error);
    res.status(500).json({
      error: "Transaction failed",
      message: error.message,
      details: error.data
        ? JSON.stringify(error.data, (key, value) =>
            typeof value === "bigint" ? value.toString() : value
          )
        : "No additional details available",
    });
  }
});

app.post("/verify-ticket", async (req, res) => {
  const { ticketId, from } = req.body;
  try {
    const cleanTicketId = ticketId.startsWith("TI")
      ? ticketId.slice(2)
      : ticketId;
    const ticket = await contract.methods.getTicket(cleanTicketId).call();
    const event = await contract.methods.getEvent(ticket[0]).call();
    const currentTime = BigInt(Math.floor(Date.now() / 1000));

    if (currentTime < BigInt(event[1])) {
      return res.status(400).json({ error: "Event has not started yet" });
    }

    const gasEstimate = await contract.methods
      .verifyTicket(ticketId)
      .estimateGas({ from });

    const result = await contract.methods
      .verifyTicket(ticketId)
      .send({ from, gas: gasEstimate });

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
        ? JSON.stringify(error.data, (key, value) =>
            typeof value === "bigint" ? value.toString() : value
          )
        : "No additional details available",
    });
  }
});

app.get("/event/:id", async (req, res) => {
  const { id } = req.params;
  try {
    const cleanId = id.startsWith("EV") ? id.slice(2) : id;
    const event = await contract.methods.getEvent(cleanId).call();

    if (!event[0]) {
      return res.status(404).json({ error: "Event not found" });
    }

    const serializedEvent = {
      id: id,
      name: event[0],
      date: event[1].toString(),
      ticketPrice: event[2].toString(),
      totalTickets: event[3].toString(),
      soldTickets: event[4].toString(),
      organizer: event[5],
    };
    res.json(serializedEvent);
  } catch (error) {
    console.error("Error fetching event:", error);
    res.status(500).json({ error: "Failed to fetch event details" });
  }
});

app.get("/ticket/:id", async (req, res) => {
  const { id } = req.params;
  try {
    const cleanId = id.startsWith("TI") ? id.slice(2) : id;
    const ticket = await contract.methods.getTicket(cleanId).call();

    const serializedTicket = {
      id: id,
      eventId: ticket[0].toString(),
      owner: ticket[1],
      used: ticket[2],
    };
    res.json(serializedTicket);
  } catch (error) {
    console.error("Error fetching ticket:", error);
    res.status(500).json({ error: "Failed to fetch ticket details" });
  }
});

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

3. Starting the backend application: (in the backend directory)

```bash
node app.js
```

## Step 4: Setting up the Frontend Directory (in the main ticket-verification-blockchain directory)

1. Create a frontend directory using Next.js:

```bash
npx create-next-app@latest frontend 
```

When prompted, answer the questions as follows:

- Would you like to use TypeScript? ... No
- Would you like to use ESLint? ... No
- Would you like to use Tailwind CSS? ... Yes
- Would you like to use `src/` directory? ... No
- Would you like to use App Router? (recommended) ... Yes
- Would you like to customize the default import alias (@/*)? ... No

This will create a new Next.js project in the `frontend` directory with the specified configuration.

```bash
cd frontend
```
```bash
npm install -D tailwindcss postcss autoprefixer
```

2. Delete the `frontend/app/page.js` file

3. Create a new `index.js` file in the new `pages` directory <- setAccount is obtained from Step 2, Section 4

```jsx
// pages/index.js
import { useState, useEffect } from "react";

export default function Home() {
  const [account, setAccount] = useState("");
  const [eventName, setEventName] = useState("");
  const [eventDate, setEventDate] = useState("");
  const [ticketPrice, setTicketPrice] = useState("");
  const [totalTickets, setTotalTickets] = useState("");
  const [eventId, setEventId] = useState("");
  const [ticketId, setTicketId] = useState("");
  const [eventDetails, setEventDetails] = useState(null);
  const [ticketDetails, setTicketDetails] = useState(null);
  const [creationStatus, setCreationStatus] = useState("");
  const [purchaseStatus, setPurchaseStatus] = useState("");
  const [verificationStatus, setVerificationStatus] = useState("");
  const [errorMessage, setErrorMessage] = useState("");

  useEffect(() => {
    setAccount("account_Step_2_6");
  }, []);

  const createEvent = async () => {
    try {
      const response = await fetch("http://localhost:3001/create-event", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          name: eventName,
          date: new Date(eventDate).getTime() / 1000,
          ticketPrice,
          totalTickets,
          from: account,
        }),
      });
      const result = await response.json();
      if (response.ok) {
        setCreationStatus(`Event "${eventName}" (ID: ${result.eventId}) has been created successfully!`);
        setEventName("");
        setEventDate("");
        setTicketPrice("");
        setTotalTickets("");
      } else {
        setErrorMessage(result.error || "Failed to create event");
      }
    } catch (error) {
      console.error("Error creating event:", error);
      setErrorMessage("An error occurred while creating the event");
    }
  };

  const purchaseTicket = async () => {
    try {
      const response = await fetch("http://localhost:3001/purchase-ticket", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ eventId, from: account }),
      });
      const result = await response.json();
      if (response.ok) {
        setPurchaseStatus(`Ticket (ID: ${result.ticketId}) has been purchased successfully!`);
        setEventId("");
      } else {
        setErrorMessage(result.error || result.message || "Failed to purchase ticket");
      }
    } catch (error) {
      console.error("Error purchasing ticket:", error);
      setErrorMessage("An error occurred while purchasing the ticket");
    }
  };

  const verifyTicket = async () => {
    try {
      const response = await fetch("http://localhost:3001/verify-ticket", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ ticketId, from: account }),
      });
      const result = await response.json();
      if (response.ok) {
        setVerificationStatus(`Ticket (ID: ${ticketId}) has been verified successfully!`);
        setTicketId("");
      } else {
        setErrorMessage(result.error || result.message || "Failed to verify ticket");
      }
    } catch (error) {
      console.error("Error verifying ticket:", error);
      setErrorMessage("An error occurred while verifying the ticket");
    }
  };

  const fetchEventDetails = async () => {
    try {
      const response = await fetch(`http://localhost:3001/event/${eventId}`);
      const result = await response.json();
      if (response.ok) {
        setEventDetails(result);
        setErrorMessage("");
      } else {
        setEventDetails(null);
        setErrorMessage(result.error || "Failed to fetch event details");
      }
    } catch (error) {
      console.error("Error fetching event details:", error);
      setEventDetails(null);
      setErrorMessage("An error occurred while fetching event details");
    }
  };

  const fetchTicketDetails = async () => {
    try {
      const response = await fetch(`http://localhost:3001/ticket/${ticketId}`);
      const result = await response.json();
      if (response.ok) {
        setTicketDetails(result);
        setErrorMessage("");
      } else {
        setTicketDetails(null);
        setErrorMessage(result.error || "Failed to fetch ticket details");
      }
    } catch (error) {
      console.error("Error fetching ticket details:", error);
      setTicketDetails(null);
      setErrorMessage("An error occurred while fetching ticket details");
    }
  };

  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-6">
        Ticket Verification Blockchain
      </h1>
      <p className="mb-8">
        Your account: <span className="font-mono">{account}</span>
      </p>

      <div className="mb-8">
        <h2 className="text-2xl font-semibold mb-4">Create an Event</h2>
        <div className="flex flex-col space-y-4">
          <input
            type="text"
            placeholder="Event Name"
            value={eventName}
            onChange={(e) => setEventName(e.target.value)}
            className="px-3 py-2 border rounded"
          />
          <input
            type="datetime-local"
            value={eventDate}
            onChange={(e) => setEventDate(e.target.value)}
            min={new Date(new Date().setDate(new Date().getDate() + 1))
              .toISOString()
              .slice(0, 16)}
            className="px-3 py-2 border rounded"
          />
          <input
            type="number"
            placeholder="Ticket Price (in Wei)"
            value={ticketPrice}
            onChange={(e) => setTicketPrice(e.target.value)}
            className="px-3 py-2 border rounded"
          />
          <input
            type="number"
            placeholder="Total Tickets"
            value={totalTickets}
            onChange={(e) => setTotalTickets(e.target.value)}
            className="px-3 py-2 border rounded"
          />
          <button
            onClick={createEvent}
            className="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600"
          >
            Create Event
          </button>
        </div>
        {creationStatus && (
          <p className="mt-2 text-green-600">{creationStatus}</p>
        )}
      </div>

      <div className="mb-8">
        <h2 className="text-2xl font-semibold mb-4">Purchase a Ticket</h2>
        <div className="flex space-x-4">
          <input
            type="text"
            placeholder="Event ID"
            value={eventId}
            onChange={(e) => setEventId(e.target.value)}
            className="flex-grow px-3 py-2 border rounded"
          />
          <button
            onClick={purchaseTicket}
            className="px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600"
          >
            Purchase Ticket
          </button>
        </div>
        {purchaseStatus && (
          <p className="mt-2 text-green-600">{purchaseStatus}</p>
        )}
      </div>

      <div className="mb-8">
        <h2 className="text-2xl font-semibold mb-4">Verify a Ticket</h2>
        <div className="flex space-x-4">
          <input
            type="text"
            placeholder="Ticket ID"
            value={ticketId}
            onChange={(e) => setTicketId(e.target.value)}
            className="flex-grow px-3 py-2 border rounded"
          />
          <button
            onClick={verifyTicket}
            className="px-4 py-2 bg-purple-500 text-white rounded hover:bg-purple-600"
          >
            Verify Ticket
          </button>
        </div>
        {verificationStatus && (
          <p className="mt-2 text-green-600">{verificationStatus}</p>
        )}
      </div>

      <div className="mb-8">
        <h2 className="text-2xl font-semibold mb-4">Event Details</h2>
        <div className="flex space-x-4 mb-4">
          <input
            type="text"
            placeholder="Event ID"
            value={eventId}
            onChange={(e) => setEventId(e.target.value)}
            className="flex-grow px-3 py-2 border rounded"
          />
          <button
            onClick={fetchEventDetails}
            className="px-4 py-2 bg-yellow-500 text-white rounded hover:bg-yellow-600"
          >
            Fetch Event Details
          </button>
        </div>
        {eventDetails && (
          <div className="bg-gray-100 p-4 rounded">
            <p>
              <span className="font-semibold">Name:</span> {eventDetails.name}
            </p>
            <p>
              <span className="font-semibold">Date:</span>{" "}
              {new Date(eventDetails.date * 1000).toLocaleString()}
            </p>
            <p>
              <span className="font-semibold">Ticket Price:</span>{" "}
              {eventDetails.ticketPrice} Wei
            </p>
            <p>
              <span className="font-semibold">Total Tickets:</span>{" "}
              {eventDetails.totalTickets}
            </p>
            <p>
              <span className="font-semibold">Sold Tickets:</span>{" "}
              {eventDetails.soldTickets}
            </p>
            <p>
              <span className="font-semibold">Organizer:</span>{" "}
              {eventDetails.organizer}
            </p>
          </div>
        )}
      </div>

      <div>
        <h2 className="text-2xl font-semibold mb-4">Ticket Details</h2>
        <div className="flex space-x-4 mb-4">
          <input
            type="text"
            placeholder="Ticket ID"
            value={ticketId}
            onChange={(e) => setTicketId(e.target.value)}
            className="flex-grow px-3 py-2 border rounded"
          />
          <button
            onClick={fetchTicketDetails}
            className="px-4 py-2 bg-indigo-500 text-white rounded hover:bg-indigo-600"
          >
            Fetch Ticket Details
          </button>
        </div>
        {ticketDetails && (
          <div className="bg-gray-100 p-4 rounded">
            <p>
              <span className="font-semibold">Event ID:</span>{" "}
              {ticketDetails.eventId}
            </p>
            <p>
              <span className="font-semibold">Owner:</span>{" "}
              {ticketDetails.owner}
            </p>
            <p>
              <span className="font-semibold">Used:</span>{" "}
              {ticketDetails.used ? "Yes" : "No"}
            </p>
          </div>
        )}
      </div>

      {errorMessage && <p className="mt-4 text-red-600">{errorMessage}</p>}
    </div>
  );
}
```

4. Create a new `_app.js` file in the `pages` directory:
```js
import "../app/globals.css";

function MyApp({ Component, pageProps }) {
  return <Component {...pageProps} />;
}

export default MyApp;
```

5. Update the `postcss.config.mjs` file:
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

6. Update the `next.config.mjs` file:

```mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
};

export default nextConfig;
```

7. Update the `app/globals.css` file:

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