#  Data Retrieval from FactSet APIs

This guide demonstrates how to use the following FactSet APIs:

- **[FactSet Supply Chain API](https://developer.factset.com/api-catalog/factset-supply-chain-api)**

Retrieve supply chain relationship data between companies. The API exposes and classifies business relationships (Supplier, Customer, Partner, Competitor) among global companies, sourced from trusted primary sources and reverse-linked to non-disclosing parties. Rate-limited to 10 requests per second and 10 concurrent requests per user.

- **[FactSet ESG API](https://developer.factset.com/api-catalog/factset-esg-api)**

Retrieve ESG scores, spotlights, and articles powered by FactSet Truvalue Labs. The API applies Natural Language Processing and Machine Learning to uncover risks and opportunities from companies' Environmental, Social and Governance behavior, scored across 26 SASB categories. Rate-limited to 10 requests per second per user.

## Prerequisites

- Create API Key

You need to create an API key for you in the site: https://developer.factset.com/api-authentication

Use your work email as "Name" so it is easily identifiable.

Once finish create a `.env` file in the parent directory with your personal and unique ID (provided by Factset via mail) and api key (that you just created):

```bash
USERNAME="CPH_ABCD_123456"
API-KEY="AAAABBBB22223333aaaaBBBB4444"
```

**Please note that these are secrets, do not upload them to any public repository or share them with colleagues.**

- Install dependencies

It is strongly recommended to use a Python package manager. Below, I will use [miniconda](https://www.anaconda.com/docs/getting-started/miniconda/main).


```bash
(base) ➜ conda env create -f environment.yml
(base) ➜ conda activate factset
```

If you use other Python package manager (e.g `uv`), please install the python packages required in `environment.yml` - otherwise you won't be able to get any data.

## `/factset-supply-chain/v1/relationships` Endpoint Details

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

## `/factset-esg/v3/truvalue` Endpoint Details

FactSet ESG (powered by FactSet Truvalue) applies Natural Language Processing and Machine Learning to uncover risks and opportunities from companies' Environmental, Social and Governance (ESG) behavior. Scores are aggregated and categorized into continuously updated, material ESG scores based on 26 SASB categories.

The API extracts, analyzes, and generates scores from millions of documents each month collected from more than 200,000 data sources in over 38 languages.

This API is rate-limited to 10 requests per second per user.

**Base URL:** `https://api.factset.com/content/factset-esg/v3`

The v3 API exposes three main endpoints:

| Endpoint | Method | Description |
| --- | --- | --- |
| `/truvalue/scores` | GET / POST | Truvalue Scores and Ranks based on 26 SASB categories, Pillars, and Dimensions |
| `/truvalue/spotlights` | GET / POST | Daily collection of the most significant positive and negative ESG events |
| `/truvalue/articles` | GET / POST | Underlying news articles used by the AI engine to calculate ESG Scores |

### `/truvalue/scores` request parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `ids` | list | **Yes** | — | Security or Entity identifiers. Market Tickers, SEDOL, ISINs, CUSIPs, or FactSet Permanent IDs. Max 1000. |
| `scoreType` | string | No | `RANKS` | The Truvalue score type to retrieve. Only one score type per request. |
| `fields` | list | No | All fields | Controls the level of detail returned (see values below). |
| `startDate` | string | No | Previous close | Start date in `YYYY-MM-DD` format. Future dates not accepted. |
| `endDate` | string | No | Previous close | End date in `YYYY-MM-DD` format. Future dates not accepted. |
| `frequency` | string | No | `D` | Display frequency of data returned (see values below). |
| `calendar` | string | No | `SEVENDAY` | Calendar type: `FIVEDAY` (weekdays only) or `SEVENDAY` (includes weekends). |

### `scoreType` values

| Value | Description |
| --- | --- |
| `PULSE` | Pulse Score — measure of near-term performance changes that highlights opportunities and controversies |
| `PULSE_PCTL` | Pulse Percentile — context on Pulse Scores relative to peers in the same SICS Industry |
| `INSIGHT` | Insight Score — measure of a company's longer-term ESG track record |
| `INSIGHT_PCTL` | Insight Percentile — context on Insight Scores relative to peers (not valid for TOPLEVEL) |
| `MOMENTUM` | Momentum Score — trend of a company's Insight score over a trailing twelve-month period |
| `DYN_MATERIALITY` | Dynamic Materiality Score — percentage of data flow by category vs. total (not valid for TOPLEVEL) |
| `ARTVOL_DAY` | Article Volume (daily) — number of articles about a company on a daily basis |
| `CATVOL_DAY` | Category Volume (daily) — total number of category scores received daily |
| `ARTVOL_TTM` | Article Volume (TTM) — number of articles over the past 12 months |
| `CATVOL_TTM` | Category Volume (TTM) — total category scores over a trailing twelve-month period |
| `ARTVOL_TOT` | Article Volume (total) — number of articles throughout the entire history |
| `CATVOL_TOT` | Category Volume (total) — category tags throughout the entire history |
| `RANKS` | Ranks — Leader, Above Average, Average, Below Average, or Laggard (mapped from Industry Percentiles) |
| `ADJ_INSIGHT` | Adjusted Insight — blends company scores with industry medians for lower-volume firms (TOPLEVEL only) |
| `IND_PCTL` | Industry Percentiles — ranks companies from Laggards to Leaders (TOPLEVEL only) |

### `fields` values

| Value | Description |
| --- | --- |
| `TOPLEVEL` | Overall scores: `ALLCATEGORIES` (cumulative average of all 26 SASB categories) and `MATERIALITY` (composite of material categories) |
| `PILLARS` | High-level ESG groupings: Environmental, Social, and Governance |
| `DIMENSIONS` | Five areas: Environment, Business Model and Innovation, Human Capital, Leadership and Governance, and Social Capital |
| `SASBCATEGORIES` | All 26 individual SASB sustainability categories |

### `frequency` values

| Value | Description |
| --- | --- |
| `D` | Daily |
| `W` | Weekly, based on the last day of the week of the start date |
| `M` | Monthly, based on the last trading day of the month |
| `CY` | Calendar Annual, based on the last trading day of the calendar year |

### `/truvalue/spotlights` request parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `ids` | list | **Yes** | — | Security or Entity identifiers. Max 1500. |
| `categories` | list | No | `AllCategories` | SASB categories to filter by (see SASB categories below). |
| `fields` | list | No | All fields | Fields to include in the response. |
| `startDate` | string | **Yes** | — | Start date in `YYYY-MM-DD` format. |
| `endDate` | string | **Yes** | — | End date in `YYYY-MM-DD` format. |
| `primaryOnly` | boolean | No | — | When `true`, return only primary spotlights. |
| `isRemoved` | boolean | No | — | When `false`, exclude removed entries. |

### `/truvalue/articles` request parameters

| Parameter | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `ids` | list | **Yes** | — | Security or Entity identifiers. Max 1500. |
| `categories` | list | No | `AllCategories` | SASB categories to filter by (see SASB categories below). |
| `fields` | list | No | — | Fields to include in the response. |
| `startDate` | string | **Yes** | — | Start date in `YYYY-MM-DD` format. |
| `endDate` | string | **Yes** | — | End date in `YYYY-MM-DD` format. |
| `dateOf` | string | No | `PUBLICATION` | `PUBLICATION` (article publish date) or `INGESTION` (TVL first processing date). |

### SASB `categories` values

Used by the Spotlights and Articles endpoints. The 26 SASB categories plus two aggregate categories:

| Value | Pillar | Dimension |
| --- | --- | --- |
| `AllCategories` | — | — |
| `AccessAndAffordability` | Social | Social Capital |
| `AirQuality` | Environmental | Environment |
| `BusinessEthics` | Governance | Leadership and Governance |
| `BusinessModelResilience` | Governance | Leadership and Governance |
| `CompetitiveBehavior` | Governance | Leadership and Governance |
| `CriticalIncidentRiskManagement` | Governance | Leadership and Governance |
| `CustomerPrivacy` | Social | Social Capital |
| `CustomerWelfare` | Social | Social Capital |
| `DataSecurity` | Social | Social Capital |
| `EcologicalImpacts` | Environmental | Environment |
| `EmployeeEngagementDiversityAndInclusion` | Social | Human Capital |
| `EmployeeHealthAndSafety` | Social | Human Capital |
| `EnergyManagement` | Environmental | Environment |
| `GHGEmissions` | Environmental | Environment |
| `HumanRightsAndCommunityRelations` | Social | Human Capital |
| `LaborPractices` | Social | Human Capital |
| `ManagementOfTheLegalAndRegulatoryEnvironment` | Governance | Leadership and Governance |
| `MaterialSourcingAndEfficiency` | Environmental | Business Model and Innovation |
| `PhysicalImpactsOfClimateChange` | Environmental | Business Model and Innovation |
| `ProductDesignAndLifecycleManagement` | Environmental | Business Model and Innovation |
| `ProductQualityAndSafety` | Social | Social Capital |
| `SellingPracticesAndProductLabeling` | Social | Social Capital |
| `SupplyChainManagement` | Governance | Business Model and Innovation |
| `SystemicRiskManagement` | Governance | Leadership and Governance |
| `WasteAndHazardousMaterialsManagement` | Environmental | Environment |
| `WaterAndWastewaterManagement` | Environmental | Environment |

**Example request (`/truvalue/scores`):**

```json
{
    "data": {
        "ids": ["AMZN-US"],
        "scoreType": "PULSE",
        "fields": ["TOPLEVEL"],
        "startDate": "2023-12-31",
        "endDate": "2023-12-31",
        "frequency": "M",
        "calendar": "FIVEDAY"
    }
}
```

**Example request (`/truvalue/spotlights`):**

```json
{
    "data": {
        "ids": ["MSFT-US"],
        "startDate": "2022-01-01",
        "endDate": "2023-10-30",
        "categories": ["HumanRightsAndCommunityRelations"],
        "fields": ["spotlightPillar", "tvGroupId"],
        "primaryOnly": true,
        "isRemoved": false
    }
}
```

**Example request (`/truvalue/articles`):**

```json
{
    "data": {
        "ids": ["AMZN-US"],
        "categories": ["HumanRightsAndCommunityRelations"],
        "fields": ["datePublication"],
        "startDate": "2023-01-01",
        "endDate": "2023-10-30",
        "dateOf": "PUBLICATION"
    }
}
```

**Scores response fields:**

| Field | Type | Description |
| --- | --- | --- |
| `date` | string | Date for the period in `YYYY-MM-DD` format |
| `fsymId` | string | FactSet Entity Identifier (e.g., `000BJT-E`) |
| `requestId` | string | Identifier used in the request |
| `scoreType` | string | The score type returned (e.g., `PULSE`, `INSIGHT`) |
| `allCategoriesPulse` | number | Overall pulse across all categories (when TOPLEVEL requested) |
| `materialityPulse` | number | Pulse related to materiality (when TOPLEVEL requested) |
| `*Pulse` / `*Insight` / `*Momentum` | number | Score for each requested field/category, suffixed by score type |

**Spotlights response fields:**

| Field | Type | Description |
| --- | --- | --- |
| `factsetEntityId` | string | FactSet Entity Identifier |
| `requestId` | string | Identifier used in the request |
| `liveDate` | string | Live date of the event |
| `tvGroupId` | string | Truvalue group ID |
| `primaryArticleUrl` | string | URL of the primary article |
| `primaryArticleHeadline` | string | Headline of the primary article |
| `spotlightCategory` | string | SASB category of the spotlight |
| `spotlightPillar` | string | ESG pillar of the spotlight |
| `pulseOnStartDate` | number | Pulse value on the start date |
| `totalSpotlightVolume` | integer | Total spotlight volume |

**Articles response fields:**

| Field | Type | Description |
| --- | --- | --- |
| `datePublication` | string | Publication date in `YYYY-MM-DD` format |
| `factsetEntityId` | string | FactSet Entity Identifier |
| `requestId` | string | Identifier used in the request |
| `articleId` | string | Unique identifier for the article |

## Getting Started Examples

Refer to the `Suply Chain - GettingStarted.ipynb` and `ESG - GettingStarted.ipynb` notebooks for a full walkthrough with code examples using Python.
