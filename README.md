# AFv2 Pattern #4: Iteration

Quality-driven iterative refinement loop with convergence detection.

## Pattern Structure

```
Start → Planner → Gate → Research → [loop back to Gate] → Report → Direct Reply
```

## Key Features

- Loop-back edge (Research → Gate, animated)
- Scoring system (0.0-1.0 scale, target 0.85)
- Max 3 iterations with early exit on convergence
- Quality improvement tracking

## Files

- `04-iteration.json` - Complete Flowise workflow (760 lines)

## Quick Start

1. Import `04-iteration.json` into Flowise
2. Configure Anthropic API key for all agents
3. Test with content refinement task

## Use Cases

- Content refinement until quality threshold met
- Code optimization to performance target
- Data quality improvement cycles
- Iterative document enhancement

## Documentation

See [Context Foundry Pattern Library](https://github.com/snedea/afv2-patterns-index) for complete documentation.

🤖 Built with Context Foundry
