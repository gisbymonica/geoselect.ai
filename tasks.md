# Implementation Plan: Smart Site Selection Geo-Agent

## Overview

This implementation plan breaks down the Smart Site Selection Geo-Agent into discrete coding tasks. The system uses an agentic workflow pattern where an LLM orchestrator coordinates specialized tools (DemographicTool, POITool, RealEstateTool, GeometryTool) to analyze potential retail locations using H3 hexagonal spatial indexing.

The implementation follows a bottom-up approach: first building the spatial foundation (H3 indexing), then individual tools, then the orchestrator, and finally the integration layer.

## Tasks

- [ ] 1. Set up project structure and core dependencies
  - Create Python project structure with appropriate directories (src/, tests/, data/, config/)
  - Set up virtual environment and install core dependencies: h3-py, geopandas, shapely, requests, fastapi, pytest, hypothesis
  - Configure PostGIS database connection for spatial data storage
  - Set up logging and configuration management
  - Create base classes and interfaces for tools
  - _Requirements: 3.8.3, 3.6.3_

- [ ] 2. Implement H3 spatial indexing foundation
  - [ ] 2.1 Create H3IndexManager class
    - Implement methods for converting lat/lon to H3 cells at multiple resolutions (8, 9, 10)
    - Implement methods for H3 cell neighbor lookups
    - Implement methods for H3 cell to polygon conversion
    - Implement hierarchical aggregation across H3 resolutions
    - _Requirements: 3.2.5, 3.4.5_
  
  - [ ]* 2.2 Write property test for H3 cell containment
    - **Property 6.1.2: H3 Cell Containment**
    - For any point P and H3 cell C, if P is reported as being in C, then P must be within the geometric boundary of C
    - **Validates: Requirements 3.2.5, 3.7.1**
  
  - [ ]* 2.3 Write unit tests for H3IndexManager
    - Test resolution conversion edge cases
    - Test boundary conditions for cell lookups
    - Test hierarchical aggregation correctness
    - _Requirements: 3.2.5_

- [ ] 3. Implement GeometryTool for spatial operations
  - [ ] 3.1 Create GeometryTool class with distance calculations
    - Implement geodesic distance calculation between two points
    - Implement buffering operations for proximity analysis
    - Implement spatial intersection and union operations
    - _Requirements: 3.7.3, 3.4.4_
  
  - [ ] 3.2 Implement isochrone calculation
    - Integrate with OSRM routing engine for walk-time isochrones (5 and 10 minutes)
    - Integrate with OSRM routing engine for drive-time isochrones (5 and 10 minutes)
    - Implement isochrone polygon generation from routing results
    - Handle edge cases (unreachable areas, routing failures)
    - _Requirements: 3.4.2, 3.4.3, 3.7.4_
  
  - [ ]* 3.3 Write property test for distance calculation accuracy
    - **Property 6.1.1: Distance Calculation Accuracy**
    - For any two points A and B, the calculated distance must be within 50 meters of the true geodesic distance
    - **Validates: Requirements 3.7.3**
  
  - [ ]* 3.4 Write property test for isochrone consistency
    - **Property 6.1.3: Isochrone Consistency**
    - For any location L and travel time T, all points within the T-minute isochrone must be reachable within T minutes via the road network
    - **Validates: Requirements 3.4.2, 3.4.3, 3.7.4**
  
  - [ ]* 3.5 Write unit tests for GeometryTool
    - Test edge cases for distance calculations (antipodal points, same location)
    - Test isochrone generation with mock routing responses
    - Test spatial operations (buffer, intersection, union)
    - _Requirements: 3.4.2, 3.4.3, 3.7.3_

- [ ] 4. Checkpoint - Ensure spatial foundation tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 5. Implement DemographicTool for census data
  - [ ] 5.1 Create DemographicTool class
    - Implement data loading from UK Census 2021 Output Area data
    - Implement H3 cell aggregation for demographic metrics (age distribution, income levels)
    - Implement query methods for retrieving demographics by H3 cell or geographic area
    - Implement caching layer for frequently accessed cells
    - _Requirements: 3.2.1, 3.4.4, 3.6.3_
  
  - [ ]* 5.2 Write property test for spatial join completeness
    - **Property 6.2.1: Spatial Join Completeness**
    - When overlaying demographic data onto H3 cells, the sum of all cell populations must equal the total population of the source data
    - **Validates: Requirements 3.2.4, 3.4.4**
  
  - [ ]* 5.3 Write unit tests for DemographicTool
    - Test demographic aggregation with synthetic data
    - Test caching behavior
    - Test query filtering by age range, income level
    - _Requirements: 3.2.1, 3.6.3_

- [ ] 6. Implement POITool for Points of Interest
  - [ ] 6.1 Create POITool class
    - Implement integration with Overture Maps Places dataset
    - Implement OpenStreetMap Overpass API integration for real-time POI data
    - Implement category-based filtering (cafes, gyms, retail, competitors)
    - Implement proximity search within specified radius
    - Implement H3 cell-based POI aggregation
    - _Requirements: 3.2.2, 3.7.1_
  
  - [ ]* 6.2 Write property test for POI proximity accuracy
    - **Property 6.2.2: POI Proximity Accuracy**
    - For any candidate site S and POI category C, if the system reports N POIs of category C within distance D, then there must be exactly N POIs of category C within distance D of S
    - **Validates: Requirements 3.2.2**
  
  - [ ]* 6.3 Write unit tests for POITool
    - Test category filtering with mock POI data
    - Test proximity search edge cases (zero results, large result sets)
    - Test H3 aggregation correctness
    - _Requirements: 3.2.2_

- [ ] 7. Implement RealEstateTool for property listings
  - [ ] 7.1 Create RealEstateTool class
    - Implement PropertyData UK API integration (or mock API for testing)
    - Implement filtering by size, type, price, availability status
    - Implement geographic filtering by bounding box or H3 cells
    - Implement property detail extraction (address, size, price)
    - Handle API rate limiting and error responses
    - _Requirements: 3.3.1, 3.3.2, 3.3.3_
  
  - [ ]* 7.2 Write unit tests for RealEstateTool
    - Test filtering logic with mock property data
    - Test API error handling (timeout, rate limit, invalid response)
    - Test geographic filtering accuracy
    - _Requirements: 3.3.1, 3.3.2, 3.3.3_

- [ ] 8. Checkpoint - Ensure all tool tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 9. Implement spatial synthesis and hotspot identification
  - [ ] 9.1 Create SpatialSynthesizer class
    - Implement multi-layer data overlay onto H3 grid
    - Implement metric aggregation within H3 cells (demographics, POI counts, foot traffic)
    - Implement hotspot scoring algorithm based on query requirements
    - Implement ranking and filtering of top-scoring H3 cells
    - _Requirements: 3.2.4, 3.4.4_
  
  - [ ]* 9.2 Write unit tests for SpatialSynthesizer
    - Test multi-layer overlay with synthetic data
    - Test scoring algorithm with known inputs and expected outputs
    - Test ranking correctness
    - _Requirements: 3.2.4_

- [ ] 10. Implement cannibalization detection
  - [ ] 10.1 Create CannibalizationChecker class
    - Implement distance-based cannibalization detection
    - Implement isochrone-based catchment area overlap calculation
    - Implement configurable distance threshold
    - Implement flagging logic for at-risk sites
    - _Requirements: 3.4.1_
  
  - [ ]* 10.2 Write property test for cannibalization detection
    - **Property 6.3.1: Cannibalization Detection**
    - For any candidate site C and existing store E, if the distance between C and E is less than the cannibalization threshold T, then C must be flagged as a cannibalization risk
    - **Validates: Requirements 3.4.1**
  
  - [ ]* 10.3 Write unit tests for CannibalizationChecker
    - Test edge cases (exactly at threshold, just below/above threshold)
    - Test with multiple existing stores
    - Test isochrone overlap calculation
    - _Requirements: 3.4.1_

- [ ] 11. Implement LLM orchestrator with ReAct loop
  - [ ] 11.1 Create Orchestrator class
    - Implement LLM integration (Gemini 1.5 Pro or GPT-4)
    - Implement ReAct loop (Reasoning + Acting pattern)
    - Implement query parsing and requirement extraction
    - Implement tool selection logic based on query requirements
    - Implement parallel tool execution coordination
    - Handle ambiguous queries by generating clarifying questions
    - _Requirements: 3.1.1, 3.1.2, 3.1.3, 3.1.4_
  
  - [ ] 11.2 Create tool registry and function calling interface
    - Define tool schemas for LLM function calling
    - Implement tool registration and discovery
    - Implement tool execution wrapper with error handling
    - Implement result aggregation from multiple tools
    - _Requirements: 3.1.2_
  
  - [ ]* 11.3 Write unit tests for Orchestrator
    - Test query parsing with various natural language inputs
    - Test tool selection logic with mock LLM responses
    - Test error handling for tool failures
    - Test clarifying question generation for ambiguous queries
    - _Requirements: 3.1.1, 3.1.2, 3.1.3_

- [ ] 12. Implement SWOT analysis generation
  - [ ] 12.1 Create SWOTAnalyzer class
    - Implement SWOT structure generation (Strengths, Weaknesses, Opportunities, Threats)
    - Implement LLM-based SWOT content generation using synthesized data
    - Implement Go/No-Go recommendation logic with confidence scoring
    - Implement data citation extraction and formatting
    - _Requirements: 3.5.1, 3.5.2, 3.5.3, 3.5.4, 3.9.1, 3.9.2_
  
  - [ ]* 12.2 Write property test for SWOT completeness
    - **Property 6.3.2: SWOT Completeness**
    - For any site recommendation, the SWOT analysis must include at least one item in each category (Strength, Weakness, Opportunity, Threat)
    - **Validates: Requirements 3.5.2**
  
  - [ ]* 12.3 Write property test for citation traceability
    - **Property 6.3.3: Citation Traceability**
    - For any recommendation R, every claim in R must have a corresponding citation that includes: data source, metric name, and specific value
    - **Validates: Requirements 3.9.1, 3.9.2**
  
  - [ ]* 12.4 Write unit tests for SWOTAnalyzer
    - Test SWOT structure validation
    - Test citation formatting
    - Test Go/No-Go logic with various confidence thresholds
    - _Requirements: 3.5.1, 3.5.2, 3.5.3, 3.5.4_

- [ ] 13. Checkpoint - Ensure orchestration and analysis tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 14. Implement end-to-end workflow integration
  - [ ] 14.1 Create SiteSelectionAgent class (main entry point)
    - Implement complete workflow orchestration (Steps 1-9 from design)
    - Wire together: Orchestrator → Tools → SpatialSynthesizer → CannibalizationChecker → SWOTAnalyzer
    - Implement result formatting and output generation
    - Implement top-3 site selection logic
    - _Requirements: 3.1.1, 3.2.4, 3.4.1, 3.5.1_
  
  - [ ]* 14.2 Write integration tests for end-to-end workflow
    - Test complete workflow with known query and expected results
    - Test with various query types (demographic-focused, POI-focused, mixed)
    - Test error propagation and recovery
    - _Requirements: 3.1.1, 3.5.1_

- [ ] 15. Implement performance optimization and caching
  - [ ] 15.1 Add caching layer
    - Implement Redis-based caching for H3 cell demographics
    - Implement query result caching with TTL
    - Implement POI data caching
    - _Requirements: 3.6.3_
  
  - [ ] 15.2 Add performance monitoring
    - Implement timing instrumentation for each tool
    - Implement overall query response time tracking
    - Add logging for performance metrics
    - _Requirements: 3.6.1_
  
  - [ ]* 15.3 Write property test for response time bound
    - **Property 6.4.1: Response Time Bound**
    - For any valid query Q, the system must return a complete recommendation within 30 seconds
    - **Validates: Requirements 3.6.1**

- [ ] 16. Implement REST API with FastAPI
  - [ ] 16.1 Create API endpoints
    - Implement POST /query endpoint for natural language queries
    - Implement GET /sites/{site_id} endpoint for detailed site information
    - Implement GET /health endpoint for health checks
    - Implement request validation and error handling
    - Implement API documentation with OpenAPI/Swagger
    - _Requirements: 3.1.1, 3.8.2_
  
  - [ ] 16.2 Add authentication and rate limiting
    - Implement API key authentication
    - Implement rate limiting to prevent abuse
    - Implement request logging
    - _Requirements: Security & Privacy section_
  
  - [ ]* 16.3 Write API integration tests
    - Test all endpoints with valid and invalid inputs
    - Test authentication and authorization
    - Test rate limiting behavior
    - Test concurrent request handling
    - _Requirements: 3.8.2_

- [ ] 17. Implement data loading and ETL pipeline
  - [ ] 17.1 Create data loaders for initial datasets
    - Implement UK Census 2021 data loader with H3 indexing
    - Implement Overture Maps Places data loader
    - Implement OpenStreetMap data extraction scripts
    - Implement data validation and quality checks
    - _Requirements: 3.2.1, 3.2.2, 3.7.1, 3.7.2_
  
  - [ ] 17.2 Create database schema and migrations
    - Design PostGIS schema for spatial data storage
    - Create tables for demographics, POIs, properties, H3 cells
    - Implement spatial indexes for performance
    - Create database migration scripts
    - _Requirements: 3.2.5, 3.6.3_
  
  - [ ]* 17.3 Write data validation tests
    - Test data loader correctness with sample datasets
    - Test H3 indexing during data load
    - Test data quality checks
    - _Requirements: 3.7.1, 3.7.2_

- [ ] 18. Implement explainability and citation system
  - [ ] 18.1 Create CitationManager class
    - Implement citation tracking throughout the workflow
    - Implement citation formatting (data source, metric name, value)
    - Implement citation attachment to recommendations
    - Implement drill-down capability for detailed data access
    - _Requirements: 3.9.1, 3.9.2, 3.9.3, 3.9.4_
  
  - [ ]* 18.2 Write unit tests for CitationManager
    - Test citation tracking and aggregation
    - Test citation formatting
    - Test drill-down data retrieval
    - _Requirements: 3.9.1, 3.9.2, 3.9.4_

- [ ] 19. Final checkpoint - Run full test suite
  - Run all unit tests, property tests, and integration tests
  - Verify all correctness properties pass with 100+ iterations
  - Ensure response time requirements are met
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 20. Create example queries and documentation
  - [ ] 20.1 Create example query scripts
    - Write example queries demonstrating various use cases
    - Create scripts for testing with real data
    - Document expected outputs for each example
    - _Requirements: 3.1.1, 3.1.4_
  
  - [ ] 20.2 Write developer documentation
    - Document API usage and authentication
    - Document tool extension guide for adding new data sources
    - Document H3 resolution selection guidelines
    - Document deployment and configuration
    - _Requirements: 3.8.3_

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation at key milestones
- Property tests validate universal correctness properties with 100+ iterations
- Unit tests validate specific examples and edge cases
- The implementation follows a bottom-up approach: spatial foundation → tools → orchestrator → integration
- All spatial operations use H3 hexagonal indexing for consistency and performance
- The LLM orchestrator uses the ReAct pattern (Reasoning + Acting) for query decomposition and tool coordination
