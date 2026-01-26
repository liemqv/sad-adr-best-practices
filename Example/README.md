# Examples Directory

This directory contains real-world examples demonstrating how to use the SAD and ADR templates.

## Contents

### SAD Example

- **`Uber-SAD.md`**: Complete Solution Architecture Document for a ride-sharing platform (Uber-like system)
  - Demonstrates all sections of the SAD template
  - Includes C4 Model diagrams (all 4 levels)
  - Complete ERD for ride-sharing domain
  - Real microservices architecture
  - Multi-region deployment strategy
  - Event-driven architecture with Kafka

### ADR Examples

- **`ADR-001-Example.md`**: Microservices Architecture Decision
  - Demonstrates all optional sections
  - Architecture Impact (Container diagrams, flows)
  - Security Impact
  - Performance Impact
  - Dependencies & Libraries
  - Deployment Impact

- **`ADR-002-Example.md`**: PostgreSQL Database Choice Decision
  - Focuses on Data Model Impact
  - Performance analysis
  - Infrastructure requirements
  - Security considerations

- **`ADR-011-Example.md`**: PCI DSS Compliance Decision
  - Security and compliance focus
  - Payment architecture
  - Tokenization strategy
  - Compliance requirements

## How to Use

1. **Review the examples** to understand how to apply the templates
2. **Copy the templates** from the root directory to your project
3. **Customize** with your specific architecture and decisions
4. **Reference these examples** when you need guidance on specific sections

## Notes

- All examples are based on the same fictional ride-sharing platform
- ADR examples reference each other to show how decisions relate
- Examples use Mermaid diagrams that render on GitHub
- All examples follow 2026 best practices
