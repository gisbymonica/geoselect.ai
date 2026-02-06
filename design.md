# Design: Smart Site Selection Geo-Agent

## 1. System Architecture

The Smart Site Selection Geo-Agent follows an **Agentic Workflow pattern**, where a central "Brain" (LLM) orchestrates specialized "Tools" to deliver prescriptive site recommendations.

### 1.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    User Interface Layer                      │
│              (Natural Language Query Input)                  │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│                  Orchestrator (Brain)                        │
│           LLM with ReAct Loop (Reasoning + Acting)          │
│                  (e.g., Gemini 1.5 Pro)                     │
└───────────────────────────┬─────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼──────┐  ┌────────▼────────┐
│ DemographicTool│  │   POITool   │  │ RealEstateTool  │
└───────┬────────┘  └──────┬──────┘  └────────┬────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                ┌───────────▼───────────┐
                │   GeometryTool        │
                │ (Isochrone Calculator)│
                └───────────┬───────────┘
                            │
                ┌───────────▼───────────┐
                │   H3 Spatial Index    │
                │  (Hexagonal Grid)     │
                └───────────────────────┘
```

### 1.2 Core Components

#### 1.2.1 The Orchestrator (Brain)
- **Technology**: Large Language Model (LLM) - Gemini 1.5 Pro or equivalent
- **Pattern**: ReAct (Reasoning + Acting) loop
- **Responsibilities**:
  - Parse and decompose natural language queries
  - Determine which tools and data are needed
  - Coordinate parallel tool execution
  - Synthesize results into actionable recommendations
  - Generate SWOT analysis and Go/No-Go decisions

#### 1.2.2 Spatial Index (H3)
- **Technology**: Uber's H3 Hexagonal Hierarchical Geospatial Indexing System
- **Purpose**: Lightning-fast spatial joins between disparate datasets
- **Benefits**:
  - Uniform cell sizes for consistent analysis
  - Hierarchical resolution levels (from city-wide to street-level)
  - Efficient neighbor lookups
  - Seamless aggregation across scales

#### 1.2.3 Toolbox (The Hands)

**DemographicTool**
- Queries census data indexed by H3 cells
- Provides age distribution, income levels, lifestyle segments
- Data source: UK Census 2021 (Output Area level)

**POITool**
- Fetches nearby Points of Interest (competitors, complementary businesses)
- Data source: Overture Maps Places dataset
- Supports category-based filtering (cafes, gyms, retail, etc.)

**RealEstateTool**
- Retrieves live commercial property listings
- Filters by size, type, price, availability
- Data source: PropertyData UK API / Mock Listings

**GeometryTool**
- Calculates walk-time and drive-time isochrones (5/10 minutes)
- Performs spatial operations (buffering, intersection, union)
- Uses routing engine (OSRM or Valhalla) for realistic travel times

## 2. Process Flow

### 2.1 End-to-End Workflow

```
1. User Input
   ↓
2. Query Parsing & Reasoning
   ↓
3. Parallel Data Retrieval
   ├─→ Demographics
   ├─→ POI Data
   ├─→ Real Estate Listings
   └─→ Geometry Calculations
   ↓
4. Spatial Synthesis (H3 Overlay)
   ↓
5. Hotspot Identification
   ↓
6. Site Filtering
   ↓
7. Cannibalization Check
   ↓
8. SWOT Analysis Generation
   ↓
9. Output (Top 3 Sites + Map)
```

### 2.2 Detailed Process Steps

**Step 1: User Input**
- User submits natural language query
- Example: "Find a high-traffic spot near trendy cafes for a Gen Z clothing brand"

**Step 2: Reasoning (LLM Orchestrator)**
- LLM decomposes query into structured requirements:
  - Target demographic: Gen Z (ages 18-24)
  - Proximity requirement: Near cafes
  - Traffic requirement: High foot traffic
  - Business type: Clothing retail
- LLM determines required tools and data layers

**Step 3: Data Retrieval (Parallel Tool Execution)**
- DemographicTool: Fetch Gen Z density by H3 cell
- POITool: Fetch cafe locations and existing clothing stores
- RealEstateTool: Fetch available commercial spaces
- GeometryTool: Calculate isochrones for candidate locations

**Step 4: Spatial Synthesis**
- Overlay all data layers onto H3 hexagonal grid
- Aggregate metrics within each H3 cell:
  - Gen Z population count
  - Number of cafes within 200m
  - Foot traffic intensity
  - Competitor density

**Step 5: Hotspot Identification**
- Score each H3 cell based on query requirements
- Identify top-scoring cells as "hotspots"
- Ranking criteria:
  - High target demographic density
  - Presence of complementary businesses
  - High foot traffic
  - Low competitor saturation

**Step 6: Site Filtering**
- Filter real estate listings within hotspot cells
- Apply constraints:
  - Size requirements (e.g., 500-2000 sq ft for retail)
  - Price range (if specified)
  - Property type (commercial, retail)
  - Availability status (currently available)

**Step 7: Cannibalization Check**
- Compare candidate sites against user's existing store locations
- Flag sites within configurable distance threshold (e.g., 1km)
- Calculate potential overlap in catchment areas using isochrones

**Step 8: Final Evaluation**
- LLM generates SWOT analysis for top 3 sites
- Each SWOT includes:
  - **Strengths**: High Gen Z density, proximity to cafes, available property
  - **Weaknesses**: Higher rent, limited parking, narrow storefront
  - **Opportunities**: Growing neighborhood, new transit line planned
  - **Threats**: Competitor opening nearby, gentrification risk
- Provide Go/No-Go recommendation with confidence score

**Step 9: Output**
- Curated list of top 3 sites with detailed analysis
- Interactive map showing:
  - Candidate locations
  - Catchment area isochrones
  - Demographic heatmaps
  - POI markers (competitors, complements)
- Explainable citations for each data point used

## 3. Comparison & Unique Selling Proposition

### 3.1 Traditional GIS Tools vs. Smart Site Selection Geo-Agent

| Aspect | Traditional GIS Tools | Smart Site Selection Geo-Agent |
|--------|----------------------|-------------------------------|
| **Approach** | Descriptive (shows what is there) | Prescriptive (tells you where to go) |
| **User Interface** | Technical maps and layers | Natural language queries |
| **Expertise Required** | GIS specialist needed | Business user friendly |
| **Output** | Raw data visualizations | Actionable recommendations with reasoning |
| **Decision Support** | User interprets data | Agent provides Go/No-Go decisions |
| **Time to Insight** | Days to weeks | Minutes |

### 3.2 How It Solves the Problem

**Problem**: Site selection requires synthesizing multiple data sources, spatial analysis expertise, and business judgment. This process typically takes weeks and requires specialized GIS analysts.

**Solution**: The agent bridges the gap between raw data and business strategy by:
1. **Automating data integration**: Pulls from multiple sources automatically
2. **Applying spatial intelligence**: Uses H3 indexing for fast, accurate analysis
3. **Reasoning about business context**: LLM understands brand positioning and market dynamics
4. **Providing explainable recommendations**: Every decision is backed by cited data

**Result**: Site selection cycle reduced from weeks to minutes, democratizing access to expert-level analysis.

### 3.3 Unique Selling Proposition (USP)

**"Agentic Insight"** - The system doesn't just display data; it reasons about business implications.

Example:
- **Traditional Tool**: "This area has 45% Gen Z population density"
- **Smart Site Selection Geo-Agent**: "This listing at 123 High Street is a superior choice because it's within a 5-minute walk of 8 trendy cafes, sits in H3 cell #8912 with 45% Gen Z density (highest in the region), has 2,000 daily foot traffic, and is 1.2km from your nearest store (no cannibalization risk). The available 1,200 sq ft space is ideal for your brand format. **Recommendation: GO**"

The agent replaces a GIS expert, not just a map.

## 4. Data Sources & Integration

### 4.1 Starter Datasets

| Data Type | Source | Purpose | Update Frequency |
|-----------|--------|---------|------------------|
| **Infrastructure** | Overture Maps (Buildings & Places) | Building footprints, POI locations | Quarterly |
| **Demographics** | UK Census 2021 (Output Area level) | Age, income, lifestyle segments | Annual |
| **Amenities** | OpenStreetMap via Overpass API | Cafes, gyms, transit, competitors | Real-time |
| **Property Listings** | PropertyData UK API / Mock Listings | Available commercial spaces | Daily |
| **Foot Traffic** | Commercial provider (e.g., Placer.ai) | Pedestrian volume heatmaps | Weekly |

### 4.2 Data Processing Pipeline

```
Raw Data Sources
    ↓
ETL Pipeline
    ↓
H3 Indexing & Aggregation
    ↓
Spatial Database (PostGIS)
    ↓
Tool APIs (DemographicTool, POITool, etc.)
    ↓
LLM Orchestrator
```

### 4.3 H3 Resolution Strategy

- **Resolution 8** (~0.46 km² per cell): City-wide hotspot identification
- **Resolution 9** (~0.10 km² per cell): Neighborhood-level analysis
- **Resolution 10** (~0.02 km² per cell): Street-level precision for final site selection

## 5. Technology Stack

### 5.1 Core Technologies

- **LLM Orchestrator**: Gemini 1.5 Pro (or GPT-4, Claude 3)
- **Spatial Indexing**: H3 (Uber's Hexagonal Hierarchical Geospatial Indexing)
- **Spatial Database**: PostGIS (PostgreSQL with spatial extensions)
- **Routing Engine**: OSRM (Open Source Routing Machine) or Valhalla
- **Backend Framework**: Python (FastAPI or Flask)
- **Frontend**: React with Mapbox GL JS for interactive maps

### 5.2 Key Libraries

- **h3-py**: Python bindings for H3
- **geopandas**: Geospatial data manipulation
- **shapely**: Geometric operations
- **requests**: API integration
- **langchain** or **llamaindex**: LLM orchestration framework

## 6. Correctness Properties

### 6.1 Spatial Accuracy Properties

**Property 6.1.1: Distance Calculation Accuracy**
- For any two points A and B, the calculated distance must be within 50 meters of the true geodesic distance
- **Validates**: Requirements 3.7.3

**Property 6.1.2: H3 Cell Containment**
- For any point P and H3 cell C, if P is reported as being in C, then P must be within the geometric boundary of C
- **Validates**: Requirements 3.2.5, 3.7.1

**Property 6.1.3: Isochrone Consistency**
- For any location L and travel time T, all points within the T-minute isochrone must be reachable within T minutes via the road network
- **Validates**: Requirements 3.4.2, 3.4.3, 3.7.4

### 6.2 Data Integration Properties

**Property 6.2.1: Spatial Join Completeness**
- When overlaying demographic data onto H3 cells, the sum of all cell populations must equal the total population of the source data
- **Validates**: Requirements 3.2.4, 3.4.4

**Property 6.2.2: POI Proximity Accuracy**
- For any candidate site S and POI category C, if the system reports N POIs of category C within distance D, then there must be exactly N POIs of category C within distance D of S
- **Validates**: Requirements 3.2.2

### 6.3 Business Logic Properties

**Property 6.3.1: Cannibalization Detection**
- For any candidate site C and existing store E, if the distance between C and E is less than the cannibalization threshold T, then C must be flagged as a cannibalization risk
- **Validates**: Requirements 3.4.1

**Property 6.3.2: SWOT Completeness**
- For any site recommendation, the SWOT analysis must include at least one item in each category (Strength, Weakness, Opportunity, Threat)
- **Validates**: Requirements 3.5.2

**Property 6.3.3: Citation Traceability**
- For any recommendation R, every claim in R must have a corresponding citation that includes: data source, metric name, and specific value
- **Validates**: Requirements 3.9.1, 3.9.2

### 6.4 Performance Properties

**Property 6.4.1: Response Time Bound**
- For any valid query Q, the system must return a complete recommendation within 30 seconds
- **Validates**: Requirements 3.6.1

**Property 6.4.2: Query Success Rate**
- For any set of 100 valid natural language queries, at least 90 must be successfully parsed and processed
- **Validates**: Success Metrics

## 7. Testing Strategy

### 7.1 Property-Based Testing Framework

**Framework**: Hypothesis (Python)

**Test Data Generators**:
- Random geographic coordinates within UK bounds
- Random H3 cells at various resolutions
- Synthetic demographic distributions
- Mock POI datasets with known spatial relationships
- Synthetic property listings

### 7.2 Unit Testing

- Test each tool independently (DemographicTool, POITool, etc.)
- Test H3 indexing and spatial operations
- Test LLM prompt parsing and tool selection logic

### 7.3 Integration Testing

- Test end-to-end workflow with known queries
- Validate data flow between components
- Test parallel tool execution and result synthesis

### 7.4 Acceptance Testing

- Test against real-world scenarios with known good/bad locations
- Validate SWOT analysis quality with domain experts
- User acceptance testing with retail brand managers

## 8. Scalability & Future Enhancements

### 8.1 Regional Expansion

- **Phase 1**: North London (pilot)
- **Phase 2**: Greater London
- **Phase 3**: All of UK
- **Phase 4**: International (requires new data sources per region)

### 8.2 Future Features

- Multi-brand portfolio optimization (find optimal locations for multiple stores simultaneously)
- Temporal analysis (predict future demographic shifts)
- Competitive intelligence (track competitor expansion patterns)
- Financial modeling integration (ROI projections)
- Mobile app for field validation

### 8.3 Performance Optimization

- Implement caching layer for frequently accessed H3 cells
- Pre-compute demographic aggregations at multiple H3 resolutions
- Use vector tiles for efficient map rendering
- Implement query result caching with TTL

## 9. Security & Privacy

### 9.1 Data Privacy

- Aggregate demographic data to prevent individual identification
- Comply with GDPR for UK data handling
- Anonymize user query logs

### 9.2 API Security

- Implement API key authentication for tool access
- Rate limiting to prevent abuse
- Secure storage of third-party API credentials

## 10. Monitoring & Observability

### 10.1 Key Metrics

- Query success rate
- Average response time
- Tool execution times (per tool)
- LLM token usage and cost
- User satisfaction ratings
- Recommendation acceptance rate (Go decisions that lead to actual site selection)

### 10.2 Logging

- Log all user queries (anonymized)
- Log tool execution traces for debugging
- Log LLM reasoning chains for explainability audits

### 10.3 Alerting

- Alert on response time > 30 seconds
- Alert on query failure rate > 10%
- Alert on data source unavailability
