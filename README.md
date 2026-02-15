# SahaayaAI - AI-Powered Civic Assistant

SahaayaAI is a local-language, low-bandwidth, AI-powered conversational assistant that helps citizens—especially rural and underserved communities—discover, understand, and apply for government schemes, education programs, healthcare services, and job opportunities.

## 🎯 Mission

To bridge the information gap between citizens and public services by providing an accessible, voice-first, multi-language platform that works on low-end devices with minimal internet connectivity.

## 🌟 Key Features

- **Conversational Interface**: Natural language chat in Tamil, Hindi, Telugu, and English
- **Voice-First Design**: Speak your questions, hear the answers
- **Smart Eligibility Matching**: Get personalized scheme recommendations based on your age, income, location, occupation, and gender
- **Step-by-Step Guidance**: Detailed application instructions with document checklists
- **Low-Bandwidth Optimized**: Works on 2G/3G networks with minimal data usage
- **Privacy-Focused**: Your personal information is never stored permanently

## 🏗️ Architecture

```
User Interface (Web/Mobile)
         ↓
    API Gateway
         ↓
Conversational Assistant Core
    ├── Session Management
    ├── NLP Service (Intent & Entity Extraction)
    ├── Eligibility Engine (Multi-criteria Filtering)
    └── Recommendation Engine (Relevance Ranking)
         ↓
External Services (LLM, Speech-to-Text, Text-to-Speech)
         ↓
Data Layer (Knowledge Base, Session Store, Cache)
```

## 📋 Project Status

This project is currently in the **specification and design phase**. We have completed:

- ✅ Requirements document with 10 comprehensive requirements
- ✅ Design document with detailed architecture and 32 correctness properties
- ✅ Implementation plan with 18 tasks broken down into manageable steps

## 📚 Documentation

- **[Requirements Document](.kiro/specs/sahaayaai-conversational-assistant/requirements.md)**: Detailed functional and non-functional requirements using EARS patterns
- **[Design Document](.kiro/specs/sahaayaai-conversational-assistant/design.md)**: Complete system architecture, component interfaces, data models, and correctness properties
- **[Implementation Tasks](.kiro/specs/sahaayaai-conversational-assistant/tasks.md)**: Step-by-step implementation plan with testing strategy

## 🚀 Getting Started

### Prerequisites

- Node.js 18+ or Python 3.10+
- TypeScript 5+ (for TypeScript implementation)
- Redis (for session storage and caching)
- Access to LLM API (Kiro platform)
- Speech-to-Text and Text-to-Speech service credentials

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/sahaayaai.git
cd sahaayaai

# Install dependencies (TypeScript)
npm install

# Or for Python
pip install -r requirements.txt

# Set up environment variables
cp .env.example .env
# Edit .env with your API keys and configuration
```

### Running Tests

```bash
# Run unit tests
npm test

# Run property-based tests
npm run test:property

# Run all tests with coverage
npm run test:coverage
```

## 🧪 Testing Strategy

SahaayaAI uses a dual testing approach:

- **Property-Based Testing**: 32 correctness properties tested with 100+ random inputs each using fast-check (TypeScript) or hypothesis (Python)
- **Unit Testing**: Specific examples, edge cases, and error conditions

This ensures both general correctness across all inputs and specific behavior validation.

## 🛠️ Technology Stack

- **Backend**: TypeScript with Express/Fastify or Python with FastAPI
- **AI/ML**: Kiro platform for LLM integration
- **Voice**: Speech-to-Text and Text-to-Speech APIs
- **Database**: PostgreSQL for knowledge base, Redis for sessions and caching
- **Testing**: Jest + fast-check (TypeScript) or pytest + hypothesis (Python)
- **Deployment**: Docker, Kubernetes (cloud-agnostic)

## 🌍 Supported Languages

- Tamil (தமிழ்)
- Hindi (हिन्दी)
- Telugu (తెలుగు)
- English

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

### Development Workflow

1. Check the [Implementation Tasks](.kiro/specs/sahaayaai-conversational-assistant/tasks.md)
2. Pick a task and create a branch
3. Implement the task following the design specifications
4. Write tests (both unit and property-based)
5. Submit a pull request

## 📊 Correctness Properties

SahaayaAI is built with formal correctness properties that ensure reliable behavior:

- Multi-criteria eligibility filtering
- Session isolation and expiration
- Language consistency across responses
- Response time performance (<3 seconds)
- Data privacy and encryption
- Knowledge base accuracy

See the [Design Document](.kiro/specs/sahaayaai-conversational-assistant/design.md#correctness-properties) for all 32 properties.

## 🔒 Privacy & Security

- All data transmission is encrypted (HTTPS/TLS)
- Personal information is never persisted beyond session duration (15 minutes)
- Users can explicitly clear their data at any time
- Compliant with data protection regulations

## 📝 License

[MIT License](LICENSE)

## 👥 Team

SahaayaAI is built with the goal of making public services accessible to everyone, regardless of language, literacy, or connectivity.

## 🙏 Acknowledgments

- Government scheme data sources
- Open-source community
- Contributors and testers

## 📞 Contact

For questions or support, please open an issue on GitHub or contact [your-email@example.com]

---

**Note**: This project is under active development. The current phase focuses on building the core conversational assistant. Future phases will include mobile apps, offline capabilities, and expanded language support.
