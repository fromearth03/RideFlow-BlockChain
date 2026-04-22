# rust-blockchain

![example workflow](https://github.com/mrnaveira/rust-blockchain/actions/workflows/build.yaml/badge.svg) ![example workflow](https://github.com/mrnaveira/rust-blockchain/actions/workflows/lint.yaml/badge.svg) ![example workflow](https://github.com/mrnaveira/rust-blockchain/actions/workflows/test.yaml/badge.svg) [![Coverage Status](https://coveralls.io/repos/github/mrnaveira/rust-blockchain/badge.svg?service=github)](https://coveralls.io/github/mrnaveira/rust-blockchain)

A simplified event-chain backend written in Rust.

Features:
* Stores event records in chained blocks
* Uses deterministic block hashing with `previous_hash` linkage
* Provides a minimal REST API with one `POST /blocks` and one `GET /blocks`

## 🚀 Live Site
The application is live at: [https://rideflow.aliakbar.systems/](https://rideflow.aliakbar.systems/)

## Getting Started
You will need Rust and Cargo installed.

```bash
# Download the code
$ git clone https://github.com/mrnaveira/rust-blockchain
$ cd rust-blockchain

# Run all tests
$ cargo test

# Build the project in release mode
$ cargo build --release

# Run the application
$ ./target/release/rust_blockchain
```

The application will start the REST API on port `8000` by default. To change environment variables (for example, `PORT`) refer to the `.env.example` file.

### PostgreSQL storage

On each successful `POST /blocks`, the server also persists one row in PostgreSQL table `blockchain`.

- `hashcode`: block hash (key)
- `data`: a single string value containing `eventtype`, `data`, `timestamp`, and `previous_hash`

Expected table schema:

```sql
CREATE TABLE IF NOT EXISTS blockchain (
	hashcode text NOT NULL,
	data text
);
```

Environment variables (aligned with your Spring datasource values):

```bash
DATABASE_ENABLED=true
DATABASE_URL= URL_HERE
sslmode=require
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=DB_Password_Here
```

For development setup, check the [development notes section](#development-notes).

## Client REST API
The application provides a REST API for clients to operate with the blockchain.

| Method | URL | Description |
| --- | --- | --- |
| GET | /blocks | List all blocks of the blockchain
| POST | /blocks | Append a new event block to the blockchain

### `POST /blocks` request body

```json
{
	"eventtype": "Ride",
	"data": "From City A to City B",
	"timestamp": 1700000000123
}
```

### `GET /blocks` response item

```json
{
	"eventtype": "Ride",
	"data": "Ride from City A to City B",
	"timestamp": 1700000000123,
	"previous_hash": "0x...",
	"current_hash": "0x..."
}
```

The file `doc/rest_api.postman_collection.json` contains a Postman collection with examples of all requests.

## Block Structure

In this project, each block stores a single event payload plus chaining metadata.

The `model` module in this project contains the data structures to model the blockchain, as described in the next diagram:

![Blockchain structure diagram](./doc/blockchain_structure.png)

Each block contains the following data:
* **index**: position of the block in the blockchain
* **timestamp**: event timestamp supplied by the API payload
* **eventtype**: event category string
* **data**: event data string
* **previous_hash**: hash of the previous block in the chain
* **hash/current_hash**: SHA-256 hash of the block including all fields
