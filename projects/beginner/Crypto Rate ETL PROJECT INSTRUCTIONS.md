### **Binance Crypto Price ETL**

A small, containerized Python ETL application that extracts live cryptocurrency
prices from the **Binance Spot REST API**, extracts the current **USD/KES**
exchange rate from the **Frankfurter API**, converts the crypto price into
**Kenyan Shillings**, and prints a clean report to the terminal.

The whole thing ships as a Docker image, so anyone can run it without cloning
the repo or installing Python.

```bash
docker run --rm vickituku/binance-etl --symbol BTCUSDT
```

---

## Quick Start

Pull the image:

```bash
docker pull vickituku/binance-etl:latest
```

Run it:

```bash
docker run --rm vickituku/binance-etl --symbol BTCUSDT
```

Any Binance spot pair works:

```bash
docker run --rm vickituku/binance-etl --symbol ETHUSDT
```

```bash
docker run --rm vickituku/binance-etl --symbol SOLUSDT
```

---

## Example Output

```text
[EXTRACT] Fetching BTCUSDT from Binance...
[EXTRACT] Fetching USD/KES exchange rate...
[TRANSFORM] Calculating approximate KES value...

--------------------------------------------------
BINANCE CRYPTO PRICE
--------------------------------------------------
Symbol:            BTCUSDT
Price USDT:        $64,582.31
USD/KES Rate:      129.15
Approx. Price KES: KSh 8,340,798.34
--------------------------------------------------
Source: Binance
Status: SUCCESS
--------------------------------------------------
```

> The KES figure is an **approximation**. Binance quotes the pair in **USDT**,
> not in actual US dollars, so the conversion assumes 1 USDT ≈ 1 USD.

---

## How It Works

```text
                USER
                  |
        docker run --symbol BTCUSDT
                  v
        +--------------------+
        |   Docker Container |
        +--------------------+
                  v
        +--------------------+
        |    Python ETL      |
        +--------------------+
             |          |
             v          v
       Binance API    FX API
             |          |
       BTCUSDT Price   USD/KES
             |          |
             +-----+----+
                   v
             TRANSFORMATION
             (USD -> KES)
                   v
               OUTPUT
```

### 1. Extract

| Source | Endpoint | Returns |
|---|---|---|
| Binance | `https://api.binance.com/api/v3/ticker/price?symbol=<SYMBOL>` | Latest spot price in USDT |
| Frankfurter | `https://api.frankfurter.dev/v1/latest?base=USD&symbols=KES` | Current USD/KES rate |

Neither API requires a key.

### 2. Transform

Raw strings become numbers, then:

```python
crypto_price_kes = crypto_price_usd * usd_kes_rate
```

### 3. Load

Results are written to the terminal (default), or to JSON / CSV / PostgreSQL
depending on the `--output` flag.

---

## Command-Line Arguments

Built with Python's `argparse`.

| Argument | Required | Description |
|---|---|---|
| `--symbol` | Yes | Binance trading pair, e.g. `BTCUSDT`. Accepts a comma-separated list. |
| `--output` | No | `table` (default), `json`, `csv`, or `postgres`. |
| `-h`, `--help` | No | Show usage information. |

Running with no arguments prints usage instead of crashing:

```bash
docker run --rm vickituku/binance-etl
```

```text
usage: main.py [-h] --symbol SYMBOL [--output {table,json,csv,postgres}]
main.py: error: the following arguments are required: --symbol
```

---

## Error Handling

The application never dumps a raw Python traceback for a normal user mistake.

**Invalid symbol**

```bash
docker run --rm vickituku/binance-etl --symbol HARUNUSDT
```

```text
ERROR: HARUNUSDT is not a valid Binance trading pair.
```

**Network failure**

```text
ERROR: Unable to connect to Binance API.
Please check your internet connection.
```

**Exchange-rate API failure**

```text
ERROR: Unable to retrieve the USD/KES exchange rate.
```

Every failure path exits with a non-zero status code so the container can be
used inside a scheduler or CI pipeline.

---

## Project Structure

```text
binance-etl/
│
├── app/
│   ├── __init__.py
│   ├── extractor.py      # Binance + Frankfurter calls
│   ├── transformer.py    # USD -> KES conversion
│   └── loader.py         # terminal / json / csv / postgres output
│
├── main.py               # argparse entry point
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── README.md
├── .dockerignore
└── .gitignore
```

---

## Running Locally (without Docker)

```bash
git clone https://github.com/vicKituku/binance-etl.git
```

```bash
cd binance-etl && python -m venv .venv && source .venv/bin/activate
```

```bash
pip install -r requirements.txt
```

```bash
python main.py --symbol BTCUSDT
```

---

## Building the Image Yourself

```bash
docker build -t binance-etl .
```

```bash
docker images
```

```bash
docker run --rm binance-etl --symbol BTCUSDT
```

The Dockerfile uses `ENTRYPOINT` rather than `CMD` so that arguments typed
after the image name are passed straight through to the Python program:

```dockerfile
ENTRYPOINT ["python", "main.py"]
```

With `CMD`, `docker run binance-etl --symbol BTCUSDT` would try to *replace*
the command instead of appending to it.

---

## Publishing to Docker Hub

```bash
docker tag binance-etl vickituku/binance-etl:latest
```

```bash
docker login
```

```bash
docker push vickituku/binance-etl:latest
```

---

## Optional Output Formats

### JSON

```bash
docker run --rm vickituku/binance-etl --symbol BTCUSDT --output json
```

```json
{
    "symbol": "BTCUSDT",
    "price_usdt": 64582.31,
    "usd_kes_rate": 129.15,
    "price_kes": 8340798.34,
    "source": "binance",
    "status": "success"
}
```

### CSV

A container started with `--rm` is deleted on exit, taking any file it wrote
with it. Mount a volume so the CSV survives:

```bash
docker run --rm -v "$(pwd)/data:/app/data" vickituku/binance-etl --symbol BTCUSDT --output csv
```

```text
symbol,price_usdt,usd_kes_rate,price_kes,extracted_at
BTCUSDT,64582.31,129.15,8340798.34,2026-08-24T14:52:10
```

### Multiple Symbols

```bash
docker run --rm vickituku/binance-etl --symbol BTCUSDT,ETHUSDT,SOLUSDT
```

```text
---------------------------------------------------------------
SYMBOL        USDT PRICE             APPROXIMATE KES
---------------------------------------------------------------
BTCUSDT       $64,582.31             KSh 8,340,798.34
ETHUSDT       $2,746.82              KSh 354,752.85
SOLUSDT       $145.32                KSh 18,767.08
---------------------------------------------------------------
```

---

## PostgreSQL Load

Results can be appended to a Postgres table instead of only being printed:

```sql
CREATE TABLE crypto_prices (
    id SERIAL PRIMARY KEY,
    symbol VARCHAR(20),
    price_usdt NUMERIC,
    usd_kes_rate NUMERIC,
    price_kes NUMERIC,
    extracted_at TIMESTAMP
);
```

Connection details come from environment variables:

| Variable | Default |
|---|---|
| `POSTGRES_HOST` | `db` |
| `POSTGRES_PORT` | `5432` |
| `POSTGRES_DB` | `crypto` |
| `POSTGRES_USER` | `postgres` |
| `POSTGRES_PASSWORD` | — (required) |

---

## Docker Compose

Bring up the ETL and its database together:

```bash
docker compose up
```

Compose starts PostgreSQL, waits for it to be healthy, then runs the ETL with
`--output postgres`.

---

## Requirements

* Docker (for the containerized workflow)
* Python 3.11+ and `requests` (only if running from source)
* An internet connection — both data sources are live public APIs

---

## Author

Built as the Week 14 project — a reusable, containerized ETL data product that
another engineer can pull from Docker Hub and run anywhere.
