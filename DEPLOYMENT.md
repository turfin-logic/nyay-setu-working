# Nyay Setu Deployment & Operations Guide

This document outlines the production deployment strategy, error recovery mechanisms, and rate limiting considerations for the Nyay Setu platform.

## Architecture Orchestration
The application relies on a microservices architecture managed via Docker Compose (`docker-compose.yml`).
It consists of:
1. **Web App (Port 8000)**: The main user interface and API gateway.
2. **NLP Orchestrator (Port 8001)**: Handles heavy ML inference and LLM orchestration.
3. **PostgreSQL Database (Port 5432)**: Persistent data storage.

## Error Recovery (NLP Orchestrator)
If the NLP Orchestrator service becomes unresponsive or crashes:
- **Automatic Restart**: The `docker-compose.yml` specifies `restart: always` for this service.
- **Fallback Mechanism**: The Web App is designed to detect a `503 Service Unavailable` from the NLP Orchestrator. In such cases, the Web App will fall back to cached responses (if available) or display a standard "Processing delayed" banner to the user, preventing a hard crash.

## Rate Limiting (Groq/Gemini APIs)
Nyay Setu relies on external LLM APIs which are subject to strict rate limits.
- **Handling Limits**: If a `429 Too Many Requests` is received, the NLP Orchestrator queues the request using an exponential backoff strategy (base 2 seconds, max 30 seconds).
- **Monitoring**: API usage is logged in the `struct_log.txt` to monitor consumption against the quota.

## Database Migrations
PostgreSQL schema migrations should be executed automatically via an `init.sql` script mounted to `/docker-entrypoint-initdb.d/` on fresh deployments, or manually via standard SQL migration scripts during version upgrades. Avoid making structural changes directly on the production database.
