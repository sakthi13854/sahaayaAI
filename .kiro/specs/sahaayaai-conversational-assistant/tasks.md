# Implementation Plan: SahaayaAI Conversational Assistant

## Overview

This implementation plan breaks down the SahaayaAI conversational assistant into discrete, manageable coding tasks. The approach follows an incremental development strategy: starting with core data models and interfaces, building the eligibility engine, implementing the NLP and recommendation services, adding voice capabilities, and finally integrating all components with comprehensive testing.

Each task builds on previous work, ensuring no orphaned code. Testing tasks are integrated throughout to validate functionality early and often.

## Tasks

- [ ] 1. Set up project structure and core data models
  - Create TypeScript project with necessary dependencies (Express/Fastify, TypeScript, testing frameworks)
  - Define core data models: Scheme, UserAttributes, Session, EligibilityCriteria, ApplicationProcess
  - Implement JSON schema validation for data models
  - Set up testing framework (Jest) with hypothesis/fast-check for property-based testing
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 7.1, 10.1_

- [ ]* 1.1 Write property test for data model validation
  - **Property: Data Model Round Trip**
  - Test that serializing and deserializing data models preserves all fields
  - _Requirements: 10.1_

- [ ] 2. Implement Session Management Service
  - [ ] 2.1 Create SessionManager class with in-memory storage
    - Implement createSession, getSession, updateSession, clearSession, isSessionActive methods
    - Add session expiration logic (15-minute timeout)
    - _Requirements: 7.1, 7.3, 7.4, 7.5_
  
  - [ ]* 2.2 Write property tests for session management
    - **Property 18: Session State Persistence**
    - **Property 19: Session Expiration**
    - **Property 20: Session Isolation**
    - **Property 21: Explicit Session Clearing**
    - _Requirements: 7.1, 7.3, 7.4, 7.5_
  
  - [ ]* 2.3 Write unit tests for session edge cases
    - Test session at exactly 15 minutes
    - Test concurrent session operations
    - Test session with maximum conversation history
    - _Requirements: 7.3_

- [ ] 3. Implement Eligibility Engine Service
  - [ ] 3.1 Create EligibilityEngine class
    - Implement filterSchemes method with multi-criteria filtering logic
    - Implement checkEligibility method for detailed eligibility checking
    - Implement getMissingAttributes method to identify required information
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7_
  
  - [ ]* 3.2 Write property test for multi-criteria eligibility filtering
    - **Property 4: Multi-Criteria Eligibility Filtering**
    - Generate random user attributes and schemes, verify all criteria are satisfied
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6_
  
  - [ ]* 3.3 Write property test for missing information detection
    - **Property 5: Missing Information Detection**
    - Generate incomplete user attributes, verify correct identification of missing fields
    - _Requirements: 2.7_
  
  - [ ]* 3.4 Write unit tests for eligibility edge cases
    - Test age at exact min/max boundaries
    - Test income at exact threshold
    - Test empty allowed lists
    - Test schemes with no eligibility criteria
    - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5_

- [ ] 4. Checkpoint - Ensure core services work correctly
  - Run all tests for session management and eligibility engine
  - Verify data models are properly validated
  - Ask the user if questions arise

- [ ] 5. Implement Knowledge Base Service
  - [ ] 5.1 Create KnowledgeBase class with scheme storage
    - Implement methods to load, query, and update scheme data
    - Add caching layer for frequently accessed schemes
    - Implement scheme filtering by active status
    - _Requirements: 10.1, 10.2, 10.3, 8.4_
  
  - [ ]* 5.2 Write property tests for knowledge base operations
    - **Property 28: Knowledge Base as Source of Truth**
    - **Property 30: Active Schemes Only**
    - **Property 24: Scheme Information Caching**
    - _Requirements: 10.1, 10.3, 8.4_
  
  - [ ]* 5.3 Write unit tests for knowledge base edge cases
    - Test empty knowledge base
    - Test scheme updates
    - Test cache invalidation
    - _Requirements: 10.2, 8.4_

- [ ] 6. Implement NLP Service (stub with mock LLM)
  - [ ] 6.1 Create NLPService class
    - Implement processQuery method to extract intent and entities
    - Implement detectLanguage method for language identification
    - Implement extractEntities method for attribute extraction
    - Create mock LLM integration for testing (to be replaced with real Kiro integration)
    - _Requirements: 1.1, 1.3, 1.4, 1.5, 5.1_
  
  - [ ]* 6.2 Write property tests for NLP service
    - **Property 1: Intent Extraction for Supported Languages**
    - **Property 2: Clarification on Ambiguity**
    - **Property 3: Error Handling for Unintelligible Queries**
    - _Requirements: 1.1, 1.3, 1.4, 1.5_
  
  - [ ]* 6.3 Write unit tests for NLP edge cases
    - Test empty queries
    - Test maximum length queries
    - Test queries with special characters
    - Test language detection accuracy
    - _Requirements: 1.1, 1.4, 5.1_

- [ ] 7. Implement Recommendation Engine Service
  - [ ] 7.1 Create RecommendationEngine class
    - Implement rankSchemes method with relevance scoring algorithm
    - Implement generateResponse method for natural language responses
    - Implement generateApplicationGuidance method for detailed guidance
    - Add logic to limit recommendations to maximum 5 per response
    - _Requirements: 3.1, 3.2, 3.3, 3.5, 6.1, 6.2, 6.3, 6.4, 6.5_
  
  - [ ]* 7.2 Write property tests for recommendation engine
    - **Property 6: Relevance-Based Ranking**
    - **Property 7: Recommendation Response Completeness**
    - **Property 8: Detailed Scheme Information Completeness**
    - **Property 9: Maximum Recommendations Limit**
    - **Property 15: Application Guidance Completeness**
    - **Property 16: Document Checklist Generation**
    - _Requirements: 3.1, 3.2, 3.3, 3.5, 6.1, 6.2, 6.3_
  
  - [ ]* 7.3 Write unit tests for recommendation edge cases
    - Test empty eligible schemes list (no results scenario)
    - Test exactly 5 schemes
    - Test more than 5 schemes (verify truncation)
    - Test location-specific guidance
    - _Requirements: 3.4, 3.5, 6.4_

- [ ] 8. Checkpoint - Ensure recommendation pipeline works end-to-end
  - Test complete flow: query → NLP → eligibility → recommendation
  - Verify all components integrate correctly
  - Ask the user if questions arise

- [ ] 9. Implement Voice Services Integration
  - [ ] 9.1 Create VoiceService class with external API integration
    - Implement speechToText method with audio format validation
    - Implement textToSpeech method with audio compression
    - Add error handling for low-confidence transcriptions
    - _Requirements: 4.1, 4.3, 8.3_
  
  - [ ]* 9.2 Write property tests for voice services
    - **Property 10: Speech-to-Text Conversion**
    - **Property 11: Low Confidence Audio Handling**
    - **Property 23: Audio Compression**
    - _Requirements: 4.1, 4.3, 8.3_
  
  - [ ]* 9.3 Write unit tests for voice service edge cases
    - Test invalid audio formats
    - Test audio files exceeding size limits
    - Test empty audio data
    - _Requirements: 4.1_

- [ ] 10. Implement Multi-Language Support
  - [ ] 10.1 Add language management to session and response generation
    - Implement language detection and switching in SessionManager
    - Add language parameter to all response generation methods
    - Implement scheme translation lookup (name and description by language)
    - Ensure audio output matches query language
    - _Requirements: 5.1, 5.2, 5.4, 4.4_
  
  - [ ]* 10.2 Write property tests for language support
    - **Property 12: Language Consistency in Responses**
    - **Property 13: Session Language Initialization**
    - **Property 14: Scheme Terminology Consistency**
    - _Requirements: 4.4, 5.1, 5.2, 5.4_
  
  - [ ]* 10.3 Write unit tests for language edge cases
    - Test language switching mid-session
    - Test unsupported language handling
    - Test translation fallback behavior
    - _Requirements: 5.2_

- [ ] 11. Implement API Gateway and Request Handling
  - [ ] 11.1 Create Express/Fastify API server
    - Set up routes for query endpoint (POST /api/query)
    - Implement request validation and error handling
    - Add middleware for HTTPS/TLS enforcement
    - Implement response formatting
    - _Requirements: 8.1, 9.1_
  
  - [ ]* 11.2 Write property tests for API layer
    - **Property 22: Response Time Performance**
    - **Property 25: Encrypted Data Transmission**
    - _Requirements: 8.1, 9.1_
  
  - [ ]* 11.3 Write unit tests for API edge cases
    - Test malformed JSON requests
    - Test missing required fields
    - Test request timeout handling
    - _Requirements: 8.1_

- [ ] 12. Implement Conversational Assistant Core Orchestrator
  - [ ] 12.1 Create ConversationalAssistant class to coordinate all services
    - Wire together SessionManager, NLPService, EligibilityEngine, RecommendationEngine, VoiceService, KnowledgeBase
    - Implement main handleQuery method that orchestrates the complete flow
    - Add error handling and fallback logic for service failures
    - Implement uncertainty indication when confidence is low
    - _Requirements: 1.1, 1.3, 1.4, 2.6, 3.1, 10.5_
  
  - [ ]* 12.2 Write integration tests for complete query flow
    - Test text query end-to-end
    - Test voice query end-to-end
    - Test multi-turn conversation with context
    - Test error recovery scenarios
    - _Requirements: 1.1, 4.1, 7.1_
  
  - [ ]* 12.3 Write property test for uncertainty indication
    - **Property 32: Uncertainty Indication**
    - _Requirements: 10.5_

- [ ] 13. Implement Data Privacy and Security Features
  - [ ] 13.1 Add data encryption and privacy controls
    - Implement session data encryption at rest
    - Add data deletion endpoint and logic
    - Implement automatic PII cleanup on session expiration
    - Add audit logging for data access
    - _Requirements: 9.1, 9.2, 9.4_
  
  - [ ]* 13.2 Write property tests for privacy features
    - **Property 26: Session Data Non-Persistence**
    - **Property 27: Data Deletion Compliance**
    - _Requirements: 9.2, 9.4_
  
  - [ ]* 13.3 Write unit tests for security edge cases
    - Test data deletion request handling
    - Test session expiration cleanup
    - Test PII detection and handling
    - _Requirements: 9.2, 9.4_

- [ ] 14. Checkpoint - Ensure complete system works end-to-end
  - Run full test suite (unit + property + integration tests)
  - Test complete user journeys with multiple scenarios
  - Verify all 32 correctness properties pass
  - Ask the user if questions arise

- [ ] 15. Add sample scheme data and knowledge base seeding
  - [ ] 15.1 Create sample scheme data for testing
    - Add at least 20 representative schemes across categories (welfare, healthcare, education, employment)
    - Include schemes with various eligibility criteria combinations
    - Add schemes in all supported languages
    - _Requirements: 10.1, 5.4_
  
  - [ ]* 15.2 Write unit tests for data seeding
    - Test knowledge base initialization
    - Test scheme data validation
    - Test multi-language scheme loading
    - _Requirements: 10.1_

- [ ] 16. Implement monitoring and observability
  - [ ] 16.1 Add logging and metrics
    - Implement structured logging for all services
    - Add error tracking and alerting
    - Implement performance metrics collection (response time, cache hit rate)
    - Add health check endpoint
    - _Requirements: 8.1, 8.4_
  
  - [ ]* 16.2 Write unit tests for monitoring features
    - Test logging output format
    - Test metrics collection
    - Test health check endpoint
    - _Requirements: 8.1_

- [ ] 17. Replace mock LLM with real Kiro platform integration
  - [ ] 17.1 Integrate Kiro LLM API
    - Replace mock LLM in NLPService with real Kiro API calls
    - Implement prompt engineering for intent extraction and response generation
    - Add retry logic and error handling for API failures
    - Configure LLM parameters (temperature, max tokens, etc.)
    - _Requirements: 1.1, 1.3, 3.2, 3.3_
  
  - [ ]* 17.2 Write integration tests for Kiro LLM
    - Test intent extraction with real LLM
    - Test response generation quality
    - Test error handling for API failures
    - _Requirements: 1.1, 3.2_

- [ ] 18. Final checkpoint - Complete system validation
  - Run complete test suite one final time
  - Verify all 32 correctness properties pass with 100+ iterations each
  - Test with realistic user scenarios
  - Ensure all requirements are met
  - Ask the user if questions arise

## Notes

- Tasks marked with `*` are optional and can be skipped for faster MVP
- Each task references specific requirements for traceability
- Checkpoints ensure incremental validation throughout development
- Property tests validate universal correctness properties with 100+ iterations
- Unit tests validate specific examples, edge cases, and error conditions
- The implementation uses TypeScript with Express/Fastify for the backend
- Testing uses Jest with fast-check for property-based testing
- Mock LLM is used initially and replaced with real Kiro integration in task 17
