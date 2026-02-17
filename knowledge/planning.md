# Strategic Planning

This document contains strategic phase planning for the project. It captures high-level direction and goals accumulated from historical conversations.

---

## Project Vision

**Core Goal**: Build a generalized probability calibration system that helps users understand the reliability of their predictions.

### Primary Objective

**Overcome ambiguity and characterize subjective probability execution** - Users often make vague predictions like "I think this will happen" without quantifying their confidence. This system aims to:

- Force explicit probability expression (not just "likely" but "70% confident")
- Track whether user's stated confidence matches reality
- Characterize user's subjective probability execution across different domains

### Secondary Objective

**Assess prediction quality by domain** - Based on external ground truth:

- Evaluate how well user assesses in specific areas (e.g., stock movements, event outcomes)
- Identify systematic biases or blind spots
- Build calibration curves to show over/under-confidence patterns

### User Pattern

- User makes several major decisions per week
- Results typically known within weeks
- Goal is not to predict better than before, but to have clear understanding of confidence level

---

## Core Architecture

The system consists of three layers:

### 1. Data Ingestion Layer (LLM Parser)

Transform user's vague conversations into structured schemas:

- Extract implicit probability expressions
- Identify resolution criteria
- Set expiration dates automatically

### 2. Business Logic Layer (Fatebook Core)

Utilize existing Fatebook systems:

- Prediction recording
- Result settlement
- API systems

### 3. Calibration Engine Layer (Calibration Brain) - NEW MODULE

- Export historical prediction data
- Calculate calibration parameters using MCMC models (a\*logit + b)
- Generate calibration curves and reports

---

## Current Phase: Project Analysis

### Goals

1. **Schema Deep Dive**: Analyze Prediction, Forecast, and User models in `prisma/schema.prisma`

   - Confirm existing fields support: Prediction (P), Confidence (C), Outcome
   - Identify gaps requiring extensions

2. **API Capability Assessment**: Review `pages/api` endpoints

   - Identify core endpoints for creating predictions
   - Identify endpoints for marking results
   - Design `/api/calibration` interface for data export

3. **MCMC Bridge Design**: Decide architecture for statistical computation
   - Option A: Pyodide (run Python in browser)
   - Option B: Python microservice
   - Option C: Pure TypeScript implementation

### Key Requirements

- Keep code modular; do not pollute Fatebook's core business logic
- All subjective probability records must require "resolution criteria" and "expiration date"
- Follow TDD approach: design test cases before implementation

---

## Milestone Tracking

### Phase 1: Analysis & Design (Current)

- [ ] Schema analysis complete
- [ ] API endpoints identified
- [ ] MCMC bridge decision made
- [ ] Data export interface designed

### Phase 2: Core Implementation

- [ ] Implement calibration data export API
- [ ] Implement MCMC computation layer
- [ ] Add calibration visualization

### Phase 3: User Experience

- [ ] Calibration dashboard
- [ ] Domain-specific analysis
- [ ] Bayesian update integration

---

## Decision Points

### MCMC Implementation Decision

| Option              | Pros                            | Cons                        |
| ------------------- | ------------------------------- | --------------------------- |
| Pyodide             | No backend needed, runs locally | Larger bundle, performance  |
| Python Microservice | Mature ecosystem (PyMC)         | Infrastructure complexity   |
| TypeScript          | No extra dependencies           | Less mature stats libraries |

### Data Model Extension

Need to decide if extending Fatebook schema or creating parallel calibration tables.

---

## Next Steps

When user asks "what should we do next?", consult this document and provide actionable options based on:

1. Current phase progress
2. Pending decisions
3. Dependencies between tasks
