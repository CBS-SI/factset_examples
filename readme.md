#  Data Retrieval from FactSet Supply Chain API

This guide demonstrates how to use the [FactSet Supply Chain API](https://developer.factset.com/api-catalog/factset-supply-chain-api) to retrieve supply chain relationship data between companies. The API exposes and classifies business relationships (Supplier, Customer, Partner, Competitor) among global companies, sourced from trusted primary sources and reverse-linked to non-disclosing parties.

This API is rate-limited to 10 requests per second and 10 concurrent requests per user.

## Prerequisites

- Create API Key

You need to create an API key for you in the site: https://developer.factset.com/api-authentication

Use your email as "Name" so it is easily identifiable.

Once finish create a `.env` file in the parent directory as:

```bash
USERNAME="CPH_YOUR_PERSONAL_USER_NAME"
API-KEY="A1bc_YOUR_PERSONAL_API_PASSWORD"
```

Please note that these are secrets, do not upload them to any public repository or share them with colleagues.

- Install dependencies

It is strongly recommended to use a Python package manager. Below, I will use [miniconda](https://www.anaconda.com/docs/getting-started/miniconda/main).


```bash
(base) ➜ conda env create -f environment.yml
(base) ➜ conda activate factset
```

If you use other Python package manager (e.g `uv`), please install the python packages required in `environment.yml` - otherwise you won't be able to get any data.

## `/relationships` Endpoint Details

Retrieves supply chain relationship data sorted by product overlap count and percentage. Returns the entity ID and associated company names for categories such as Supplier, Competitor, Customer, and Partner.

**Base URL:** `https://api.factset.com/content/factset-supply-chain/v1`

**Request parameters:**

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `ids` | list | **Yes** | — | List of security identifiers. Accepted types: Market Tickers, SEDOL, ISINs, CUSIPs, or FactSet Permanent IDs. Max 500. |
| `relationshipType` | string | **Yes** | `SUPPLIERS` | Category of business relationship (see values below) |
| `companyType` | string | No | `PUBLIC_COMPANIES_ONLY` | Filter results by company public/private status (see values below) |
| `relationshipDirection` | string | No | `ALL` | Filter by how the relationship was disclosed (see values below) |

### `ids` values

- `Market Tickers`: A ticker symbol appended with a region code separated by a hyphen. For example, `AAPL-USA` represents Apple Inc. traded in the USA. The region suffix corresponds to the country or region in which the listing trades.

- `SEDOL`: A SEDOL (Stock Exchange Daily Official List) is a 7-character alphanumeric code assigned by the London Stock Exchange to securities—including stocks, bonds, and funds—primarily in the UK and Ireland.

- `CUSIP`: A 9-character alphanumeric identifier assigned by CUSIP Global Services to North American securities (stocks, bonds, mutual funds). Commonly used as a linking key for US companies.

- `ISIN`: An International Securities Identification Number is a 12-character alphanumeric code that uniquely identifies a specific security internationally. It consists of a two-letter country code, a nine-character identifier, and a check digit.

- `FactSet Entity ID`: FactSet's own permanent identifier for entities (e.g., `000C7F-E`). FactSet also maintains permanent security-level identifiers (`FSYM_ID`, e.g., `R85KLC-S`) at security, regional, and listing levels.

### `relationshipType` values

| Value | Description |
| --- | --- |
| `SUPPLIERS` | Organizations providing goods or services to the source company (source company is the buyer) |
| `CUSTOMERS` | Organizations receiving goods or services from the source company (source company is the seller) |
| `COMPETITORS` | Organizations the source company has identified as rivals, typically operating in comparable markets or sectors |
| `PARTNERS` | Organizations in which the source company holds ownership interests through shares or equity stakes |

### `companyType` values

| Value | Description |
| --- | --- |
| `PUBLIC_COMPANIES_ONLY` | Include only relationships involving publicly traded companies |
| `PRIVATE_COMPANIES_ONLY` | Include only relationships involving privately held companies |
| `ALL` | Include both publicly traded and privately held companies |

### `relationshipDirection` values

| Value | Description |
| --- | --- |
| `ALL` | Include both direct and reverse relationships |
| `DIRECT` | Include only relationships where the source company identified and named the connection to the target company |
| `REVERSE` | Include only relationships where the target company identified and named the connection to the source company |

**Example request:**

```json
{
    "data": {
        "ids": ["AAPL-USA"],
        "relationshipType": "SUPPLIERS",
        "companyType": "PUBLIC_COMPANIES_ONLY",
        "relationshipDirection": "ALL"
    }
}
```

**Response fields:**

| Field | Type | Description |
| --- | --- | --- |
| `entityId` | string | FactSet entity identifier of the related company |
| `companyName` | string | Name of the related company |
| `overlappingProductCount` | string | Count of overlapping products |
| `overlapPercentage` | int | Percentage of product overlap |
| `relationshipDirection` | string | Whether the relationship is Direct or Reverse |
| `requestId` | string | The original request identifier |
| `requestEntityId` | string | The entity ID of the requested company |

## Getting Started

Refer to the `GettingStarted.ipynb` notebook for a full walkthrough with code examples using Python and the `requests` library.
