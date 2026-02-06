# Implementation Plan: Smart Site Selection Geo-Agent

## Overview

This implementation plan breaks down the Smart Site Selection Geo-Agent into discrete coding tasks. The system uses an **AWS-native agentic workflow pattern** where **Amazon Bedrock Agents** (with Anthropic Claude 3.5 Sonnet) orchestrate specialized **AWS Lambda action groups** to analyze potential retail locations using H3 hexagonal spatial indexing.

The architecture leverages:
- **Amazon Bedrock Agents** for orchestration with ReAct (Reasoning + Acting + Reflection) loop
- **Amazon Bedrock Knowledge Bases** (S3-backed) for regulatory and development context
- **AWS Lambda functions** as action group tools (DemographicTool, SearchTool, SpatialAnalysisTool, POITool)
- **Amazon RDS with PostGIS** for spatial data storage and queries
- **Amazon S3** for Knowledge Base and data lake storage

The implementation follows a bottom-up approach: first building the spatial foundation (H3 indexing and PostGIS), then individual Lambda tools, then the Bedrock Agent configuration, and finally the integration layer.

**Development Methodology**: This project uses AI-assisted development with Kiro (AI IDE) for spec-driven development, vertical slicing, and incremental validation.

## Tasks

- [ ] 1. Set up AWS infrastructure and core dependencies
  - Create AWS account and configure IAM roles with least-privilege access
  - Set up Amazon RDS PostgreSQL instance with PostGIS extension
  - Configure Amazon S3 buckets for Knowledge Base and data lake
  - Set up Amazon Bedrock access and enable Claude 3.5 Sonnet model
  - Create AWS Lambda execution roles with appropriate permissions
  - Configure Amazon API Gateway for web UI integration
  - Set up Amazon CloudWatch for logging and monitoring
  - Configure AWS Secrets Manager for API credentials
  - Install core dependencies in Lambda layers: h3-py, psycopg2, boto3, requests, shapely
  - Set up local development environment with AWS SAM or Serverless Framework
  - _Requirements: 3.8.3, 3.6.3, 3.10.3, 3.10.4_

- [ ] 2. Set up Amazon RDS PostGIS database schema
  - [ ] 2.1 Design and create PostGIS database schema
    - Create tables for demographics (with H3 cell indexing)
    - Create tables for POIs (with spatial indexes)
    - Create tables for properties (with spatial indexes)
    - Create tables for H3 cell metadata
    - Implement spatial indexes using PostGIS (GIST indexes)
    - Create database migration scripts
    - _Requirements: 3.2.5, 3.6.3_
  
  - [ ] 2.2 Implement H3 indexing in PostGIS
    - Create stored procedures for H3 cell operations
    - Implement H3 cell to polygon conversion in PostGIS
    - Create indexes on H3 cell columns for fast lookups
    - Implement hierarchical aggregation queries across H3 resolutions (8, 9, 10)
    - _Requirements: 3.2.5, 3.4.5_
  
  - [ ]* 2.3 Write database schema validation tests
    - Test spatial index creation and performance
    - Test H3 cell queries
    - Test data integrity constraints
    - _Requirements: 3.2.5_

- [ ] 3. Implement SpatialAnalysisTool Lambda function
  - [ ] 3.1 Create Lambda function for spatial operations
    - Implement Lambda handler with proper error handling
    - Implement PostGIS connection management (using RDS connection pooling)
    - Implement geodesic distance calculation using PostGIS ST_Distance
    - Implement buffering operations using PostGIS ST_Buffer
    - Implement spatial intersection and union using PostGIS ST_Intersects, ST_Union
    - Implement H3 cell operations (lat/lon to H3, H3 to polygon)
    - _Requirements: 3.7.3, 3.4.4, 3.4.5_
  
  - [ ] 3.2 Implement isochrone calculation
    - Integrate with OSRM routing engine API for walk-time isochrones (5 and 10 minutes)
    - Integrate with OSRM routing engine API for drive-time isochrones (5 and 10 minutes)
    - Implement isochrone polygon generation from routing results
    - Store isochrone polygons in PostGIS for spatial queries
    - Handle edge cases (unreachable areas, routing failures)
    - _Requirements: 3.4.2, 3.4.3, 3.7.4_
  
  - [ ] 3.3 Implement cannibalization detection
    - Implement distance-based cannibalization detection using PostGIS ST_Distance
    - Implement isochrone-based catchment area overlap calculation using PostGIS ST_Intersects
    - Implement configurable distance threshold
    - Implement flagging logic for at-risk sites
    - _Requirements: 3.4.1_
  
  - [ ]* 3.4 Write property test for distance calculation accuracy
    - **Property 7.1.1: Distance Calculation Accuracy**
    - For any two points A and B, the calculated distance must be within 50 meters of the true geodesic distance
    - **Validates: Requirements 3.7.3**
  
  - [ ]* 3.5 Write property test for isochrone consistency
    - **Property 7.1.3: Isochrone Consistency**
    - For any location L and travel time T, all points within the T-minute isochrone must be reachable within T minutes via the road network
    - **Validates: Requirements 3.4.2, 3.4.3, 3.7.4**
  
  - [ ]* 3.6 Write property test for cannibalization detection
    - **Property 7.3.1: Cannibalization Detection**
    - For any candidate site C and existing store E, if the distance between C and E is less than the cannibalization threshold T, then C must be flagged as a cannibalization risk
    - **Validates: Requirements 3.4.1**
  
  - [ ]* 3.7 Write unit tests for SpatialAnalysisTool Lambda
    - Test Lambda handler with mock events
    - Test PostGIS query generation and execution
    - Test edge cases for distance calculations (antipodal points, same location)
    - Test isochrone generation with mock routing responses
    - Test spatial operations (buffer, intersection, union)
    - Test cannibalization detection edge cases
    - _Requirements: 3.4.1, 3.4.2, 3.4.3, 3.7.3_

- [ ] 4. Checkpoint - Ensure spatial foundation tests pass
  - Run all Lambda function tests locally using AWS SAM
  - Test PostGIS queries against RDS instance
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 5. Implement DemographicTool Lambda function
  - [ ] 5.1 Create Lambda function for demographic data queries
    - Implement Lambda handler with proper error handling
    - Implement PostGIS connection to RDS for demographic data queries
    - Implement data loading from UK Census 2021 Output Area data into PostGIS
    - Implement H3 cell aggregation for demographic metrics (age distribution, income levels)
    - Implement query methods for retrieving demographics by H3 cell or geographic area using PostGIS ST_Within
    - Implement caching layer using Amazon ElastiCache (Redis) for frequently accessed cells
    - _Requirements: 3.2.1, 3.4.4, 3.6.3_
  
  - [ ]* 5.2 Write property test for spatial join completeness
    - **Property 7.2.1: Spatial Join Completeness**
    - When overlaying demographic data onto H3 cells, the sum of all cell populations must equal the total population of the source data
    - **Validates: Requirements 3.2.4, 3.4.4**
  
  - [ ]* 5.3 Write unit tests for DemographicTool Lambda
    - Test Lambda handler with mock events
    - Test demographic aggregation with synthetic data
    - Test caching behavior (ElastiCache integration)
    - Test query filtering by age range, income level
    - Test PostGIS spatial join queries
    - _Requirements: 3.2.1, 3.6.3_

- [ ] 6. Implement POITool Lambda function
  - [ ] 6.1 Create Lambda function for POI queries
    - Implement Lambda handler with proper error handling
    - Implement integration with Overture Maps Places dataset (load into PostGIS)
    - Implement OpenStreetMap Overpass API integration for real-time POI data
    - Implement category-based filtering (cafes, gyms, retail, competitors) using PostGIS queries
    - Implement proximity search within specified radius using PostGIS ST_DWithin
    - Implement H3 cell-based POI aggregation
    - _Requirements: 3.2.2, 3.7.1_
  
  - [ ]* 6.2 Write property test for POI proximity accuracy
    - **Property 7.2.2: POI Proximity Accuracy**
    - For any candidate site S and POI category C, if the system reports N POIs of category C within distance D, then there must be exactly N POIs of category C within distance D of S
    - **Validates: Requirements 3.2.2**
  
  - [ ]* 6.3 Write unit tests for POITool Lambda
    - Test Lambda handler with mock events
    - Test category filtering with mock POI data
    - Test proximity search edge cases (zero results, large result sets)
    - Test H3 aggregation correctness
    - Test PostGIS spatial queries
    - _Requirements: 3.2.2_

- [ ] 7. Implement SearchTool Lambda function for property listings
  - [ ] 7.1 Create Lambda function for property listing scraping
    - Implement Lambda handler with proper error handling
    - Implement PropertyData UK API integration (or mock API for testing)
    - Implement web scraping logic for commercial property listings
    - Implement filtering by size, type, price, availability status
    - Implement geographic filtering by bounding box or H3 cells using PostGIS
    - Implement property detail extraction (address, size, price)
    - Handle API rate limiting and error responses
    - Store scraped listings in PostGIS for spatial queries
    - _Requirements: 3.3.1, 3.3.2, 3.3.3_
  
  - [ ]* 7.2 Write unit tests for SearchTool Lambda
    - Test Lambda handler with mock events
    - Test filtering logic with mock property data
    - Test API error handling (timeout, rate limit, invalid response)
    - Test geographic filtering accuracy using PostGIS
    - Test web scraping with mock HTML responses
    - _Requirements: 3.3.1, 3.3.2, 3.3.3_

- [ ] 8. Checkpoint - Ensure all Lambda tool tests pass
  - Run all Lambda function tests locally using AWS SAM
  - Test integration with RDS PostGIS database
  - Test caching with ElastiCache (if implemented)
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 9. Implement data loading and ETL pipeline
  - [ ] 9.1 Create ETL Lambda functions or AWS Glue jobs
    - Implement UK Census 2021 data loader with H3 indexing
    - Implement Overture Maps Places data loader
    - Implement OpenStreetMap data extraction scripts
    - Implement data validation and quality checks
    - Load data into Amazon RDS PostGIS with spatial indexes
    - _Requirements: 3.2.1, 3.2.2, 3.7.1, 3.7.2_
  
  - [ ]* 9.2 Write data validation tests
    - Test data loader correctness with sample datasets
    - Test H3 indexing during data load
    - Test data quality checks
    - Test PostGIS spatial index creation
    - _Requirements: 3.7.1, 3.7.2_

- [ ] 10. Set up Amazon Bedrock Knowledge Base
  - [ ] 10.1 Create S3 bucket and upload knowledge documents
    - Create S3 bucket for Knowledge Base storage
    - Upload city development plans and zoning regulations
    - Upload local market reports and transit expansion plans
    - Organize documents with proper metadata and tags
    - _Requirements: 3.2.6, 3.10.1_
  
  - [ ] 10.2 Configure Amazon Bedrock Knowledge Base
    - Create Bedrock Knowledge Base connected to S3 bucket
    - Configure document chunking and embedding strategy
    - Set up vector database for semantic search
    - Test knowledge retrieval with sample queries
    - _Requirements: 3.2.6_
  
  - [ ]* 10.3 Write Knowledge Base integration tests
    - Test document retrieval accuracy
    - Test semantic search quality
    - Test regulatory compliance queries
    - _Requirements: 3.2.6_
- [ ] 11. Implement spatial synthesis Lambda function
  - [ ] 11.1 Create SpatialSynthesizer Lambda function
    - Implement Lambda handler for data aggregation
    - Implement multi-layer data overlay onto H3 grid using PostGIS
    - Implement metric aggregation within H3 cells (demographics, POI counts, foot traffic)
    - Implement hotspot scoring algorithm based on query requirements
    - Implement context-aware scoring that considers brand vibe (luxury vs. budget, trendy vs. traditional)
    - Implement ranking and filtering of top-scoring H3 cells
    - Integrate with real-time transit and event feeds for adaptive scoring
    - _Requirements: 3.2.4, 3.2.6, 3.4.4_
  
  - [ ]* 11.2 Write unit tests for SpatialSynthesizer Lambda
    - Test Lambda handler with mock events
    - Test multi-layer overlay with synthetic data
    - Test scoring algorithm with known inputs and expected outputs
    - Test context-aware filtering with different brand vibes
    - Test ranking correctness
    - Test real-time data integration
    - _Requirements: 3.2.4, 3.2.6_

- [ ] 12. Configure Amazon Bedrock Agent
  - [ ] 12.1 Create Bedrock Agent with Claude 3.5 Sonnet
    - Create Amazon Bedrock Agent in AWS Console or via IaC (CloudFormation/Terraform)
    - Configure agent with Claude 3.5 Sonnet model
    - Write agent instructions for site selection reasoning
    - Configure ReAct (Reasoning + Acting + Reflection) prompting strategy
    - Enable reasoning log generation for transparency
    - Configure brand vibe extraction and context-aware filtering instructions
    - Implement autonomous parameter discovery instructions
    - _Requirements: 3.1.1, 3.1.2, 3.1.3, 3.1.4, 3.2.6, 3.9.5_
  
  - [ ] 12.2 Configure Action Groups (Lambda tool integrations)
    - Create action group for DemographicTool Lambda
    - Create action group for POITool Lambda
    - Create action group for SearchTool Lambda
    - Create action group for SpatialAnalysisTool Lambda
    - Create action group for SpatialSynthesizer Lambda
    - Define OpenAPI schemas for each action group
    - Configure Lambda permissions for Bedrock Agent invocation
    - _Requirements: 3.1.2_
  
  - [ ] 12.3 Integrate Knowledge Base with Agent
    - Connect Bedrock Knowledge Base to Agent
    - Configure knowledge retrieval instructions
    - Test regulatory compliance queries
    - Test development plan context retrieval
    - _Requirements: 3.2.6, 3.10.1_
  
  - [ ]* 12.4 Write agent integration tests
    - Test agent reasoning with various natural language inputs
    - Test tool selection logic
    - Test error handling for tool failures
    - Test clarifying question generation for ambiguous queries
    - Test reasoning log generation
    - Test brand vibe extraction
    - Test autonomous parameter discovery
    - Test reflection and self-correction
    - _Requirements: 3.1.1, 3.1.2, 3.1.3, 3.9.5_

- [ ] 13. Implement SWOT analysis generation Lambda function
  - [ ] 13.1 Create SWOTAnalyzer Lambda function
    - Implement Lambda handler for SWOT generation
    - Implement SWOT structure generation (Strengths, Weaknesses, Opportunities, Threats)
    - Integrate with Bedrock Agent for LLM-based SWOT content generation
    - Implement Go/No-Go recommendation logic with confidence scoring
    - Implement data citation extraction and formatting
    - Query Knowledge Base for regulatory and development context
    - _Requirements: 3.5.1, 3.5.2, 3.5.3, 3.5.4, 3.9.1, 3.9.2_
  
  - [ ]* 13.2 Write property test for SWOT completeness
    - **Property 7.3.2: SWOT Completeness**
    - For any site recommendation, the SWOT analysis must include at least one item in each category (Strength, Weakness, Opportunity, Threat)
    - **Validates: Requirements 3.5.2**
  
  - [ ]* 13.3 Write property test for citation traceability
    - **Property 7.3.3: Citation Traceability**
    - For any recommendation R, every claim in R must have a corresponding citation that includes: data source, metric name, and specific value
    - **Validates: Requirements 3.9.1, 3.9.2**
  
  - [ ]* 13.4 Write unit tests for SWOTAnalyzer Lambda
    - Test Lambda handler with mock events
    - Test SWOT structure validation
    - Test citation formatting
    - Test Go/No-Go logic with various confidence thresholds
    - Test Knowledge Base integration
    - _Requirements: 3.5.1, 3.5.2, 3.5.3, 3.5.4_

- [ ] 14. Checkpoint - Ensure Bedrock Agent and analysis tests pass
  - Test Bedrock Agent with sample queries in AWS Console
  - Verify all action groups are properly configured
  - Test Knowledge Base integration
  - Test SWOT generation with real data
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 15. Implement end-to-end workflow integration
  - [ ] 15.1 Create orchestration workflow
    - Configure Bedrock Agent to orchestrate complete workflow (Steps 1-10 from design)
    - Wire together: Agent → Lambda Action Groups → PostGIS → Knowledge Base → SWOT
    - Implement result formatting and output generation
    - Implement top-3 site selection logic
    - Test reflection loop (agent reviews and adjusts parameters)
    - _Requirements: 3.1.1, 3.2.4, 3.4.1, 3.5.1_
  
  - [ ]* 15.2 Write integration tests for end-to-end workflow
    - Test complete workflow with known query and expected results
    - Test with various query types (demographic-focused, POI-focused, mixed)
    - Test error propagation and recovery
    - Test autonomous parameter discovery
    - Test real-time data integration
    - Test reflection and self-correction
    - _Requirements: 3.1.1, 3.5.1_

- [ ] 16. Implement performance optimization and caching
  - [ ] 16.1 Add caching layer with Amazon ElastiCache
    - Set up Amazon ElastiCache (Redis) cluster
    - Implement Redis-based caching for H3 cell demographics
    - Implement query result caching with TTL
    - Implement POI data caching
    - Update Lambda functions to use ElastiCache
    - _Requirements: 3.6.3_
  
  - [ ] 16.2 Add performance monitoring with CloudWatch
    - Implement timing instrumentation for each Lambda function
    - Implement overall query response time tracking
    - Add CloudWatch custom metrics for performance
    - Set up CloudWatch dashboards for monitoring
    - Configure AWS X-Ray for distributed tracing
    - _Requirements: 3.6.1_
  
  - [ ] 16.3 Optimize Lambda and RDS performance
    - Configure Lambda provisioned concurrency for frequently used functions
    - Optimize PostGIS queries with proper indexes
    - Set up RDS read replicas for query load distribution
    - Implement connection pooling for RDS
    - _Requirements: 3.6.1, 3.6.2_
  
  - [ ]* 16.4 Write property test for response time bound
    - **Property 7.4.1: Response Time Bound**
    - For any valid query Q, the system must return a complete recommendation within 30 seconds
    - **Validates: Requirements 3.6.1**

- [ ] 17. Implement web UI and API Gateway integration
  - [ ] 17.1 Create API Gateway endpoints
    - Configure Amazon API Gateway REST API
    - Implement POST /query endpoint for natural language queries (invokes Bedrock Agent)
    - Implement GET /sites/{site_id} endpoint for detailed site information
    - Implement GET /health endpoint for health checks
    - Implement request validation and error handling
    - Generate API documentation with OpenAPI/Swagger
    - _Requirements: 3.1.1, 3.8.2_
  
  - [ ] 17.2 Add authentication and rate limiting
    - Implement API key authentication via API Gateway
    - Configure API Gateway usage plans and rate limiting
    - Implement request logging to CloudWatch
    - Configure AWS WAF for web application firewall protection
    - _Requirements: 3.10.3, 3.10.4_
  
  - [ ] 17.3 Create simple web UI
    - Build React-based web UI for natural language queries
    - Integrate Mapbox GL JS for interactive maps
    - Display candidate locations with isochrones and heatmaps
    - Display SWOT analysis and reasoning logs
    - Deploy UI to Amazon S3 + CloudFront
    - _Requirements: 3.1.1_
  
  - [ ]* 17.4 Write API integration tests
    - Test all endpoints with valid and invalid inputs
    - Test authentication and authorization
    - Test rate limiting behavior
    - Test concurrent request handling
    - Test Bedrock Agent invocation via API Gateway
    - _Requirements: 3.8.2_

- [ ] 18. Implement explainability and citation system
  - [ ] 18.1 Create CitationManager Lambda function
    - Implement Lambda handler for citation tracking
    - Implement citation tracking throughout the workflow
    - Implement citation formatting (data source, metric name, value)
    - Implement citation attachment to recommendations
    - Implement drill-down capability for detailed data access
    - Store citations in DynamoDB or RDS for retrieval
    - _Requirements: 3.9.1, 3.9.2, 3.9.3, 3.9.4_
  
  - [ ] 18.2 Implement ReasoningLogger in Bedrock Agent
    - Configure Bedrock Agent to capture reasoning logs at each decision point
    - Implement structured logging of agent reasoning steps to CloudWatch
    - Implement log formatting for human readability
    - Implement log storage and retrieval via API
    - Implement uncertainty flagging when data is incomplete
    - Display reasoning logs in web UI
    - _Requirements: 3.9.5, 3.10.5_
  
  - [ ]* 18.3 Write unit tests for CitationManager Lambda
    - Test Lambda handler with mock events
    - Test citation tracking and aggregation
    - Test citation formatting
    - Test drill-down data retrieval
    - _Requirements: 3.9.1, 3.9.2, 3.9.4_
  
  - [ ]* 18.4 Write tests for ReasoningLogger
    - Test log capture and formatting
    - Test log retrieval via API
    - Test uncertainty flagging
    - _Requirements: 3.9.5, 3.10.5_

- [ ] 19. Implement ethical AI and bias mitigation
  - [ ] 19.1 Create BiasDetector Lambda function
    - Implement Lambda handler for bias detection
    - Implement demographic pattern analysis in recommendations
    - Implement socioeconomic diversity metrics calculation
    - Implement automated bias detection algorithms
    - Implement flagging of recommendations that may reinforce historical inequities
    - Store bias metrics in DynamoDB or RDS
    - _Requirements: 3.10.1, 3.10.2_
  
  - [ ] 19.2 Create AuditLogger Lambda function
    - Implement Lambda handler for audit logging
    - Implement comprehensive audit trail logging to CloudWatch and S3
    - Implement demographic distribution tracking across recommendations
    - Implement periodic bias report generation
    - Implement export functionality for external audits (CSV, JSON)
    - _Requirements: 3.10.2_
  
  - [ ] 19.3 Implement data privacy and encryption
    - Enable S3 bucket encryption for Knowledge Base and data lake
    - Enable RDS encryption at rest
    - Configure TLS/SSL for all data in transit
    - Implement data isolation between brand clients using IAM policies and RDS schemas
    - Implement PII detection and removal in Lambda functions
    - Configure GDPR-compliant data retention policies
    - Use AWS Secrets Manager for all API credentials
    - _Requirements: 3.10.3, 3.10.4_
  
  - [ ]* 19.4 Write unit tests for BiasDetector Lambda
    - Test Lambda handler with mock events
    - Test demographic pattern detection
    - Test bias flagging with synthetic biased data
    - Test diversity metrics calculation
    - _Requirements: 3.10.1, 3.10.2_
  
  - [ ]* 19.5 Write unit tests for AuditLogger Lambda
    - Test Lambda handler with mock events
    - Test audit trail completeness
    - Test report generation
    - Test data export functionality
    - _Requirements: 3.10.2_

- [ ] 20. Checkpoint - Ensure ethical AI and explainability tests pass
  - Test bias detection with diverse datasets
  - Verify audit logging completeness
  - Test data encryption and isolation
  - Verify reasoning logs and citations
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 21. Deploy infrastructure with IaC (Infrastructure as Code)
  - [ ] 21.1 Create CloudFormation or Terraform templates
    - Define all AWS resources (RDS, Lambda, S3, Bedrock, API Gateway, etc.)
    - Configure IAM roles and policies
    - Set up VPC, security groups, and networking
    - Configure CloudWatch alarms and dashboards
    - Implement blue-green deployment strategy
    - _Requirements: 3.8.1, 3.8.2_
  
  - [ ] 21.2 Set up CI/CD pipeline
    - Configure AWS CodePipeline or GitHub Actions
    - Implement automated testing in pipeline
    - Implement automated deployment to staging and production
    - Configure rollback mechanisms
    - _Requirements: 3.8.2_
  
  - [ ]* 21.3 Write infrastructure tests
    - Test IaC templates for syntax and best practices
    - Test deployment to staging environment
    - Test rollback procedures
    - _Requirements: 3.8.1_

- [ ] 22. Final checkpoint - Run full test suite
  - Run all Lambda function unit tests locally using AWS SAM
  - Run all property tests and integration tests
  - Verify all correctness properties pass with 100+ iterations
  - Test end-to-end workflow in staging environment
  - Test Bedrock Agent with diverse queries
  - Ensure response time requirements are met (<30 seconds)
  - Run bias detection tests to verify ethical AI guardrails
  - Verify reasoning logs are generated for all recommendations
  - Test real-time data integration (transit, events)
  - Test autonomous parameter discovery and reflection
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 23. Create example queries and documentation
  - [ ] 23.1 Create example query scripts
    - Write example queries demonstrating various use cases
    - Create scripts for testing with real data
    - Document expected outputs for each example
    - Create demo video showing autonomous discovery and real-time adaptation
    - _Requirements: 3.1.1, 3.1.4_
  
  - [ ] 23.2 Write developer documentation
    - Document AWS architecture and service interactions
    - Document Bedrock Agent configuration and prompting strategies
    - Document Lambda function APIs and schemas
    - Document PostGIS database schema and queries
    - Document API Gateway usage and authentication
    - Document ethical AI guardrails and bias mitigation strategies
    - Document reasoning log format and interpretation
    - Document deployment and configuration with IaC
    - Document monitoring and troubleshooting with CloudWatch and X-Ray
    - Document cost optimization strategies
    - _Requirements: 3.8.3, 3.9.5, 3.10.1_
  
  - [ ] 23.3 Create operational runbooks
    - Document incident response procedures
    - Document scaling procedures
    - Document backup and disaster recovery
    - Document security incident response
    - _Requirements: 3.8.2_

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation at key milestones
- Property tests validate universal correctness properties with 100+ iterations
- Unit tests validate specific examples and edge cases
- The implementation follows a bottom-up approach: PostGIS database → Lambda tools → Bedrock Agent → integration
- All spatial operations use H3 hexagonal indexing for consistency and performance
- The Amazon Bedrock Agent uses the ReAct pattern (Reasoning + Acting + Reflection) for query decomposition, tool coordination, and self-correction
- All Lambda functions are tested locally using AWS SAM before deployment
- Infrastructure is deployed using IaC (CloudFormation or Terraform) for reproducibility
- Development follows AI-assisted methodology with Kiro (AI IDE) for spec-driven development and vertical slicing
