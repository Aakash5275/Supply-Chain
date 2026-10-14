# Supply Chain Blockchain DApp

A supply chain tracker that records every step of a product's journey on a blockchain, so the history cannot be edited afterwards.

## About the project

Products move through five stages, and each stage can only be completed by the participant registered for it:

```text
Created → Processing → In Transit → For Sale → Sold
```

| Participant | What they do |
| --- | --- |
| **Owner** | The account that deployed the contract. Registers all other participants. |
| **Supplier** | Supplies a new product (Created → Processing). |
| **Producer** | Creates products, then processes them (Processing → In Transit). |
| **Distributor** | Distributes the product (In Transit → For Sale). |
| **Seller** | Lists the product for sale, then marks it Sold. |

How the pieces fit together:

- `backend` is a Hardhat project holding the Solidity contract `SupplyChain.sol`.
- `client` is a Next.js app that talks to the contract through Web3.js.
- Ganache runs a local blockchain on your machine, so no real money or network is involved.

App pages:

| Page | Path | Purpose |
| --- | --- | --- |
| Dashboard | `/` | Totals for products, participants, and stages |
| Register Roles | `/register-roles` | Add participants (owner only) |
| Order Materials | `/order-materials` | Create products (registered producer only) |
| Supply Materials | `/supply-materials` | Move a product to its next stage |
| Track Materials | `/track-materials` | See where a product is and who handled it |

## Setup

### Prerequisites

- [Node.js 18+](https://nodejs.org/)
- A browser. MetaMask is optional.

### 1. Install dependencies

From the project root:

```bash
cd backend
npm install

cd ../client
npm install
```

### 2. Start Ganache

Leave this terminal running:

```bash
npx ganache --host 127.0.0.1 --port 7545 --chain.chainId 1337
```

Ganache is ready when the log shows `RPC Listening on 127.0.0.1:7545`. A `uws` binary warning on newer Node.js versions is harmless.

### 3. Deploy the contract

In a second terminal:

```bash
cd backend
npm run deploy:ganache
```

The script writes the contract address to `client/src/deployments.json`. The first Ganache account deploys the contract and becomes its owner. Deploy again whenever Ganache is restarted, because its data is lost when it stops.

### 4. Start the app

In a third terminal:

```bash
cd client
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

For a production build:

```bash
cd client
npm run build
npm start
```

## How to use the app

Every page shows `Account: 0x...` near the top. That is the address sending your transactions.

### Which account is used

- **Without MetaMask**, the app always uses Ganache's first account, which is the owner. It cannot switch accounts.
- **With MetaMask**, the app uses whichever account is selected there.

The contract allows one address to hold several roles. Without MetaMask, register the owner address as supplier, producer, distributor, and seller. Then you can run the whole flow from one account.

### 1. Add roles

Go to **Register Roles** (`/register-roles`). Only the owner can register participants. Otherwise you see an "Owner access required" notice.

Fill in the form:

| Field | What to enter |
| --- | --- |
| Role Type | Supplier, Producer, Distributor, or Seller |
| Ethereum Address | The participant's address, starting with `0x` |
| Name | Participant name |
| Location | City and country |

Click **Register**. The participant appears in the list for that role below the form.

Register at least one of each role: four registrations. You can reuse the same address for all four. Copy it from the `Account:` line at the top of the page.

#### Example: register four roles

Open **Register Roles** and submit the form four times, once per row. Replace `<ADDRESS>` with your own address.

Without MetaMask, use the owner address from the `Account:` line in all four rows. With MetaMask, use four different accounts from the Ganache output.

| # | Role Type | Ethereum Address | Name | Location |
| --- | --- | --- | --- | --- |
| 1 | Supplier | `<ADDRESS>` | Green Valley Farms | Pune, India |
| 2 | Producer | `<ADDRESS>` | Sunrise Foods Ltd | Ahmedabad, India |
| 3 | Distributor | `<ADDRESS>` | FastTrack Logistics | Mumbai, India |
| 4 | Seller | `<ADDRESS>` | FreshMart Store | Delhi, India |

After each click on **Register**, the participant appears under its role, and its ID starts at 1. When all four rows are in, **Order Materials** unlocks.

Addresses change every time Ganache restarts. Copy the current ones, and register the roles again after a restart and redeploy.

### 2. Order a product

Go to **Order Materials** (`/order-materials`).

- Your account must be a registered producer. Otherwise you see "Access restricted".
- All four roles must have at least one participant. Otherwise you see "Requirements not met", with a count per role.

Enter a product name and a description, then click **Add product**. The product gets the next ID (1, 2, 3, …) and starts at stage **Created**.

#### Example: order two materials

| Product name | Product description | Resulting ID | Starting stage |
| --- | --- | --- | --- |
| Organic Wheat Flour | 50 kg bags, batch WF-001 | 1 | Created |
| Basmati Rice | 25 kg bags, batch BR-014 | 2 | Created |

Both products show in the **Products** table below the form with their stage.

### 3. Move the product through the stages

Go to **Supply Materials** (`/supply-materials`). Enter the product ID, then run these steps in order. Each step must be sent from an account registered for that role.

| Step | Registered role needed | Stage change |
| --- | --- | --- |
| 1. Supply | Supplier | Created → Processing |
| 2. Process | Producer | Processing → In Transit |
| 3. Distribute | Distributor | In Transit → For Sale |
| 4. List for sale | Seller | Records the seller. Stage stays For Sale. |
| 5. Mark as sold | The same seller | For Sale → Sold |

Skipping a step shows "Wrong stage". Using an account without the role shows a message such as "Not supplier".

### 4. Track a product

Go to **Track Materials** (`/track-materials`).

- Enter a product ID and click **Track**, or click any row in the **Products on chain** table.
- The result shows the current stage, a QR code for the product, and a journey view. For each completed stage it shows who handled it: their ID, name, location, and address.

#### Example: follow product 1 from start to finish

Use product ID `1` (Organic Wheat Flour) on **Supply Materials**. After each step, open **Track Materials**, enter `1`, and click **Track**.

| After this step | Stage shown in Track Materials | Participants shown in the journey |
| --- | --- | --- |
| Order (Add product) | Product Created | None yet. The journey says "Product created — awaiting supplier". |
| 1. Supply | Processing Stage | Supplied by Green Valley Farms (Supplier, Pune, India) |
| 2. Process | In Transit Stage | Adds Processed by Sunrise Foods Ltd (Producer, Ahmedabad, India) |
| 3. Distribute | For Sale Stage | Adds Distributed by FastTrack Logistics (Distributor, Mumbai, India), plus a pending "Seller listing" step |
| 4. List for sale | For Sale Stage | The pending step becomes Listed by FreshMart Store (Seller, Delhi, India) |
| 5. Mark as sold | Product Sold | All four participants, then a final "Sold" step |

Each participant entry shows their ID, name, location, and address. Every tracked product also shows a QR code and its current stage.

Product 2 stays at **Created** the whole time, so tracking `2` shows no participants. Both products are still listed in the **Products on chain** table.

The **Dashboard** (`/`) shows the totals: products, registered participants, and how many products sit in each stage.

## Common problems

| What you see | What to do |
| --- | --- |
| Dashboard is empty or shows a connection error | Check that Ganache is running on port `7545`, then refresh. |
| `Contract not found on chainId …` | Run `npm run deploy:ganache` in `backend` again. |
| Owner access required | The current account is not the deployer. Use the owner account. |
| Access restricted on Order Materials | Register the current account as a producer first. |
| `Wrong stage` | Run the steps in order for that product ID. |

## MetaMask (optional)

To use different accounts for different roles, add this network in MetaMask and import accounts from the Ganache output:

- Network name: `Ganache Local`
- RPC URL: `http://127.0.0.1:7545`
- Chain ID: `1337`
- Currency symbol: `ETH`

Select the account that matches the role before each action.
