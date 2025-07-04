# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Solana Gym is a reinforcement learning environment for training agents to interact with the Solana blockchain. It implements a skill-based learning approach inspired by the Voyager paper, where agents learn by accumulating reusable TypeScript skills.

## Critical Migration Rules (from .clinerules/01-project.md)

### Language & Runtime Requirements
- **All new skills MUST be written in TypeScript** and export `async function executeSkill(env)`
- **Bun v1.x is the only approved runtime**
- Compilation budget ≤ 5s; execution budget ≤ 10s (enforced by `runSkill.ts`)
- Python skills are deprecated (keep legacy ones in `skills/legacy_py/` for reference only)
- Use `uv run` instead of `python ...`

### Subsystem Responsibilities
| Component | Must Handle | Must NOT Handle |
|-----------|-------------|-----------------|
| **Sandbox Service** | REST `/start /ready /shutdown`, Surfpool lifecycle | Running LLM or skill code |
| **Gym Adapter** | Convert RPC → tensor obs; enforce 15s step timeout | Skill compilation/execution |
| **Skill Runner** | Compile & run TS (`runSkill <file> <timeoutMs>`) | Managing validator or RPC |
| **RAG Store** | `put()` & `query()` over NumPy vectors (`vecs.npy`) | Network DBs (pgvector, etc.) |
| **Planner** | Build prompt, 4-try repair loop, emit TS | Direct RPC or filesystem calls |
| **Voyager Wrapper** | Discrete actions, reward shaping, CSV program-ID load | Embedding generation |

### Protocol Labeling (Critical Bug Fix)
- The labeler **MUST scan every instruction in a transaction receipt** and collect ALL program-IDs
- For each program-ID: map to project name via `data/program_ids.csv`
- If project hasn't been seen in current episode, add +1 exploration bonus
- A single transaction can yield **multiple +1 bonuses** for multiple new programs
- **DO NOT** stop after first match or ignore additional IDs

## Architecture

The codebase follows a hybrid Python-TypeScript architecture:

- **Python Layer**: Controls the RL environment (Gymnasium), manages state, orchestrates skill execution
- **TypeScript Layer**: Implements individual skills that interact with Solana via Bun runtime
- **Communication**: JSON-based IPC between Python and TypeScript processes

Key components:
- `voyager_env.py`: Main Gymnasium environment wrapper
- `surfpool_env.py`: Low-level Solana interaction environment
- `skill_manager/`: TypeScript skill management and execution
- `skill_runner/`: Bun-based TypeScript execution environment
- `planner/`: LLM-based skill planning and generation

## Development Commands

### Setup
```bash
# Python dependencies
uv pip install -r requirements.txt
uv pip install -e .

# TypeScript dependencies
cd skill_runner && bun install
```

### Testing
```bash
# Run all Python tests
uv run python -m unittest discover tests/python -v

# Run specific test
uv run python -m unittest tests.python.test_voyager_env

# TypeScript tests
cd skill_runner && bun test
```

**CI Requirements (from .clinerules):**
- All PRs must pass: `pytest` (Python)
- All PRs must pass: `bun test` & `bunx eslint . --max-warnings 0` (TypeScript)
- End-to-end dummy episode must have reward > 0

### Running
```bash
# Main demo
uv run python main.py

# Run environment directly
uv run python voyager_env.py
```

## Skill Development

Skills are TypeScript functions that must follow this pattern:

```typescript
import { Transaction, SystemProgram, PublicKey } from '@solana/web3.js';

export async function executeSkill(env: any): Promise<[number, string, string | null]> {
    // Build complete transaction with all instructions
    const tx = new Transaction();
    tx.add(/* your instructions */);
    
    // Serialize to base64
    const serializedTx = tx.serialize({
        requireAllSignatures: false,
        verifySignatures: false
    }).toString('base64');
    
    // Returns: [reward, done_reason, base64_serialized_unsigned_tx]
    return [1.0, "success", serializedTx];
}
```

Skills build complete transactions using Solana web3.js and return them as base64 strings.

**Requirements for new skills:**
- Must include one-paragraph description ≤ 80 tokens
- Must include unit test in `tests/ts/`
- Must be TypeScript (no new Python skills allowed)
- **CRITICAL: Each skill MUST create exactly ONE unsigned transaction**
- Build the complete transaction with all instructions using web3.js
- Serialize the entire transaction to base64 format
- Do NOT sign or send the transaction - the environment handles that
- Multiple operations can be added as instructions to the same transaction

## Important Implementation Notes

1. **Protocol Discovery**: 
   - Load `data/program_ids.csv` at runtime (NEVER hard-code program IDs)
   - Fail CI if CSV is missing or empty
   - Scan ALL instructions in transaction receipts

2. **RAG Vector Store**:
   - Use NumPy in-memory matrix for embeddings
   - Persist to `rag_store/vecs.npy`
   - `query()` must return top-k via brute-force cosine (no external services)

3. **Data Handling**: Never modify core data in CSVs. Handle quirks via post-processing.

## Key Files to Understand

- `voyager_env.py`: Main environment logic and reward system
- `skill_manager/ts_skill_manager.py`: How skills are executed
- `skill_runner/runSkill.ts`: TypeScript execution environment
- `tests/python/test_voyager_env.py`: Example usage patterns
- `.clinerules/01-project.md`: Critical project rules and migration guidelines

## Surfpool Specific Commands / Flags

- `surfpool start`: Initialize the Surfpool sandbox environment
- `surfpool status`: Check current status of the Surfpool sandbox
- `surfpool reset`: Reset the Surfpool sandbox to its initial state
- `surfpool logs`: View recent logs from the Surfpool service
- `--fast-mode`: Flag to run Surfpool in accelerated mode with reduced validation
- `--debug`: Enable verbose debugging for Surfpool interactions

## AI Model Considerations

- Use OpenRouter when possible over other providers to make it easy to swap out mode

## Milestones

- Complete milestone 3!

## Preferred Commands

- Use `python -m unittest discover tests/python -v` as our main way of running all the tests