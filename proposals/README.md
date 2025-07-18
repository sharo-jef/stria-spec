# Stria Language Proposals

This directory contains proposals for changes and additions to the Stria language specification.

## How to Submit a Proposal

1. **Fork the repository** and create a new branch for your proposal
2. **Copy the template** from [`0000-template.md`](0000-template.md) to `proposals/0000-my-feature.md`
3. **Fill in the template** with your proposal details
4. **Submit a pull request** with your proposal

## Proposal Process

### 1. Draft Phase

- Create a proposal using the template
- Discuss the idea in the pull request
- Iterate on feedback from maintainers and community

### 2. Review Phase

- Maintainers review the proposal for technical feasibility
- Community provides feedback on the design
- Proposal may be accepted, rejected, or require further iteration

### 3. Implementation Phase

- Once accepted, the proposal can be implemented
- Implementation may reveal issues requiring proposal updates
- Changes to the specification are made to reflect the implementation

## Proposal Status

- **Draft** - Initial proposal, under discussion
- **Accepted** - Proposal has been accepted and can be implemented
- **Rejected** - Proposal has been rejected with reasoning
- **Implemented** - Proposal has been implemented and is part of the specification
- **Withdrawn** - Proposal has been withdrawn by the author

## Proposal Categories

### Language Features

- New syntax additions
- Type system enhancements
- Control flow improvements

### Standard Library

- New built-in functions
- Standard library extensions
- Import system changes

### Tooling

- Compiler improvements
- IDE support features
- Development tools

### Schema System

- Schema validation enhancements
- New annotation types
- Configuration management features

## Active Proposals

| ID | Title | Status | Category |
|----|-------|--------|----------|
| [0001](0001-example-proposal.md) | Enhanced String Interpolation with Format Specifiers | Draft | Language Features |
| [0002](0002-schema-definition.md) | Improved schema definition | Accepted | Schema System |
## Recently Implemented

| ID  | Title | Implemented In | Category |
| --- | ----- | -------------- | -------- |
| -   | -     | -              | -        |

## Rejected Proposals

| ID  | Title | Rejection Reason | Category |
| --- | ----- | ---------------- | -------- |
| -   | -     | -                | -        |

## Guidelines

### Writing Good Proposals

1. **Be specific** - Clearly define what you're proposing
2. **Provide motivation** - Explain why this change is needed
3. **Consider alternatives** - Discuss other possible approaches
4. **Think about impact** - Consider how this affects existing code
5. **Include examples** - Show how the feature would be used

### Proposal Acceptance Criteria

- **Aligns with Stria's goals** - Fits the language's design philosophy
- **Technically feasible** - Can be implemented without breaking existing features
- **Well-motivated** - Solves a real problem or adds significant value
- **Minimal complexity** - Doesn't add unnecessary complexity to the language
- **Backward compatible** - Doesn't break existing valid Stria code (when possible)

## Contact

For questions about the proposal process, please:

- Open an issue in this repository
- Start a discussion in the pull request for your proposal
- Contact the maintainers directly

## License

All proposals and their implementations are subject to the same license as the Stria language specification.
