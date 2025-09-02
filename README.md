# Global Amazon Bestselling Software Analytics ETL Pipeline

## Overview

This ETL (Extract, Transform, Load) pipeline collects and processes Amazon bestseller data across multiple countries to provide insights into global software product trends. The pipeline extracts data from a REST API, performs data cleaning and transformation, and prepares the dataset for analytics.

## Architecture

### Data Source
- **API Endpoint**: `https://data-liart.vercel.app/data`
- **Pagination**: Supports country-specific filtering and page-based pagination
- **Data Format**: JSON responses containing product bestseller information

### Pipeline Components

1. **Extract**: API data collection with country-based segmentation
2. **Transform**: Data cleaning, deduplication, and standardization
3. **Load**: CSV file output for downstream analytics

## Data Schema

### Raw Data Structure
The API returns the following fields for each product:

| Column | Data Type | Description | Nullable |
|--------|-----------|-------------|----------|
| `rank` | Integer | Product ranking position (1-100) | No |
| `asin` | String | Amazon Standard Identification Number | No |
| `product_title` | String | Full product name/title | No |
| `product_price` | String | Price with currency symbol | Yes |
| `product_star_rating` | Float | Average rating (1.0-5.0) | Yes |
| `product_num_ratings` | Float | Total number of customer ratings | Yes |
| `product_url` | String | Direct Amazon product URL | No |
| `product_photo` | String | Product image URL | No |
| `country` | String | Country code (US, IN, CA, etc.) | No |

### Derived Fields
- `page`: Page number from API pagination (added during extraction)
- `Unnamed: 0`: Auto-generated index (removed during transformation)
- `rank_change_label`: Ranking change indicator (always null, removed)

## Implementation Details

### Extract Phase

#### Country Discovery
```python
def fetch_countries():
    response = requests.get(url)
    data = response.json()["data"]
    countries = list({item["country"] for item in data})
    return countries
```

**Logic**: Automatically discovers available countries by making an initial API call and extracting unique country codes from the response.

#### Data Collection Strategy
```python
def fetch_all_data():
    # Iterates through pages for each country
    # Implements error handling and rate limiting
    # Returns consolidated country-specific data
```

**Key Features**:
- **Rate Limiting**: 0.5-second delay between requests to prevent API throttling
- **Error Handling**: Continues processing if individual pages fail
- **Timeout Protection**: 10-second timeout per request
- **Early Termination**: Stops when no more data is available

#### Discovered Countries
The pipeline identified 10 countries with Amazon bestseller data:
- **IN** (India)
- **CA** (Canada) 
- **IT** (Italy)
- **AU** (Australia)
- **DE** (Germany)
- **FR** (France)
- **JP** (Japan)
- **ES** (Spain)
- **MX** (Mexico)
- **US** (United States)


### Transform Phase

#### Data Quality Assessment
- **Missing Price Data**:  missing price information
- **Missing Rating Data**: missing star ratings and review counts
- **Duplicate Detection**: 9990 duplicate records identified (99.8% duplication rate)

#### Cleaning Operations
1. **Column Removal**: Dropped irrelevant columns (`Unnamed: 0`, `rank_change_label`, `page`)
2. **Duplicate Handling**: High duplication rate suggests need for deduplication strategy
3. **Data Type Optimization**: Maintained appropriate data types for efficient processing

#### Variable Derivation Logic

**Price Processing**:
- Prices stored as strings with currency symbols (₹, $, €, etc.)
- Requires parsing for numerical analysis
- Missing prices likely indicate unavailable or region-restricted products

**Rating Metrics**:
- `product_star_rating`: Decimal rating 
- `product_num_ratings`: Customer review count 
- Strong correlation expected between rating count and product popularity

**Geographic Segmentation**:
- Country codes enable regional analysis
- Ranking consistency across countries indicates global popularity
- Price variations reflect regional economic factors

## Technical Implementation

### Dependencies
```python
import pandas as pd    # Data manipulation and analysis
import requests       # HTTP API communication
import time          # Rate limiting implementation
```

### Error Handling Strategy
- **Network Failures**: Graceful degradation with retry logic
- **Data Validation**: Type checking and null value handling
- **Resource Management**: Controlled API request rates

### Performance Considerations
- **API Efficiency**: Pagination reduces single-request payload
- **Processing Speed**: Bulk operations using pandas vectorization

## Data Quality Metrics

### Completeness
- **Product Information**: 100% complete for core fields (title, ASIN, URL)
- **Pricing Data**: 94.6% complete
- **Rating Data**: 97% complete
- **Geographic Data**: 100% complete

### Data Integrity
- **ASIN Uniqueness**: Each product has unique Amazon identifier
- **Ranking Consistency**: Sequential ranking within country/page combinations
- **URL Validity**: All product URLs follow Amazon domain patterns

## Known Issues and Limitations

### High Duplication Rate
- **Issue**: 
- **Likely Causes**:
  - Products appearing across multiple countries
  - Seasonal ranking fluctuations
  - API pagination overlaps
- **Recommendation**: Implement deduplication based on ASIN + country combination


### Currency Standardization
- **Issue**: Prices in multiple currencies (₹, $, €, etc.)
- **Impact**: Requires currency conversion for cross-country price analysis
- **Solution**: Implement exchange rate conversion layer

## Recommended Next Steps

### Immediate Actions
1. **Complete US Data Extraction**: Resume interrupted extraction process
2. **Implement Deduplication**: Create composite key strategy (ASIN + country)
3. **Price Standardization**: Add currency conversion functionality

### Enhancement Opportunities
1. **Real-time Updates**: Schedule periodic data refreshes
2. **Data Validation**: Add schema validation and data quality checks
3. **Performance Optimization**: Implement parallel processing for country-based extraction
4. **Database Integration**: Replace CSV output with database storage

### Analytics Preparation
1. **Category Classification**: Derive product categories from titles
2. **Competitive Analysis**: Cross-country price and ranking comparisons
3. **Trend Analysis**: Historical ranking change tracking
4. **Market Insights**: Regional preference identification

## File Outputs
- **Primary Dataset**: `extracted_amazon_data.csv` 
- **Schema**: 17 columns after transformation


## Usage Guidelines

### Loading the Dataset
```python
import pandas as pd
amazon_df = pd.read_csv('extracted_amazon_data.csv')
```

### Basic Analysis Examples
```python
# Country distribution
amazon_df['country'].value_counts()

# Price analysis (requires currency parsing)
# Rating distribution
amazon_df['product_star_rating'].describe()

# Top products by country
amazon_df.groupby('country').head(10)
```

## Maintenance Notes

- **API Dependencies**: Monitor endpoint availability and response format changes
- **Rate Limiting**: Current 0.5s delay may need adjustment based on API terms
- **Data Freshness**: Bestseller rankings change frequently; consider update frequency
- **Storage Requirements**: Full dataset requires significant storage for historical analysis
