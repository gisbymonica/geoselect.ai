# Design: Smart Site Selection Geo-Agent

## 1. System Architecture (AWS-Native Agentic Stack)

The Smart Site Selection Geo-Agent leverages **Amazon Bedrock Agents** to move beyond simple prompts into a fully autonomous runtime. The architecture follows an **Agentic Workflow pattern**, where a central "Brain" (LLM) orchestrates specialized "Tools" to deliver prescriptive site recommendations.

### 1.1 Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    User Interface Layer                      │
│              (Web UI - Natural Language Input)               │
└───────────────────────────┬─────────────────────────────────┘
                            │
┌───────────────────────────▼─────────────────────────────────┐
│              Amazon Bedrock Agents (Orchestrator)            │
│         Anthropic Claude 3.5 Sonnet (Reasoning Brain)       │
│           ReAct Loop (Reasoning + Acting + Reflection)      │
└───────────────────────────┬─────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼──────┐  ┌────────▼────────┐
│ DemographicTool│  │SearchTool   │  │SpatialAnalysis  │
│ (Lambda)       │  │(Lambda)     │  │Tool (Lambda)    │
└───────┬────────┘  └──────┬──────┘  └────────┬────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
┌───────▼────────┐  ┌──────▼──────┐  ┌────────▼────────┐
│ Amazon RDS     │  │External APIs│  │  Amazon S3      │
│ (PostGIS)      │  │(Listings)   │  │(Knowledge Base) │
└────────────────┘  └─────────────┘  └─────────────────┘
                            │
                ┌───────────▼───────────┐
                │   H3 Spatial Index    │
                │  (Hexagonal Grid)     │
                └───────────────────────┘
```

### 1.2 Core Components

#### 1.2.1 Orchestration: Amazon Bedrock Agents
- **Technology**: Amazon Bedrock Agents with Anthropic Claude 3.5 Sonnet
- **Pattern**: ReAct (Reasoning + Acting + Reflection) loop
- **Capabilities**:
  - **Perception**: Receives natural language intent via web UI
  - **Reasoning**: LLM breaks goals into tasks (e.g., "I need to find POIs first, then demographics")
  - **Tool Execution**: Invokes Lambda action groups to query PostGIS and external APIs
  - **Reflection**: Reviews tool outputs; if a site is unsuitable, autonomously adjusts search parameters
  - **Output**: Delivers finalized, prioritized list with prescriptive SWOT analysis
- **Responsibilities**:
  - Parse and decompose natural language queries
  - Determine which tools and data are needed
  - Coordinate parallel tool execution
  - Synthesize results into actionable recommendations
  - Generate SWOT analysis and Go/No-Go decisions
  - Maintain detailed reasoning logs for transparency and auditability
  - Apply context-aware filtering based on brand "vibe" and positioning
  - Autonomously discover ideal parameters without user-specified filters

#### 1.2.2 Knowledge Base: Amazon Bedrock Knowledge Bases
- **Technology**: Amazon Bedrock Knowledge Bases connected to Amazon S3
- **Purpose**: Provides contextual information for enhanced decision-making
- **Contents**:
  - City development plans
  - Local zoning regulations
  - Historical site performance data
  - Market trend reports
  - Transit expansion plans
- **Benefits**:
  - Enables agent to consider regulatory constraints
  - Provides context on future area development
  - Supports compliance with local requirements

#### 1.2.3 Spatial Index (H3)
- **Technology**: Uber's H3 Hexagonal Hierarchical Geospatial Indexing System
- **Purpose**: Lightning-fast spatial joins between disparate datasets
- **Benefits**:
  - Uniform cell sizes for consistent analysis
  - Hierarchical resolution levels (from city-wide to street-level)
  - Efficient neighbor lookups
  - Seamless aggregation across scales

#### 1.2.4 Toolbox (Action Groups - AWS Lambda Functions)

The agent orchestrates specialized geospatial tasks through AWS Lambda-based action groups:

**DemographicTool (Lambda)**
- Queries census data indexed by H3 cells from Amazon RDS (PostGIS)
- Provides age distribution, income levels, lifestyle segments
- Data source: UK Census 2021 (Output Area level)
- Executes PostGIS spatial queries (ST_Within, ST_Intersects)

**SearchTool (Lambda)**
- Scrapes commercial property listings from external APIs
- Retrieves live commercial property listings
- Filters by size, type, price, availability
- Data source: PropertyData UK API / Mock Listings
- Handles API rate limiting and error responses

**SpatialAnalysisTool (Lambda)**
- Performs advanced PostGIS queries on Amazon RDS
- Executes spatial operations: ST_Within, ST_Distance, ST_Buffer
- Calculates walk-time and drive-time isochrones (5/10 minutes)
- Performs spatial joins to aggregate demographic data
- Integrates with routing engine (OSRM or Valhalla) for realistic travel times
- Implements cannibalization detection using distance calculations

**POITool (Lambda)**
- Fetches nearby Points of Interest (competitors, complementary businesses)
- Data source: Overture Maps Places dataset
- Supports category-based filtering (cafes, gyms, retail, etc.)
- Queries PostGIS database for proximity searches
- Aggregates POI data by H3 cells

## 2. AI-Assisted Development Workflow

This project was developed using a **"Human-in-the-loop" AI development sprint** methodology, leveraging modern AI-powered development tools to accelerate delivery while maintaining quality.

### 2.1 Development Approach

**Spec-Driven Development**
- Used Kiro (AI IDE) to transform initial prompts into detailed technical specifications
- AI-generated boilerplate code for common patterns (Lambda handlers, PostGIS queries, API integrations)
- Automated generation of test scaffolding and property-based tests
- Iterative refinement of specifications through AI-assisted analysis

**Vertical Slicing**
- Each feature (e.g., "Cannibalization Check") developed as an isolated slice
- Complete implementation from database schema to agent tool
- Validated by AI-generated tests before integration
- Enables parallel development and independent testing

**Incremental Validation**
- Individual tools tested in isolation using Amazon Bedrock Agents before integration
- Property-based testing to validate correctness across input ranges
- Integration testing with mock data before live API connections
- Continuous validation throughout development cycle

### 2.2 Benefits of AI-Assisted Development

- **Accelerated Development**: Reduced boilerplate coding time by 60-70%
- **Improved Quality**: AI-generated tests catch edge cases humans might miss
- **Consistent Patterns**: Standardized code structure across all Lambda functions
- **Documentation**: Auto-generated inline documentation and API specs
- **Rapid Iteration**: Quick prototyping and refinement of complex spatial algorithms

## 3. Process Flow

### 3.1 End-to-End Workflow

```
1. Perception (User Input via Web UI)
   ↓
2. Reasoning (Amazon Bedrock Agent - Claude 3.5 Sonnet)
   ↓
3. Tool Execution (Lambda Action Groups)
   ├─→ DemographicTool (PostGIS Query)
   ├─→ POITool (PostGIS Query)
   ├─→ SearchTool (External API Scraping)
   └─→ SpatialAnalysisTool (PostGIS Spatial Operations)
   ↓
4. Spatial Synthesis (H3 Overlay)
   ↓
5. Hotspot Identification
   ↓
6. Site Filtering
   ↓
7. Cannibalization Check (SpatialAnalysisTool)
   ↓
8. Reflection (Agent Reviews & Adjusts)
   ↓
9. SWOT Analysis Generation
   ↓
10. Output (Top 3 Sites + Map + Reasoning Log)
```

### 3.2 Detailed Process Steps

**Step 1: Perception (User Input)**
- User submits natural language query via web UI
- Example: "Find a high-traffic spot near trendy cafes for a Gen Z clothing brand"
- Query is sent to Amazon Bedrock Agent

**Step 2: Reasoning (Amazon Bedrock Agent)**
- Claude 3.5 Sonnet decomposes query into structured requirements:
  - Target demographic: Gen Z (ages 18-24)
  - Proximity requirement: Near cafes
  - Traffic requirement: High foot traffic
  - Business type: Clothing retail
  - Brand vibe: Trendy, youth-oriented
- Agent determines required tools and data layers
- Agent plans execution sequence (e.g., "I need to find POIs first, then demographics")
- Reasoning steps are logged for transparency and auditability
- **Autonomous Discovery**: Agent discovers ideal parameters without user-specified filters

**Step 3: Tool Execution (Lambda Action Groups)**
- **DemographicTool**: Executes PostGIS query on Amazon RDS to fetch Gen Z density by H3 cell
- **POITool**: Queries PostGIS for cafe locations and existing clothing stores
- **SearchTool**: Scrapes external APIs for available commercial spaces
- **SpatialAnalysisTool**: Calculates isochrones using PostGIS ST_Buffer and routing engine
- Tools execute in parallel where possible for performance
- Results are returned to the agent for synthesis

**Step 4: Spatial Synthesis**
- Overlay all data layers onto H3 hexagonal grid
- Aggregate metrics within each H3 cell:
  - Gen Z population count
  - Number of cafes within 200m
  - Foot traffic intensity (from live feeds)
  - Competitor density
  - Transit accessibility (from real-time transit data)

**Step 5: Hotspot Identification**
- Score each H3 cell based on query requirements
- Apply context-aware filtering based on brand vibe
- Identify top-scoring cells as "hotspots"
- Ranking criteria:
  - High target demographic density
  - Presence of complementary businesses
  - High foot traffic (real-time data)
  - Low competitor saturation
  - Brand-context alignment (e.g., luxury area for premium brands)
  - Transit accessibility and upcoming events

**Step 6: Site Filtering**
- Filter real estate listings within hotspot cells
- Apply constraints:
  - Size requirements (e.g., 500-2000 sq ft for retail)
  - Price range (if specified)
  - Property type (commercial, retail)
  - Availability status (currently available)
  - Zoning compliance (from Knowledge Base)

**Step 7: Cannibalization Check**
- SpatialAnalysisTool executes PostGIS ST_Distance queries
- Compare candidate sites against user's existing store locations
- Flag sites within configurable distance threshold (e.g., 1km)
- Calculate potential overlap in catchment areas using isochrones

**Step 8: Reflection (Agent Reviews & Adjusts)**
- Agent reviews tool outputs and intermediate results
- If a site is unsuitable, agent autonomously adjusts search parameters
- Agent may invoke additional tools for clarification
- Iterative refinement until optimal sites are identified
- **Real-Time Adaptability**: Considers live transit and event feeds

**Step 9: SWOT Analysis Generation**
- Claude 3.5 Sonnet generates SWOT analysis for top 3 sites
- Consults Knowledge Base for regulatory and development context
- Each SWOT includes:
  - **Strengths**: High Gen Z density, proximity to cafes, available property, transit access
  - **Weaknesses**: Higher rent, limited parking, narrow storefront
  - **Opportunities**: Growing neighborhood, new transit line planned, upcoming events
  - **Threats**: Competitor opening nearby, gentrification risk, zoning changes
- Provide Go/No-Go recommendation with confidence score
- All claims backed by specific data citations

**Step 10: Output**
- Curated list of top 3 sites with detailed analysis
- Interactive map showing:
  - Candidate locations
  - Catchment area isochrones
  - Demographic heatmaps
  - POI markers (competitors, complements)
  - Real-time transit routes
- Explainable citations for each data point used
- **Reasoning Log**: Complete trace of decision-making process for transparency and human oversight
- Prescriptive recommendations with actionable next steps

## 4. Comparison & Unique Selling Proposition

### 4.1 Traditional GIS Tools vs. Smart Site Selection Geo-Agent

| Aspect | Traditional GIS Tools | Smart Site Selection Geo-Agent |
|--------|----------------------|-------------------------------|
| **Approach** | Descriptive (shows what is there) | Prescriptive (tells you where to go) |
| **User Interface** | Technical maps and layers | Natural language queries |
| **Expertise Required** | GIS specialist needed | Business user friendly |
| **Output** | Raw data visualizations | Actionable recommendations with reasoning |
| **Decision Support** | User interprets data | Agent provides Go/No-Go decisions |
| **Time to Insight** | Days to weeks | Minutes |
| **Parameter Discovery** | User must know what to filter | **Autonomous Discovery**: Agent discovers ideal parameters |
| **Data Freshness** | Historical data only | **Real-Time Adaptability**: Live transit and event feeds |
| **Adaptability** | Static analysis | **Dynamic Adjustment**: Agent autonomously refines search |

### 4.2 How It Solves the Problem

**Problem**: Site selection requires synthesizing multiple data sources, spatial analysis expertise, and business judgment. This process typically takes weeks and requires specialized GIS analysts. Traditional tools require users to know exactly what filters to apply, and they rely on outdated historical data.

**Solution**: The agent bridges the gap between raw data and business strategy by:
1. **Automating data integration**: Pulls from multiple sources automatically
2. **Applying spatial intelligence**: Uses H3 indexing for fast, accurate analysis
3. **Reasoning about business context**: LLM understands brand positioning and market dynamics
4. **Providing explainable recommendations**: Every decision is backed by cited data
5. **Autonomous Discovery**: Discovers ideal parameters without user-specified filters
6. **Real-Time Adaptability**: Integrated with live transit and event feeds to value sites based on today's reality, not last year's census
7. **Self-Correction**: Reflection loop allows agent to adjust search parameters autonomously

**Result**: Site selection cycle reduced from weeks to minutes, democratizing access to expert-level analysis with real-time market intelligence.

### 4.3 Unique Selling Proposition (USP)

**"Autonomous Discovery with Real-Time Intelligence"**

The system doesn't just display data or require manual filtering; it autonomously discovers optimal parameters and adapts to real-time conditions.

**Key Differentiators:**

1. **Autonomous Discovery**: Unlike competitors that require users to know what to filter, our agent discovers the ideal parameters itself through intelligent reasoning and exploration.

2. **Real-Time Adaptability**: Integrated with live transit and event feeds to value sites based on today's reality, not last year's census. The agent considers:
   - Current transit schedules and delays
   - Upcoming events and festivals
   - Recent competitor openings
   - Real-time foot traffic patterns
   - Breaking development news

3. **Self-Correcting Intelligence**: The reflection loop enables the agent to review its own outputs and autonomously adjust search parameters if initial results are unsuitable.

4. **Regulatory Awareness**: Knowledge Base integration ensures recommendations comply with local zoning and development regulations.

Example:
- **Traditional Tool**: "This area has 45% Gen Z population density. You need to manually filter by income, transit access, and competitors."
- **Smart Site Selection Geo-Agent**: "I've autonomously discovered that 123 High Street is optimal. It's within a 5-minute walk of 8 trendy cafes, sits in H3 cell #8912 with 45% Gen Z density (highest in the region), has 2,000 daily foot traffic (verified via today's live feed), benefits from the new metro line opening next month, and is 1.2km from your nearest store (no cannibalization risk). The available 1,200 sq ft space is ideal for your brand format. I've also verified zoning compliance with local regulations. **Recommendation: GO**"

The agent replaces a GIS expert, a market analyst, and a regulatory consultant—not just a map.

## 5. Data Sources & Integration

### 5.1 Starter Datasets

| Data Type | Source | Purpose | Update Frequency | Storage |
|-----------|--------|---------|------------------|---------|
| **Infrastructure** | Overture Maps (Buildings & Places) | Building footprints, POI locations | Quarterly | Amazon RDS (PostGIS) |
| **Demographics** | UK Census 2021 (Output Area level) | Age, income, lifestyle segments | Annual | Amazon RDS (PostGIS) |
| **Amenities** | OpenStreetMap via Overpass API | Cafes, gyms, transit, competitors | Real-time | Amazon RDS (PostGIS) |
| **Property Listings** | PropertyData UK API / Mock Listings | Available commercial spaces | Daily | External API (scraped) |
| **Foot Traffic** | Commercial provider (e.g., Placer.ai) | Pedestrian volume heatmaps | Real-time | Live API integration |
| **Transit Data** | Live transit feeds | Current schedules, delays, routes | Real-time | Live API integration |
| **Event Data** | Event APIs | Upcoming festivals, concerts, markets | Real-time | Live API integration |
| **Regulations** | City planning documents | Zoning, development plans | Quarterly | Amazon S3 (Knowledge Base) |

### 5.2 Data Processing Pipeline

```
Raw Data Sources
    ↓
ETL Pipeline (AWS Glue / Lambda)
    ↓
H3 Indexing & Aggregation
    ↓
Amazon RDS (PostGIS) + Amazon S3 (Knowledge Base)
    ↓
Lambda Action Groups (Tools)
    ↓
Amazon Bedrock Agent (Orchestrator)
    ↓
Web UI (User)
```

### 5.3 H3 Resolution Strategy

- **Resolution 8** (~0.46 km² per cell): City-wide hotspot identification
- **Resolution 9** (~0.10 km² per cell): Neighborhood-level analysis
- **Resolution 10** (~0.02 km² per cell): Street-level precision for final site selection

## 6. Technology Stack

### 6.1 Core AWS Technologies

- **Orchestration**: Amazon Bedrock Agents with Anthropic Claude 3.5 Sonnet
- **Knowledge Base**: Amazon Bedrock Knowledge Bases (S3-backed)
- **Compute**: AWS Lambda (for action group tools)
- **Database**: Amazon RDS with PostGIS extension
- **Storage**: Amazon S3 (for Knowledge Base and data lake)
- **ETL**: AWS Glue or Lambda for data processing
- **API Gateway**: Amazon API Gateway (for web UI integration)
- **Monitoring**: Amazon CloudWatch (logs and metrics)
- **Security**: AWS IAM, AWS Secrets Manager

### 6.2 Spatial Technologies

- **Spatial Indexing**: H3 (Uber's Hexagonal Hierarchical Geospatial Indexing)
- **Spatial Database**: PostGIS (PostgreSQL with spatial extensions)
- **Routing Engine**: OSRM (Open Source Routing Machine) or Valhalla

### 6.3 Key Libraries (Lambda Functions)

- **h3-py**: Python bindings for H3
- **psycopg2**: PostgreSQL adapter for Python
- **boto3**: AWS SDK for Python
- **requests**: API integration
- **shapely**: Geometric operations
- **geopandas**: Geospatial data manipulation (for ETL)

## 7. Correctness Properties

### 7.1 Spatial Accuracy Properties

**Property 7.1.1: Distance Calculation Accuracy**
- For any two points A and B, the calculated distance must be within 50 meters of the true geodesic distance
- **Validates**: Requirements 3.7.3

**Property 7.1.2: H3 Cell Containment**
- For any point P and H3 cell C, if P is reported as being in C, then P must be within the geometric boundary of C
- **Validates**: Requirements 3.2.5, 3.7.1

**Property 7.1.3: Isochrone Consistency**
- For any location L and travel time T, all points within the T-minute isochrone must be reachable within T minutes via the road network
- **Validates**: Requirements 3.4.2, 3.4.3, 3.7.4

### 7.2 Data Integration Properties

**Property 7.2.1: Spatial Join Completeness**
- When overlaying demographic data onto H3 cells, the sum of all cell populations must equal the total population of the source data
- **Validates**: Requirements 3.2.4, 3.4.4

**Property 7.2.2: POI Proximity Accuracy**
- For any candidate site S and POI category C, if the system reports N POIs of category C within distance D, then there must be exactly N POIs of category C within distance D of S
- **Validates**: Requirements 3.2.2

### 7.3 Business Logic Properties

**Property 7.3.1: Cannibalization Detection**
- For any candidate site C and existing store E, if the distance between C and E is less than the cannibalization threshold T, then C must be flagged as a cannibalization risk
- **Validates**: Requirements 3.4.1

**Property 7.3.2: SWOT Completeness**
- For any site recommendation, the SWOT analysis must include at least one item in each category (Strength, Weakness, Opportunity, Threat)
- **Validates**: Requirements 3.5.2

**Property 7.3.3: Citation Traceability**
- For any recommendation R, every claim in R must have a corresponding citation that includes: data source, metric name, and specific value
- **Validates**: Requirements 3.9.1, 3.9.2

### 7.4 Performance Properties

**Property 7.4.1: Response Time Bound**
- For any valid query Q, the system must return a complete recommendation within 30 seconds
- **Validates**: Requirements 3.6.1

**Property 7.4.2: Query Success Rate**
- For any set of 100 valid natural language queries, at least 90 must be successfully parsed and processed
- **Validates**: Success Metrics

## 8. Testing Strategy

### 8.1 Property-Based Testing Framework

**Framework**: Hypothesis (Python) for Lambda function testing

**Test Data Generators**:
- Random geographic coordinates within UK bounds
- Random H3 cells at various resolutions
- Synthetic demographic distributions
- Mock POI datasets with known spatial relationships
- Synthetic property listings

### 8.2 Unit Testing

- Test each Lambda action group independently (DemographicTool, POITool, SearchTool, SpatialAnalysisTool)
- Test H3 indexing and spatial operations
- Test PostGIS query generation and execution
- Mock external API calls for isolated testing

### 8.3 Integration Testing

- Test end-to-end workflow with Amazon Bedrock Agents in test environment
- Validate data flow between Lambda functions and RDS
- Test parallel tool execution and result synthesis
- Test Knowledge Base integration

### 8.4 Agent Testing

- Test agent reasoning with various natural language inputs
- Validate tool selection logic
- Test reflection and self-correction capabilities
- Verify reasoning log generation

### 8.5 Acceptance Testing

- Test against real-world scenarios with known good/bad locations
- Validate SWOT analysis quality with domain experts
- User acceptance testing with retail brand managers
- Verify real-time data integration accuracy

## 9. Scalability & Future Enhancements

### 9.1 Regional Expansion

- **Phase 1**: North London (pilot)
- **Phase 2**: Greater London
- **Phase 3**: All of UK
- **Phase 4**: International (requires new data sources per region)

### 9.2 Future Features

- Multi-brand portfolio optimization (find optimal locations for multiple stores simultaneously)
- Temporal analysis (predict future demographic shifts)
- Competitive intelligence (track competitor expansion patterns)
- Financial modeling integration (ROI projections)
- Mobile app for field validation

### 9.3 Performance Optimization

- Implement caching layer (Amazon ElastiCache) for frequently accessed H3 cells
- Pre-compute demographic aggregations at multiple H3 resolutions
- Use Amazon CloudFront for efficient map rendering
- Implement query result caching with TTL
- Optimize Lambda cold start times with provisioned concurrency
- Use RDS read replicas for query load distribution

## 10. Security & Privacy

### 10.1 Data Privacy

- Aggregate demographic data to prevent individual identification
- Comply with GDPR for UK data handling
- Anonymize user query logs
- Encrypt foot traffic and proprietary brand data at rest (S3 encryption) and in transit (TLS)
- Implement data isolation between different brand clients using IAM policies
- Ensure no personally identifiable information (PII) is stored or processed
- Use AWS Secrets Manager for API credentials

### 10.2 API Security

- Implement API key authentication via Amazon API Gateway
- Rate limiting to prevent abuse (API Gateway throttling)
- Secure storage of third-party API credentials (AWS Secrets Manager)
- Regular security audits and penetration testing
- Use AWS WAF for web application firewall protection
- Implement least-privilege IAM roles for Lambda functions

### 10.3 Ethical AI & Bias Mitigation

- **Bias Detection**: Implement automated checks to detect demographic bias patterns in recommendations
- **Fairness Metrics**: Track and report on diversity of recommended locations across socioeconomic zones
- **Audit Trails**: Maintain comprehensive logs of all recommendations for periodic bias audits
- **Human Oversight**: Ensure reasoning logs enable human review of AI decision-making
- **Transparency**: Clearly communicate when recommendations are based on incomplete or uncertain data
- **Guardrails**: Implement rules to prevent recommendations that could reinforce historical inequities

## 11. Monitoring & Observability

### 11.1 Key Metrics (Amazon CloudWatch)

- Query success rate
- Average response time (end-to-end and per Lambda function)
- Tool execution times (per action group)
- LLM token usage and cost (Bedrock metrics)
- User satisfaction ratings
- Recommendation acceptance rate (Go decisions that lead to actual site selection)
- Lambda invocation counts and error rates
- RDS query performance metrics
- API Gateway request counts and latency

### 11.2 Logging (Amazon CloudWatch Logs)

- Log all user queries (anonymized)
- Log tool execution traces for debugging
- Log agent reasoning chains for explainability audits
- Log Lambda function execution details
- Log PostGIS query performance
- Structured logging with correlation IDs for request tracing

### 11.3 Alerting (Amazon CloudWatch Alarms)

- Alert on response time > 30 seconds
- Alert on query failure rate > 10%
- Alert on data source unavailability
- Alert on Lambda function errors or throttling
- Alert on RDS connection pool exhaustion
- Alert on unusual cost spikes (Bedrock API usage)

### 11.4 Distributed Tracing

- Use AWS X-Ray for end-to-end request tracing
- Track latency across Lambda functions and RDS queries
- Identify performance bottlenecks in the agent workflow
