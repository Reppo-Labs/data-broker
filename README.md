# Reppo Solver Node 🌟
                   

![Reppo Solver Node](https://img.shields.io/badge/Version-0.1.0-blue.svg) ![License](https://img.shields.io/badge/License-MIT-green.svg)

The **Reppo Solver Node** is a decentralized application designed to participate in the **Reppo.Exchange**, a blockchain-based data marketplace that facilitates the creation, validation, and exchange of high-quality datasets. The Solver Node listens for Requests for Data (RFDs), generates or sources the requested datasets, uploads them to IPFS, verifies NFT ownership for access control, and submits solutions to the Reppo Exchange smart contract.

This README provides an overview of the Reppo Solver Node, its architecture, setup instructions, and usage guidelines.

---

## Table of Contents 📋

- [Overview](#overview)
- [Architecture](#architecture)
  - [Components](#components)
  - [Workflow](#workflow)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
  - [Running in Test Mode](#running-in-test-mode)
  - [Running in Production Mode](#running-in-production-mode)
- [Example RFD](#example-rfd)
- [Dependencies](#dependencies)
- [Contributing](#contributing)
- [License](#license)

---

## Overview ✨

Reppo.Exchange enables:
- **Intent-Based Data Access**: AI agents broadcast Requests for Data (RFDs) that solver nodes fulfill
- **Decentralized MCP Network**: Distributed network of solver nodes providing MCP-compliant data services
- **Permissionless Participation**: Join the network by staking a Reppo Solver NFT
- **Token Incentives**: Earn $REPPO tokens for fulfilling RFDs
- **Standardized Integration**: MCP protocol ensures consistent data access across the network

The Solver Node performs these key functions:
1. **RFD Processing**: Listens for and processes Requests for Data on the blockchain
2. **MCP Integration**: Connects to MCP servers to fulfill data requests
3. **Data Generation/Querying**: Uses MCP tools (like DynamoDB) to generate or query data
4. **NFT Verification**: Ensures node operator owns a Reppo Solver NFT
5. **Solution Submission**: Submits verified solutions to the Reppo Exchange

The Reppo Solver Node is a key participant in this ecosystem, performing the following tasks:

1. **Listening for RFDs**: Monitors the Reppo Exchange smart contract for new RFD events.
2. **Generating Datasets**: Uses a data generation service (e.g., Two Ligma server) to create synthetic or real datasets that meet RFD specifications.
3. **Uploading to IPFS**: Stores datasets on IPFS via Pinata for decentralized, persistent storage.
4. **Verifying NFT Ownership**: Ensures the node operator owns a Reppo Node NFT, which is required to participate in the network.
5. **Submitting Solutions**: Submits the IPFS URI of the dataset to the Reppo Exchange smart contract for validation and reward distribution.

The Solver Node is designed to be modular, extensible, and easy to integrate with various data sources, such as Vana (Data DAOs), self-hosted LLMs, OpenGradient (synthetic data), or enterprise datasets.

---

## Architecture 🏗️

The Reppo Solver Node is built as a Python application with a modular architecture, separating concerns into distinct components. This design ensures maintainability, scalability, and flexibility for integrating with different data sources and blockchain networks.
The Solver Node implements a modular architecture designed for the decentralized MCP network:

### Components

The Solver Node consists of the following key components, each implemented in a dedicated Python module:

1. **RFDListener (`rfdListener.py`)**
   - **Purpose**: Listens for `RFDPosted` events on the Reppo Exchange
   - **Functionality**: 
     - Monitors blockchain for new RFDs
     - Parses RFD intents (data requirements)
     - Routes to appropriate solver components
   - **Dependencies**: `web3.py`, `python-dotenv`

2. **DataSolver (`datasolver/`)**
   - **Purpose**: MCP-compliant data generation and querying
   - **Functionality**: 
     - **MCP Provider**: Primary provider for production
       - DynamoDB Tool: Query and generate data from DynamoDB
       - Extensible for additional MCP tools
     - **HuggingFace Provider**: For AI-powered generation
     - **Mock Provider**: For testing and development
   - **Dependencies**: `mcp-sdk`, `requests`, `python-dotenv`

3. **IPFSUploader (`ipfsUploader.py`)**
   - **Purpose**: Uploads datasets to IPFS for decentralized storage.
   - **Functionality**: Uses the Pinata API to pin files to IPFS, returning an `ipfs://<CID>` URI. Handles file uploads securely with API key authentication.
   - **Dependencies**: `requests`, `python-dotenv`.

4. **NFTAuthorizer (`nftAuthorizer.py`)**
   - **Purpose**: Verifies that the node operator's wallet owns a Reppo Node NFT, which is required to submit solutions.
   - **Functionality**: Queries an ERC-721 NFT contract using `web3.py` to check the `balanceOf` the wallet address. Supports block-specific queries for historical ownership verification.
   - **Dependencies**: `web3.py`, `python-dotenv`.

5. **SolutionSubmitter (`submitSolution.py`)**
   - **Purpose**: Submits the dataset's IPFS URI to the Reppo Exchange smart contract.
   - **Functionality**: Builds, signs, and sends a transaction to the `submitSolution` function of the smart contract, including the RFD ID and IPFS URI. Returns the transaction hash upon success.
   - **Dependencies**: `web3.py`, `python-dotenv`.

6. **SolverNode (`solverNode.py`)**
   - **Purpose**: Orchestrates the entire workflow by integrating all components.
   - **Functionality**: Initializes the other components, coordinates RFD processing, and supports test and mock modes for development. In production mode, it runs the listener and processes RFDs as they arrive.
   - **Dependencies**: All other modules.

7. **Main CLI (`main.py`)**
   - **Purpose**: Provides a command-line interface (CLI) to start the Solver Node.
   - **Functionality**: Uses the `click` library to offer options like `--test` (for testing with a sample RFD) and `--mock` (for offline testing without blockchain interactions).
   - **Dependencies**: `click`, `python-dotenv`.

### Workflow

1. **RFD Detection & Intent Processing**:
   - Listener detects `RFDPosted` event
   - Extracts RFD intent (data requirements)
   - Validates NFT ownership

2. **MCP Tool Selection**:
   - Analyzes RFD requirements
   - Selects appropriate MCP tool (e.g., DynamoDB)
   - Configures tool parameters

3. **Data Generation/Querying**:
   - Production: Uses MCP tools for real data
   - Test: Uses HuggingFace for AI generation
   - Mock: Generates synthetic data

3. **Dataset Generation**:
   - The `DataSolver` uses the configured data provider to generate a dataset matching the RFD schema.
   - Supported providers include HuggingFace, Mock, OpenGradient, MCP, and LocalLLM.
   - The dataset is saved locally as `data/rfd_<rfd_id>_solution.json`.

4. **IPFS Upload**:
   - The `IPFSUploader` uploads the dataset to IPFS via Pinata, returning an `ipfs://<CID>` URI.

5. **Solution Submission**:
   - The `SolutionSubmitter` builds a transaction to call the `submitSolution` function on the Reppo Exchange smart contract, passing the RFD ID and IPFS URI.
   - The transaction is signed with the private key (`PRIVATE_KEY`) and sent to the Ethereum network.
   - The transaction hash is logged for tracking.

6. **Completion**:
   - If successful, the dataset is available on the Reppo Exchange for validation and reward distribution.
   - The Solver Node continues listening for new RFDs.

---

## Prerequisites ✅

To run the Reppo Solver Node, ensure you have the following:

- **Python**: Version 3.8 or higher.
- **Ethereum Node Access**: A connection to an Ethereum node (e.g., Infura, Alchemy) via an RPC URL.
- **Pinata Account**: API keys for IPFS pinning (`PINATA_API_KEY` and `PINATA_SECRET_API_KEY`).
- **Reppo Node NFT**: Ownership of a Reppo Node NFT in the wallet used for submission.
- **Environment Variables**: A `.env` file with required configurations (see [Configuration](#configuration)).
- **Dependencies**: Python packages listed in `requirements.txt` (see [Dependencies](#dependencies)).

### Optional Data Providers 🔄

The Solver Node is designed to be flexible in how it generates or sources datasets. You can:

1. **Use Built-in Providers** (optional):
   - **[HuggingFace](https://huggingface.co/)**: Use HuggingFace models for dataset generation
     - Requires: API token and model name
     - Good for: AI-powered dataset generation
   - **[MCP Servers](https://modelcontextprotocol.io/examples)**: Connect to Model Context Protocol servers
     - Various options including:
       - Data and file systems (Filesystem, PostgreSQL, SQLite, Google Drive)
       - Development tools (Git, GitHub, GitLab)
       - Web automation (Brave Search, Fetch, Puppeteer)
       - AI tools (EverArt, Sequential Thinking)
     - Good for: Structured data access and AI-powered operations
   - **OpenGradient**: Use OpenGradient's hosted models
     - Good for: Synthetic data generation
   - **LocalLLM**: Use a locally hosted LLM model
     - Good for: Private or custom model usage
   - **Mock**: Built-in mock data generation
     - No setup required
     - Good for: Testing and development

2. **Implement Custom Logic**:
   - Create your own data provider by extending the `DataProvider` class
   - Integrate with proprietary data sources
   - Implement custom business logic for dataset generation
   - Connect to enterprise data systems

3. **Use External Services**:
   - Connect to any data service via HTTP/API
   - Integrate with data marketplaces
   - Use cloud-based data generation services

The choice of data provider is entirely up to you and depends on your specific needs. The Solver Node's modular architecture makes it easy to integrate with any data source while maintaining the core functionality of RFD processing and blockchain interaction.

---

## Installation 💾

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/Reppo-Labs/solver-node
   cd reppo-solver-node
   ```

2. **Set Up a Virtual Environment** (recommended):
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install Dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

4. **Create the Output Directory**:
   ```bash
   mkdir data
   ```

5. **Configure Environment Variables**:
   Create a `.env` file in the project root (see [Configuration](#configuration)).

---

## Configuration ⚙️

The Solver Node relies on environment variables defined in a `.env` file. Below is an example `.env` file:

```env
# Ethereum configuration
WEB3_RPC_URL=https://mainnet.infura.io/v3/your-infura-key
CHAIN_ID=1
WALLET_ADDRESS=0xYourWalletAddress
PRIVATE_KEY=your-private-key

# Reppo Exchange smart contract
EXCHANGE_CONTRACT_ADDRESS=0xExchangeContractAddress
EXCHANGE_CONTRACT_ABI_PATH=./abis/exchange_abi.json

# Reppo Node NFT contract
NFT_CONTRACT_ADDRESS=0xNFTContractAddress
NFT_CONTRACT_ABI_PATH=./abis/nft_abi.json

# Pinata IPFS configuration
PINATA_API_KEY=your-pinata-api-key
PINATA_SECRET_API_KEY=your-pinata-secret-api-key


**Security Notes**:
- Never commit your `.env` file to version control.
- Keep your `PRIVATE_KEY` and Pinata API keys secure.
- Ensure the `data/` directory has write permissions for the user running the node.

---

## Usage 🖥️

The Solver Node supports three distinct modes of operation, each designed for different use cases:

### 1. Production Mode (Default)
```bash
python main.py start
```
- **Purpose**: Run the full node in production
- **Behavior**:
  - Listens for RFDs on the blockchain
  - Uses real data generation (HuggingFace if available)
  - Performs real blockchain interactions
  - Requires all environment variables
- **Use Case**: Running the actual node in production

### 2. Test Mode
```bash
python main.py start --test
```
- **Purpose**: Test the data generation pipeline
- **Behavior**:
  - Processes a sample RFD file (`sample_rfd.json` by default)
  - Uses real data generation (HuggingFace if available)
  - Skips blockchain interactions
  - Only requires `WALLET_ADDRESS` and optionally `HUGGINGFACE_API_TOKEN`
- **Use Case**: Testing data generation logic without blockchain dependencies
- **Custom RFD File**: Use `--rfd-file path/to/rfd.json` to specify a different RFD file

### 3. Mock Mode
```bash
python main.py start --mock
```
- **Purpose**: Simulate the entire pipeline for development
- **Behavior**:
  - Uses mock data generation
  - Uses mock blockchain responses
  - No external services required
  - Only requires `WALLET_ADDRESS`
- **Use Case**: Development, debugging, or demonstration without any external dependencies

### Mode Selection Rules
- If both `--test` and `--mock` are specified, mock mode takes precedence
- Production mode is the default when no flags are provided
- Each mode has specific environment variable requirements (see Configuration section)

### Example RFD for Testing
Create a `sample_rfd.json` file for test mode:
```json
{
    "rfd_id": "sf_weather_may_aug_001",
    "name": "Synthetic Weather Data for San Francisco (May to August)",
    "description": "A synthetic dataset containing daily weather information for San Francisco from May to August, including temperature, humidity, and precipitation.",
    "schema": {
        "type": "object",
        "properties": {
            "date": { "type": "string", "format": "date" },
            "temperature": { "type": "number", "description": "Average daily temperature in degrees Fahrenheit" },
            "humidity": { "type": "number", "description": "Average daily humidity percentage" },
            "precipitation": { "type": "number", "description": "Daily precipitation in inches" }
        },
        "required": ["date", "temperature", "humidity", "precipitation"]
    }
}
```

## Configuration ⚙️

### Environment Variables
Required variables vary by mode:

#### All Modes
- `WALLET_ADDRESS`: Your Ethereum wallet address

#### Production Mode
- `WEB3_RPC_URL`: Ethereum node RPC URL
- `CHAIN_ID`: Ethereum chain ID
- `PRIVATE_KEY`: Your wallet's private key
- `EXCHANGE_CONTRACT_ADDRESS`: Reppo Exchange contract address
- `NFT_CONTRACT_ADDRESS`: Reppo Node NFT contract address
- `PINATA_API_KEY`: IPFS pinning service API key
- `PINATA_SECRET_API_KEY`: IPFS pinning service secret key
- `HUGGINGFACE_API_TOKEN` (optional): For real data generation

#### Test Mode
- `HUGGINGFACE_API_TOKEN` (optional): For real data generation

#### Mock Mode
- No additional environment variables required

---

## Example RFD 📊

Below is an example RFD for synthetic weather data, which the Solver Node can process:

```json
{
    "rfd_id": "sf_weather_may_aug_001",
    "name": "Synthetic Weather Data for San Francisco (May to August)",
    "description": "A synthetic dataset containing daily weather information for San Francisco from May to August, including temperature, humidity, and precipitation.",
    "schema": {
        "type": "object",
        "properties": {
            "date": {
                "type": "string",
                "format": "date"
            },
            "temperature": {
                "type": "number",
                "description": "Average daily temperature in degrees Fahrenheit"
            },
            "humidity": {
                "type": "number",
                "description": "Average daily humidity percentage"
            },
            "precipitation": {
                "type": "number",
                "description": "Daily precipitation in inches"
            }
        },
        "required": ["date", "temperature", "humidity", "precipitation"]
    }
}
```

When processed, the Solver Node:
- Generates a dataset matching the schema (e.g., daily weather data for May–August).
- Saves it as `data/rfd_sf_weather_may_aug_001_solution.json`.
- Uploads it to IPFS, obtaining an `ipfs://<CID>` URI.
- Submits the URI to the Reppo Exchange smart contract if the wallet owns a Reppo Node NFT.

---

## Dependencies 📦

The Reppo Solver Node relies on the following Python packages, listed in `requirements.txt`:

```text
# Blockchain interactions
web3>=6.10.0
eth-account>=0.9.0
eth-typing>=3.4.0
eth-utils>=2.2.0

# HTTP requests
requests>=2.31.0

# Environment variables
python-dotenv>=1.0.0

# JSON processing
json5>=0.9.14

# CLI interface
click>=8.1.7

# Testing
pytest>=7.4.0
pytest-mock>=3.11.1
```

Install them using:
```bash
pip install -r requirements.txt
```

---

## Contributing 🤝

We welcome contributions to improve the Reppo Solver Node! To contribute:

1. **Fork the Repository**:
   ```bash
   git clone https://github.com/your-repo/reppo-solver-node.git
   ```

2. **Create a Feature Branch**:
   ```bash
   git checkout -b feature/your-feature
   ```

3. **Make Changes**:
   - Follow PEP 8 for Python code style.
   - Add tests for new functionality in the `tests/` directory.
   - Update documentation if necessary.

4. **Submit a Pull Request**:
   - Push your branch to your fork and create a pull request against the main repository.
   - Describe your changes clearly in the pull request description.

5. **Run Tests**:
   ```bash
   pytest
   ```

Please report issues or feature requests via the GitHub issue tracker.

---

## License 📜

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

This README provides a comprehensive guide to understanding, setting up, and running the Reppo Solver Node. For further questions, please open an issue on the GitHub repository or contact the maintainers.
