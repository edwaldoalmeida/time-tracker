# Software Design Specification

## STRATEGY
- Use the Hexagonal Architecture [https://netflixtechblog.com/ready-for-changes-with-hexagonal-architecture-b315ec967749]

## TECH STACK

### frontend
  - React
  - Redux
  - TypeScript
  * unit tests
    - Jest (unit tests) / Vite + Vitest

### backend
  - Java
  - Spring/Spring Boot
  * unit tests
    - JUnit
    - JMeter ?

### database
  - PostgreSQL

### system tests
  - Python + Pytest

### UI/end-to-end tests
  - Cucumber-Gherkin / Playwright

### deployment
  - container-based
  - deployment in K8s (or K3s)

### general
  - data layer: ORM to be able to swap database (and to learn how to work with ORMs)
  - architecture: approach such as that I'd be able to easily swap backend or frontend
