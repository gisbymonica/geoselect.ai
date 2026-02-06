# Smart Site Selection Geo-Agent

> An autonomous AI consultant that transforms retail site selection using Amazon Bedrock Agents, real-time geospatial intelligence, and adaptive reasoning.

[![AWS](https://img.shields.io/badge/AWS-Bedrock%20%7C%20Lambda%20%7C%20RDS-orange)](https://aws.amazon.com/)
[![Claude](https://img.shields.io/badge/Claude-3.5%20Sonnet-blue)](https://www.anthropic.com/claude)
[![PostGIS](https://img.shields.io/badge/PostGIS-Spatial%20Database-green)](https://postgis.net/)
[![H3](https://img.shields.io/badge/H3-Hexagonal%20Indexing-red)](https://h3geo.org/)

## 🎯 Overview

The Smart Site Selection Geo-Agent is an **AWS-native autonomous AI consultant** that revolutionizes how retail brands identify and evaluate physical store locations. Unlike traditional GIS tools that provide raw data or require manual filtering, this agent:

- **Autonomously discovers** ideal parameters without user-specified filters
- **Adapts in real-time** using live transit and event feeds
- **Self-corrects** through reflection loops when initial results are unsuitable
- **Explains decisions** with transparent reasoning logs and data citations

### Why AI is Needed

**Nonlinear Complexity**: Modern retail site selection involves fragmented data (demographics, transit, POIs) that rule-based systems cannot correlate in real-time.

**Adaptive Intelligence**: The agent learns from dynamic signals—like new transit stops or competitor openings—and adjusts recommendations without manual reprogramming.

**Reducing Cognitive Load**: Automates spatial analysis, allowing teams to focus on strategic decisions rather than data wrangling.

## ✨ Key Features

### 🤖 Autonomous Discovery
Unlike competitors that require users to know what to filter, our agent discovers the ideal parameters itself through intelligent reasoning and exploration.

### ⚡ Real-Time Adaptability
Integrated with live transit and event feeds to value sites based on today's reality, not last year's census:
- Current transit schedules and delays
- Upcoming events and festivals
- Recent competitor openings
- Real-time foot traffic patterns

### 🔄 Self-Correcting Intelligence
The reflection loop enables the agent to review its own outputs and autonomously adjust search parameters if initial results are unsuitable.

### 📚 Regulatory Awareness
Knowledge Base integration ensures recommendations comply with local zoning and development regulations.

### 🔍 Full Transparency
- **Reasoning Logs**: Complete trace of decision-making process
- **Data Citations**: Every claim backed by specific data sources
- **Explainable AI**: Drill down into the data behind each recommendation

### 🛡️ Ethical AI & Bias Mitigation
- Automated bias detection in recommendations
- Fairness metrics across socioeconomic zones
- Comprehensive audit trails for human oversight

## 🏗️ Architecture

### AWS-Native Agentic Stack

```
┌─────────────────────────────────────────────────────────────┐
│                    Web UI (React + Mapbox)                   │
│              (Natural Language Query Input)                  │
└───────────────────────────┬─────────────────────────────────┘
                            │
                    Amazon API Gateway
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
```

### Core Components

- **Orchestration**: Amazon Bedrock Agents with Claude 3.5 Sonnet
- **Knowledge Base**: Amazon Bedrock Knowledge Bases (S3-backed) for city development plans and regulations
- **Action Groups**: AWS Lambda functions for specialized geospatial tasks
- **Spatial Database**: Amazon RDS with PostGIS extension
- **Spatial Indexing**: Uber H3 for hexagonal data aggregation
- **Caching**: Amazon ElastiCache (Redis) for performance
- **Monitoring**: Amazon CloudWatch + AWS X-Ray for observability

## 🚀 Quick Start

### Prerequisites

- AWS Account with Bedrock access
- AWS CLI configured
- Node.js 18+ (for web UI)
- Python 3.11+ (for Lambda functions)
- Terraform or AWS SAM (for infrastructure deployment)

### Installation

1. **Clone the repository**
```bash
git clone https://github.com/your-org/smart-site-selection.git
cd smart-site-selection
```

2. **Deploy infrastructure**
```bash
# Using Terraform
cd infrastructure/terraform
terraform init
terraform apply

# Or using AWS SAM
cd infrastructure/sam
sam build
sam deploy --guided
```

3. **Load initial data**
```bash
# Run ETL pipeline to load census data, POIs, etc.
python scripts/etl/load_census_data.py
python scripts/etl/load_poi_data.py
```

4. **Configure Bedrock Agent**
```bash
# Create and configure Bedrock Agent with action groups
python scripts/setup_bedrock_agent.py
```

5. **Deploy web UI**
```bash
cd web-ui
npm install
npm run build
aws s3 sync build/ s3://your-ui-bucket/
```

### Usage

**Natural Language Query**:
```
"Find a high-traffic spot near trendy cafes for a Gen Z clothing brand in North London"
```

**Agent Response**:
```
I've autonomously discovered that 123 High Street is optimal:

✅ Strengths:
- Within 5-minute walk of 8 trendy cafes
- H3 cell #8912 with 45% Gen Z density (highest in region)
- 2,000 daily foot traffic (verified via today's live feed)
- New metro line opening next month

⚠️ Weaknesses:
- Higher rent (£4,500/month vs. £3,800 average)
- Limited parking (street parking only)

🎯 Opportunities:
- Growing neighborhood with 15% YoY population increase
- Upcoming street festival in Q3 2026

⚡ Threats:
- Competitor (Zara) opening 400m away in Q2 2026

📍 Location: 1.2km from your nearest store (no cannibalization risk)
📐 Space: 1,200 sq ft (ideal for your brand format)
✅ Zoning: Compliant with local retail regulations

Recommendation: GO (Confidence: 87%)
```

## 📊 Use Cases

### Retail Site Selection
- Clothing stores targeting specific demographics
- Restaurant chains seeking high foot traffic areas
- Specialty retail (gyms, cafes, bookstores)

### Real Estate Analysis
- Commercial property valuation
- Market opportunity assessment
- Competitive landscape analysis

### Urban Planning
- Retail development zone identification
- Transit-oriented development planning
- Demographic trend analysis

## 🛠️ Development

### AI-Assisted Development Workflow

This project was developed using a **"Human-in-the-loop" AI development sprint** methodology:

- **Spec-Driven Development**: Using Kiro (AI IDE) to turn initial prompts into detailed technical specifications
- **Vertical Slicing**: Each feature developed as an isolated slice—from database schema to agent tool
- **Incremental Validation**: Individual tools tested in isolation using Amazon Bedrock Agents before integration

### Project Structure

```
smart-site-selection/
├── infrastructure/          # IaC (Terraform/CloudFormation)
│   ├── terraform/
│   └── sam/
├── lambda-functions/        # AWS Lambda action groups
│   ├── demographic-tool/
│   ├── poi-tool/
│   ├── search-tool/
│   ├── spatial-analysis-tool/
│   └── swot-analyzer/
├── scripts/                 # ETL and setup scripts
│   ├── etl/
│   └── setup/
├── web-ui/                  # React web interface
├── tests/                   # Unit and integration tests
├── docs/                    # Documentation
│   ├── requirements.md
│   ├── design.md
│   └── tasks.md
└── README.md
```

### Running Tests

```bash
# Unit tests for Lambda functions
cd lambda-functions/demographic-tool
pytest tests/

# Property-based tests
pytest tests/property_tests/

# Integration tests with Bedrock Agent
pytest tests/integration/

# Load tests
locust -f tests/load/locustfile.py
```

### Local Development

```bash
# Start local PostGIS database
docker-compose up -d postgres

# Run Lambda functions locally with AWS SAM
sam local start-api

# Run web UI locally
cd web-ui
npm start
```

## 📈 Performance

- **Response Time**: <30 seconds for complete analysis
- **Query Success Rate**: >90% of natural language queries successfully parsed
- **Recommendation Accuracy**: >70% of "Go" recommendations lead to actual site selection
- **Scalability**: Handles multiple concurrent queries across any region

## 🔒 Security & Privacy

- **Data Encryption**: S3 and RDS encryption at rest, TLS in transit
- **Data Isolation**: IAM policies and RDS schemas separate brand clients
- **GDPR Compliance**: Anonymized data handling and retention policies
- **API Security**: API Gateway authentication, rate limiting, AWS WAF
- **Least Privilege**: IAM roles with minimal required permissions

## 🎓 Documentation

- [Requirements](docs/requirements.md) - Detailed functional and non-functional requirements
- [Design](docs/design.md) - System architecture and design decisions
- [Tasks](docs/tasks.md) - Implementation plan and task breakdown
- [API Documentation](docs/api.md) - REST API reference
- [Deployment Guide](docs/deployment.md) - Infrastructure deployment instructions
- [Monitoring Guide](docs/monitoring.md) - CloudWatch dashboards and alerts

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md) for details.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- **Amazon Bedrock** for providing the agentic AI platform
- **Anthropic Claude 3.5 Sonnet** for advanced reasoning capabilities
- **Uber H3** for hexagonal geospatial indexing
- **PostGIS** for spatial database capabilities
- **Overture Maps** and **OpenStreetMap** for geospatial data
- **Kiro AI IDE** for AI-assisted development workflow

## 📞 Support

- **Documentation**: [docs.geoselect.ai](https://docs.geoselect.ai)
- **Issues**: [GitHub Issues](https://github.com/your-org/smart-site-selection/issues)
- **Email**: support@geoselect.ai
- **Slack**: [Join our community](https://geoselect-community.slack.com)

## 🗺️ Roadmap

### Phase 1: North London (Current)
- ✅ Core agent functionality
- ✅ UK Census 2021 data integration
- ✅ Real-time transit and event feeds
- ✅ Bias detection and ethical AI

### Phase 2: Greater London (Q2 2026)
- [ ] Expanded geographic coverage
- [ ] Multi-brand portfolio optimization
- [ ] Temporal analysis and trend prediction

### Phase 3: All of UK (Q3 2026)
- [ ] Nationwide coverage
- [ ] Competitive intelligence tracking
- [ ] Financial modeling integration

### Phase 4: International (Q4 2026)
- [ ] Multi-country support
- [ ] Localized data sources per region
- [ ] Multi-language support

---

**Built with ❤️ using AWS Bedrock Agents, Claude 3.5 Sonnet, and AI-assisted development**
