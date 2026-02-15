# Design Document: SahaayaAI Conversational Assistant

## Overview

The SahaayaAI Conversational Assistant is an AI-powered system that enables citizens to discover and access public services through natural language conversations. The system processes user queries in multiple languages, determines eligibility for various schemes based on user attributes, and provides personalized recommendations with application guidance.

The design follows a layered architecture with clear separation between the user interface, AI orchestration, eligibility determination, and knowledge management. The system prioritizes accessibility, low latency, and minimal data usage to serve users with limited connectivity and device capabilities.

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     User Interface Layer                     │
│  (Web/Mobile App with Voice Input/Output capabilities)      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                  API Gateway / Load Balancer                 │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                 Conversational Assistant Core                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │           Session Management Service                  │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         Natural Language Processing Service           │   │
│  │  - Intent Extraction                                  │   │
│  │  - Entity Recognition                                 │   │
│  │  - Language Detection                                 │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            Eligibility Engine Service                 │   │
│  │  - Rule-based Filtering                               │   │
│  │  - Multi-criteria Matching                            │   │
│  └──────────────────────────────────────────────────────┘   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │          Recommendation Engine Service                │   │
│  │  - Relevance Ranking                                  │   │
│  │  - Response Generation                                │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    External Services                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   LLM API    │  │ Speech-to-   │  │ Text-to-     │      │
│  │   (Kiro)     │  │ Text Service │  │ Speech Svc   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                      Data Layer                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  Knowledge   │  │   Session    │  │    Cache     │      │
│  │    Base      │  │    Store     │  │   (Redis)    │      │
│  │  (Schemes)   │  │  (Temp)      │  │              │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Component Interaction Flow

1. **User Query Flow:**
   - User submits query (text or voice) through UI
   - API Gateway routes request to Session Management Service
   - If voice input: Speech-to-Text converts audio to text
   - NLP Service extracts intent and entities from query
   - Session Management retrieves user context

2. **Eligibility and Recommendation Flow:**
   - Eligibility Engine filters schemes based on user attributes
   - Recommendation Engine ranks eligible schemes by relevance
   - LLM API generates natural language response
   - If voice output requested: Text-to-Speech converts response to audio
   - Response returned to user through UI

3. **Session Management Flow:**
   - Session created on first user interaction
   - User attributes stored in Session Store (temporary, in-memory)
   - Session expires after 15 minutes of inactivity
   - Sensitive data cleared on session end

## Components and Interfaces

### 1. Session Management Service

**Responsibility:** Manages user sessions, stores conversation context, and handles session lifecycle.

**Interface:**
```typescript
interface SessionManager {
  createSession(userId: string, language: string): Session
  getSession(sessionId: string): Session | null
  updateSession(sessionId: string, updates: Partial<SessionData>): void
  clearSession(sessionId: string): void
  isSessionActive(sessionId: string): boolean
}

interface Session {
  sessionId: string
  userId: string
  language: string
  userAttributes: UserAttributes
  conversationHistory: Message[]
  createdAt: Date
  lastActivityAt: Date
}

interface UserAttributes {
  age?: number
  income?: number
  location?: string
  occupation?: string
  gender?: string
}

interface Message {
  role: 'user' | 'assistant'
  content: string
  timestamp: Date
}
```

**Key Operations:**
- `createSession`: Initialize new session with user ID and preferred language
- `getSession`: Retrieve active session data
- `updateSession`: Update user attributes or add messages to conversation history
- `clearSession`: Remove session data and clear sensitive information
- `isSessionActive`: Check if session is within 15-minute activity window

### 2. Natural Language Processing Service

**Responsibility:** Processes user queries to extract intent, entities, and detect language.

**Interface:**
```typescript
interface NLPService {
  processQuery(query: string, language: string): ProcessedQuery
  detectLanguage(text: string): string
  extractEntities(query: string): Entity[]
}

interface ProcessedQuery {
  originalQuery: string
  intent: Intent
  entities: Entity[]
  language: string
  confidence: number
}

interface Intent {
  type: 'search_scheme' | 'check_eligibility' | 'get_application_info' | 'clarification' | 'unknown'
  description: string
}

interface Entity {
  type: 'age' | 'income' | 'location' | 'occupation' | 'gender' | 'scheme_name' | 'scheme_category'
  value: string | number
  confidence: number
}
```

**Key Operations:**
- `processQuery`: Analyze query text and extract structured information
- `detectLanguage`: Identify the language of the input text
- `extractEntities`: Pull out specific data points (age, location, etc.) from query

### 3. Eligibility Engine Service

**Responsibility:** Determines which schemes a user qualifies for based on their attributes.

**Interface:**
```typescript
interface EligibilityEngine {
  filterSchemes(userAttributes: UserAttributes, allSchemes: Scheme[]): Scheme[]
  checkEligibility(userAttributes: UserAttributes, scheme: Scheme): EligibilityResult
  getMissingAttributes(userAttributes: UserAttributes, scheme: Scheme): string[]
}

interface Scheme {
  schemeId: string
  name: string
  description: string
  category: string
  eligibilityCriteria: EligibilityCriteria
  benefits: string[]
  applicationProcess: ApplicationProcess
  lastUpdated: Date
  isActive: boolean
}

interface EligibilityCriteria {
  minAge?: number
  maxAge?: number
  maxIncome?: number
  allowedLocations?: string[]
  allowedOccupations?: string[]
  allowedGenders?: string[]
}

interface EligibilityResult {
  isEligible: boolean
  matchedCriteria: string[]
  unmatchedCriteria: string[]
  missingInformation: string[]
}

interface ApplicationProcess {
  steps: string[]
  requiredDocuments: string[]
  deadline?: Date
  submissionMethod: string
  contactInfo: string
}
```

**Key Operations:**
- `filterSchemes`: Return only schemes that match user's attributes
- `checkEligibility`: Detailed eligibility check for a specific scheme
- `getMissingAttributes`: Identify what information is needed to determine eligibility

**Eligibility Logic:**
- Age: User age must be within [minAge, maxAge] range if specified
- Income: User income must be ≤ maxIncome if specified
- Location: User location must be in allowedLocations list if specified
- Occupation: User occupation must be in allowedOccupations list if specified
- Gender: User gender must be in allowedGenders list if specified
- All specified criteria must be satisfied for eligibility

### 4. Recommendation Engine Service

**Responsibility:** Ranks eligible schemes by relevance and generates user-friendly responses.

**Interface:**
```typescript
interface RecommendationEngine {
  rankSchemes(schemes: Scheme[], query: ProcessedQuery, userAttributes: UserAttributes): RankedScheme[]
  generateResponse(rankedSchemes: RankedScheme[], query: ProcessedQuery, language: string): Response
  generateApplicationGuidance(scheme: Scheme, language: string): ApplicationGuidance
}

interface RankedScheme {
  scheme: Scheme
  relevanceScore: number
  matchReason: string
}

interface Response {
  text: string
  schemes: RankedScheme[]
  followUpQuestions: string[]
  requiresMoreInfo: boolean
}

interface ApplicationGuidance {
  schemeId: string
  schemeName: string
  steps: string[]
  documentChecklist: string[]
  deadline: string
  submissionMethod: string
  contactInfo: string
}
```

**Key Operations:**
- `rankSchemes`: Score and sort schemes by relevance to user query
- `generateResponse`: Create natural language response with scheme recommendations
- `generateApplicationGuidance`: Provide detailed application instructions

**Ranking Algorithm:**
- Base score: Number of matched eligibility criteria
- Query relevance: Keyword match between query and scheme description/category
- Recency: Prefer recently updated schemes
- Popularity: Consider schemes frequently accessed by similar users (optional)
- Maximum 5 schemes returned per response

### 5. Voice Services Integration

**Responsibility:** Convert between speech and text for voice interactions.

**Interface:**
```typescript
interface VoiceService {
  speechToText(audioData: Buffer, language: string): SpeechToTextResult
  textToSpeech(text: string, language: string): Buffer
}

interface SpeechToTextResult {
  text: string
  confidence: number
  language: string
}
```

**Key Operations:**
- `speechToText`: Convert audio input to text with noise filtering
- `textToSpeech`: Generate clear audio output in specified language

**Audio Processing:**
- Input: Accept common audio formats (MP3, WAV, OGG)
- Noise filtering: Apply basic noise reduction before transcription
- Compression: Use efficient codec (Opus) for audio output
- Quality: Balance clarity with file size for low-bandwidth scenarios

### 6. LLM Integration (Kiro Platform)

**Responsibility:** Generate natural, contextual responses using large language models.

**Interface:**
```typescript
interface LLMService {
  generateResponse(prompt: string, context: ConversationContext): string
  generateClarificationQuestion(query: string, missingInfo: string[]): string
}

interface ConversationContext {
  conversationHistory: Message[]
  userAttributes: UserAttributes
  availableSchemes: Scheme[]
  language: string
}
```

**Key Operations:**
- `generateResponse`: Create natural language response based on schemes and context
- `generateClarificationQuestion`: Ask for missing information in user-friendly way

**Prompt Engineering:**
- System prompt includes: role (civic assistant), tone (helpful, simple), language preference
- Context includes: recent conversation history (last 5 messages), user attributes, eligible schemes
- Output constraints: Simple vocabulary, max 200 words, structured format for schemes

## Data Models

### Knowledge Base Schema

**Scheme Document:**
```json
{
  "schemeId": "string (unique identifier)",
  "name": {
    "en": "string",
    "hi": "string",
    "ta": "string",
    "te": "string"
  },
  "description": {
    "en": "string",
    "hi": "string",
    "ta": "string",
    "te": "string"
  },
  "category": "welfare | healthcare | education | employment",
  "eligibilityCriteria": {
    "minAge": "number (optional)",
    "maxAge": "number (optional)",
    "maxIncome": "number (optional, annual income in INR)",
    "allowedLocations": ["string (state/district codes)"],
    "allowedOccupations": ["string"],
    "allowedGenders": ["male", "female", "other"]
  },
  "benefits": ["string"],
  "applicationProcess": {
    "steps": ["string"],
    "requiredDocuments": ["string"],
    "deadline": "ISO date string (optional)",
    "submissionMethod": "online | offline | both",
    "contactInfo": "string"
  },
  "lastUpdated": "ISO date string",
  "isActive": "boolean",
  "sourceUrl": "string (official government link)"
}
```

### Session Data Schema

**Session Document (Temporary Storage):**
```json
{
  "sessionId": "string (UUID)",
  "userId": "string (anonymous or authenticated)",
  "language": "en | hi | ta | te",
  "userAttributes": {
    "age": "number (optional)",
    "income": "number (optional)",
    "location": "string (optional)",
    "occupation": "string (optional)",
    "gender": "string (optional)"
  },
  "conversationHistory": [
    {
      "role": "user | assistant",
      "content": "string",
      "timestamp": "ISO date string"
    }
  ],
  "createdAt": "ISO date string",
  "lastActivityAt": "ISO date string",
  "expiresAt": "ISO date string (createdAt + 15 minutes)"
}
```

### API Request/Response Models

**Query Request:**
```json
{
  "sessionId": "string (optional for first request)",
  "query": "string (text query)",
  "audioData": "base64 string (optional, for voice input)",
  "language": "en | hi | ta | te (optional, auto-detected if not provided)",
  "requestAudio": "boolean (whether to return audio response)"
}
```

**Query Response:**
```json
{
  "sessionId": "string",
  "response": {
    "text": "string",
    "audioData": "base64 string (optional)",
    "schemes": [
      {
        "schemeId": "string",
        "name": "string",
        "description": "string",
        "relevanceScore": "number",
        "matchReason": "string"
      }
    ],
    "followUpQuestions": ["string"],
    "requiresMoreInfo": "boolean"
  },
  "processingTime": "number (milliseconds)"
}
```

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system—essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*


### Property 1: Intent Extraction for Supported Languages
*For any* query in a supported language (Tamil, Hindi, Telugu, English), the NLP service should successfully extract a valid intent without errors.
**Validates: Requirements 1.1, 1.5**

### Property 2: Clarification on Ambiguity
*For any* query where the intent confidence score is below the ambiguity threshold, the system should generate a clarification question rather than providing a direct answer.
**Validates: Requirements 1.3**

### Property 3: Error Handling for Unintelligible Queries
*For any* query that cannot be processed (empty, nonsensical, or unsupported format), the system should return an error message with suggestions for rephrasing.
**Validates: Requirements 1.4**

### Property 4: Multi-Criteria Eligibility Filtering
*For any* user attributes and any set of schemes, the filtered results should only include schemes where ALL specified eligibility criteria are satisfied (age within range, income below threshold, location in allowed list, occupation in allowed list, gender in allowed list).
**Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.5, 2.6**

### Property 5: Missing Information Detection
*For any* incomplete user attributes and any scheme with eligibility criteria, the system should correctly identify which attributes are missing for eligibility determination.
**Validates: Requirements 2.7**

### Property 6: Relevance-Based Ranking
*For any* set of eligible schemes and a query, the recommendation engine should return schemes sorted in descending order by relevance score.
**Validates: Requirements 3.1**

### Property 7: Recommendation Response Completeness
*For any* scheme recommendation, the response should include the scheme name, description, and key benefits.
**Validates: Requirements 3.2**

### Property 8: Detailed Scheme Information Completeness
*For any* scheme detail request, the response should include eligibility requirements, benefits, and application process information.
**Validates: Requirements 3.3**

### Property 9: Maximum Recommendations Limit
*For any* query, the number of scheme recommendations returned should not exceed 5.
**Validates: Requirements 3.5**

### Property 10: Speech-to-Text Conversion
*For any* valid audio input, the voice service should produce a non-empty text transcription.
**Validates: Requirements 4.1**

### Property 11: Low Confidence Audio Handling
*For any* audio input where speech-to-text confidence is below the clarity threshold, the system should request the user to repeat their query.
**Validates: Requirements 4.3**

### Property 12: Language Consistency in Responses
*For any* query in language L, all responses (text and audio) within the same session should be in language L unless explicitly changed by the user.
**Validates: Requirements 4.4, 5.2**

### Property 13: Session Language Initialization
*For any* new session, the system should either detect the language from the first query or accept an explicit language parameter.
**Validates: Requirements 5.1**

### Property 14: Scheme Terminology Consistency
*For any* scheme, the official scheme name and key terms should remain consistent across all supported language translations.
**Validates: Requirements 5.4**

### Property 15: Application Guidance Completeness
*For any* application guidance request, the response should include step-by-step procedure, required documents, submission method, and contact information.
**Validates: Requirements 6.1, 6.2, 6.5**

### Property 16: Document Checklist Generation
*For any* scheme, the system should generate a downloadable list containing all required documents from the scheme's application process.
**Validates: Requirements 6.3**

### Property 17: Location-Specific Guidance
*For any* user location and scheme with location-specific procedures, the application guidance should match the user's location.
**Validates: Requirements 6.4**

### Property 18: Session State Persistence
*For any* session, user attributes added during the session should be retrievable until the session expires or is explicitly cleared.
**Validates: Requirements 7.1**

### Property 19: Session Expiration
*For any* session inactive for more than 15 minutes, the session data should be cleared and no longer retrievable.
**Validates: Requirements 7.3**

### Property 20: Session Isolation
*For any* two distinct sessions (even with the same user ID), data from one session should not be accessible in the other session.
**Validates: Requirements 7.4**

### Property 21: Explicit Session Clearing
*For any* session, calling the clearSession operation should immediately remove all session data from storage.
**Validates: Requirements 7.5**

### Property 22: Response Time Performance
*For any* query under normal conditions (simulated with adequate resources), the system should generate a response within 3 seconds.
**Validates: Requirements 8.1**

### Property 23: Audio Compression
*For any* text-to-speech output, the audio data should be compressed (file size significantly smaller than uncompressed WAV format).
**Validates: Requirements 8.3**

### Property 24: Scheme Information Caching
*For any* scheme, when requested multiple times within a short period, subsequent requests should be served from cache (faster response time).
**Validates: Requirements 8.4**

### Property 25: Encrypted Data Transmission
*For any* request containing personal information, the data should be transmitted over an encrypted channel (HTTPS/TLS).
**Validates: Requirements 9.1**

### Property 26: Session Data Non-Persistence
*For any* expired session, personally identifiable information should not be retrievable from persistent storage.
**Validates: Requirements 9.2**

### Property 27: Data Deletion Compliance
*For any* data deletion request, after 24 hours, the user's personal information should not be retrievable from any storage system.
**Validates: Requirements 9.4**

### Property 28: Knowledge Base as Source of Truth
*For any* scheme information provided in a response, the data should match the corresponding entry in the official knowledge base.
**Validates: Requirements 10.1**

### Property 29: Knowledge Base Update Propagation
*For any* scheme updated in the knowledge base, queries made more than 24 hours after the update should return the new information.
**Validates: Requirements 10.2**

### Property 30: Active Schemes Only
*For any* query, the recommended schemes should only include schemes where isActive = true.
**Validates: Requirements 10.3**

### Property 31: Last Updated Date Inclusion
*For any* detailed scheme information response, the lastUpdated date should be included.
**Validates: Requirements 10.4**

### Property 32: Uncertainty Indication
*For any* response where the system's confidence is below the certainty threshold, the response should include a disclaimer suggesting verification with official sources.
**Validates: Requirements 10.5**

## Error Handling

### Error Categories and Handling Strategies

**1. Input Validation Errors**
- **Empty or null queries**: Return error message "Please provide a question or request"
- **Unsupported language**: Return error message "This language is not yet supported. Please try Tamil, Hindi, Telugu, or English"
- **Invalid audio format**: Return error message "Audio format not supported. Please use MP3, WAV, or OGG"
- **Audio file too large**: Return error message "Audio file is too large. Please keep recordings under 2 minutes"

**2. Processing Errors**
- **NLP service failure**: Log error, return fallback message "I'm having trouble understanding right now. Please try rephrasing your question"
- **LLM API timeout**: Retry once, if fails return "The service is taking longer than expected. Please try again in a moment"
- **Speech-to-text failure**: Return "I couldn't hear that clearly. Please try speaking again"
- **Text-to-speech failure**: Return text response only, log error for monitoring

**3. Data Errors**
- **Scheme not found**: Return "I couldn't find information about that scheme. Would you like to explore similar options?"
- **Knowledge base unavailable**: Use cached data if available, otherwise return "I'm having trouble accessing scheme information. Please try again shortly"
- **Stale cache data**: Serve cached data with disclaimer "This information was last updated on [date]. Please verify with official sources"

**4. Session Errors**
- **Session expired**: Create new session, inform user "Your session has expired. Let's start fresh. How can I help you?"
- **Session not found**: Create new session transparently
- **Session storage failure**: Continue without session state, log error

**5. Eligibility Determination Errors**
- **Insufficient information**: Request missing attributes "To find schemes for you, I need to know your [age/income/location]. Could you share that?"
- **No eligible schemes**: Suggest alternatives "I couldn't find schemes matching all your criteria. Would you like to see schemes for [broader category]?"
- **Conflicting criteria**: Ask for clarification "I found some conflicting information. Are you looking for [option A] or [option B]?"

**6. External Service Errors**
- **Voice service unavailable**: Disable voice features, continue with text-only mode
- **LLM service unavailable**: Fall back to template-based responses for common queries
- **Database connection failure**: Use read replicas or cached data, alert operations team

### Error Response Format

All errors should follow a consistent structure:

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "User-friendly error message",
    "suggestion": "Helpful suggestion for the user",
    "retryable": true/false
  },
  "sessionId": "string (if available)"
}
```

### Logging and Monitoring

- Log all errors with severity levels (ERROR, WARNING, INFO)
- Include context: sessionId, userId, query, timestamp, error details
- Monitor error rates and alert on anomalies
- Track specific error types for continuous improvement

## Testing Strategy

### Dual Testing Approach

The testing strategy employs both unit tests and property-based tests to ensure comprehensive coverage:

- **Unit tests**: Verify specific examples, edge cases, and error conditions
- **Property-based tests**: Verify universal properties across randomly generated inputs

Both approaches are complementary and necessary. Unit tests catch concrete bugs in specific scenarios, while property-based tests verify general correctness across a wide range of inputs.

### Property-Based Testing Configuration

**Framework Selection:**
- **Python**: Use `hypothesis` library for property-based testing
- **TypeScript/JavaScript**: Use `fast-check` library for property-based testing

**Test Configuration:**
- Each property test must run a minimum of 100 iterations
- Each test must include a comment tag referencing the design property
- Tag format: `# Feature: sahaayaai-conversational-assistant, Property {number}: {property_text}`

**Example Property Test Structure (Python):**
```python
from hypothesis import given, strategies as st

# Feature: sahaayaai-conversational-assistant, Property 4: Multi-Criteria Eligibility Filtering
@given(
    user_age=st.integers(min_value=0, max_value=120),
    user_income=st.integers(min_value=0, max_value=10000000),
    schemes=st.lists(st.builds(Scheme))
)
def test_eligibility_filtering_respects_all_criteria(user_age, user_income, schemes):
    user_attrs = UserAttributes(age=user_age, income=user_income)
    filtered = eligibility_engine.filter_schemes(user_attrs, schemes)
    
    for scheme in filtered:
        if scheme.eligibility_criteria.min_age:
            assert user_age >= scheme.eligibility_criteria.min_age
        if scheme.eligibility_criteria.max_age:
            assert user_age <= scheme.eligibility_criteria.max_age
        if scheme.eligibility_criteria.max_income:
            assert user_income <= scheme.eligibility_criteria.max_income
```

### Unit Testing Focus Areas

Unit tests should focus on:

1. **Specific Examples**: Test known input-output pairs
   - Example: Query "I need a scholarship" → Intent type "search_scheme", category "education"
   - Example: Age 25, income 50000 → Eligible for specific test schemes

2. **Edge Cases**: Test boundary conditions
   - Empty query strings
   - Maximum length queries
   - Age at exact min/max boundaries
   - Income at exact threshold
   - Session at exactly 15 minutes

3. **Error Conditions**: Test failure scenarios
   - Invalid audio formats
   - Malformed JSON requests
   - Database connection failures
   - External service timeouts

4. **Integration Points**: Test component interactions
   - NLP service → Eligibility engine flow
   - Session manager → Response generator flow
   - Voice service → NLP service flow

### Test Coverage Goals

- **Code coverage**: Minimum 80% line coverage
- **Property coverage**: All 32 correctness properties implemented as property tests
- **Edge case coverage**: All identified edge cases covered by unit tests
- **Error path coverage**: All error handling paths tested

### Testing Pyramid

```
        /\
       /  \
      / E2E \          (10% - Critical user flows)
     /______\
    /        \
   /Integration\       (20% - Component interactions)
  /____________\
 /              \
/   Unit + PBT   \     (70% - Individual functions and properties)
/__________________\
```

### Continuous Testing

- Run unit tests on every commit
- Run property tests on every pull request
- Run integration tests nightly
- Run performance tests weekly
- Monitor test execution time and optimize slow tests

### Test Data Management

- Use factories/builders for generating test schemes
- Maintain a seed dataset of representative schemes for consistent testing
- Use property-based testing generators for exploring edge cases
- Anonymize any real user data used in testing
