# Solana Gym: Teaching AI Agents to Navigate DeFi

> **An open-source reinforcement learning environment where AI agents learn to autonomously interact with the Solana blockchain, discovering protocols and building complex DeFi strategies from scratch.**

## =� The Vision: Autonomous DeFi Agents

Imagine AI agents that can:
- **Discover** new DeFi protocols on their own
- **Learn** to execute complex trading strategies
- **Optimize** yield farming across multiple protocols
- **Manage** portfolios with superhuman efficiency
- **Contribute** back to the ecosystem by documenting their discoveries

This isn't science fictionit's what we're building with Solana Gym.

### Why This Matters

The intersection of AI and crypto represents one of the most exciting frontiers in technology. While humans struggle to keep up with the explosive growth of DeFi protocols, AI agents can:

1. **Scale infinitely**: Monitor thousands of protocols simultaneously
2. **React instantly**: Execute strategies in milliseconds
3. **Learn continuously**: Improve from every transaction
4. **Share knowledge**: Build on each other's discoveries

**The endgame?** AI agents that can read their own on-chain positions through Jupiter Portfolio (formerly SonarWatch), optimize liquidity provision, discover arbitrage opportunities, and even contribute improvements back to the protocols they use.

## <� What is Solana Gym?

Solana Gym is a reinforcement learning environment that teaches AI agents to interact with Solana. Inspired by the groundbreaking [Voyager paper](https://voyager.minedojo.org/), our agents learn by doingstarting with zero knowledge and progressively building a library of reusable skills.

### Key Features

- **>� Self-Learning**: Agents generate their own TypeScript code to interact with Solana
- **=� Skill Library**: Accumulated knowledge persists across episodes
- **<� Gym Interface**: Standard OpenAI Gymnasium API for easy integration
- **= Protocol Discovery**: Automatic detection and reward for finding new protocols
- **� Real Blockchain**: Runs against actual Solana test validators
- **=� Safe Environment**: Sandboxed execution prevents costly mainnet mistakes

## <� Architecture

```
                                                             
                    AI Agent (Your Code)                     
                     ,                                       
                       Actions & Observations
                     �                                       
              Solana Voyager Environment                     
                                                          
   " Skill-based action space                             
   " Protocol discovery rewards                           
   " LLM-powered skill generation                         
                                                          
                     ,                                       
                       Skill Execution
                     �                                       
              TypeScript Skill Runner (Bun)                  
                                                          
   " Isolated execution environment                       
   " Mock Solana objects for testing                      
   " Transaction building and signing                     
                                                          
                     ,                                       
                       RPC Calls
                     �                                       
              Solana Test Validator (Surfpool)               
                                                          
   " Local blockchain instance                            
   " Pre-funded test accounts                             
   " Real protocol deployments                            
                                                          
                                                             
```

## =� Quick Start

### Prerequisites

- Python 3.8+
- [Bun](https://bun.sh) v1.x
- [uv](https://github.com/astral-sh/uv) (Python package manager)
- OpenRouter API key for LLM access

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/solana-gym.git
cd solana-gym

# Install Python dependencies
uv pip install -r requirements.txt
uv pip install -e .

# Install TypeScript dependencies
cd skill_runner && bun install
cd ..

# Set up your OpenRouter API key
export OPENROUTER_API_KEY="your-api-key-here"
```

### Run Your First Agent

```python
import gymnasium as gym
from solana_gym import SolanaVoyagerEnv

# Create the environment
env = gym.make('SolanaVoyager-v0')

# Reset to get initial observation
obs, info = env.reset()

# Let the agent explore!
for _ in range(100):
    # Your agent logic here
    # For now, let's try generating a new skill
    action = env.action_space.sample()
    
    obs, reward, terminated, truncated, info = env.step(action)
    
    if reward > 0:
        print(f"<� Discovered protocol: {info.get('discovered_protocol')}")
    
    if terminated or truncated:
        break
```

## >� How It Works

### 1. **Observation Space**
Agents receive rich observations about the blockchain state:
- Wallet balances (SOL and SPL tokens)
- Recent transaction history
- Available protocols
- Current skill library
- Network statistics

### 2. **Action Space**
Agents can take three types of actions:
- **Execute Skill**: Run an existing skill from the library
- **Generate Skill**: Use LLM to create a new TypeScript skill
- **Inspect Library**: View available skills and their descriptions

### 3. **Reward System**
- **+1** for each new protocol discovered in an episode
- **+0** for repeated protocol interactions
- **Variable** skill-specific rewards (e.g., profitable trades)

### 4. **Skill Generation**
When generating new skills, the agent:
1. Observes current state and objectives
2. Queries LLM with context and examples
3. Receives TypeScript code
4. Tests in sandboxed environment
5. Adds to library if successful

Example generated skill:
```typescript
export async function executeSkill(env: any): Promise<[number, string, string | null]> {
    // Swap SOL for USDC on Jupiter
    const wallet = env.getWallet();
    const jupiterProgram = env.getProgram('JUP4Fb2cqiRUcaTHdrPC8h2gNsA2ETXiPDD33WcGuJB');
    
    const tx = await jupiterProgram.swap({
        inputMint: NATIVE_MINT,
        outputMint: USDC_MINT,
        amount: 0.1 * LAMPORTS_PER_SOL,
        slippage: 1, // 1%
    });
    
    const receipt = await env.sendTransaction(tx);
    return [0, "skill_complete", JSON.stringify(receipt)];
}
```

## <� Use Cases

### For Researchers
- Study emergent behaviors in decentralized systems
- Benchmark reinforcement learning algorithms
- Explore multi-agent coordination in DeFi

### For Developers
- Test DeFi protocols with intelligent agents
- Generate documentation from agent discoveries
- Build autonomous trading systems

### For DeFi Protocols
- Stress test with unpredictable agent behavior
- Discover UX improvements from agent struggles
- Measure protocol discoverability

## =� Roadmap

### Phase 1: Foundation (Current)
-  Basic RL environment
-  TypeScript skill execution
-  Protocol discovery system
-  LLM integration for skill generation

### Phase 2: Capabilities
- = Complex multi-step skills
- = Persistent skill improvement
- = Cross-protocol strategies
- = Real-time market data integration

### Phase 3: Production
- =� Mainnet-beta support with safety limits
- =� Multi-agent coordination
- =� Integration with Jupiter Portfolio for position tracking
- =� Community skill marketplace

### Phase 4: Ecosystem
- =� Agent-to-agent skill sharing
- =� Automated protocol documentation
- =� Contribution system for protocol improvements
- =� DAO governance for agent behavior

## > Contributing

We're building the future of autonomous DeFi, and we need your help! Whether you're an RL researcher, Solana developer, or DeFi enthusiast, there's a place for you.

### Ways to Contribute

1. **Add Protocol Mappings**: Help us map more Solana programs in `data/program_ids.csv`
2. **Improve Skill Generation**: Enhance the LLM prompts and skill templates
3. **Build Better Rewards**: Design reward functions for specific DeFi strategies
4. **Create Benchmarks**: Develop standard tasks for evaluating agents
5. **Write Skills**: Contribute hand-crafted skills as examples

### Development Setup

```bash
# Run tests
uv run python -m pytest tests/python/
cd skill_runner && bun test

# Lint code
cd skill_runner && bunx eslint . --max-warnings 0

# Type check
cd skill_runner && bunx tsc --noEmit
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

## 📁 Project Structure

```
solana-gym/
├── voyager_env.py          # Main Gymnasium environment
├── surfpool_env.py         # Low-level Solana interaction layer
├── planner.py              # LLM-based skill generation
├── skill_library.py        # Skill management utilities
├── main.py                 # Entry point
│
├── docs/                   # Documentation
│   ├── MILESTONES.md      # Project roadmap and progress
│   ├── PLAN.md            # Technical implementation plan
│   └── ...                # Other docs (blockers, tracking, etc.)
│
├── examples/              # Example code and demos
│   ├── demos/            # Demo scripts and sample data
│   └── tests/            # Test scripts for various components
│
├── tracking/             # Trajectory and transaction tracking
│   ├── trajectory_tracker.py
│   ├── transaction_parser.py
│   └── voyager_env_with_tracking.py
│
├── visualizations/       # HTML dashboards and analytics
│   ├── dashboard.html
│   ├── jupiter_analysis.html
│   └── ...
│
├── skill_manager/        # TypeScript skill management
├── skill_runner/         # Bun runtime for skills
├── tests/               # Unit tests
└── data/                # Program IDs and other data
```

## =� Documentation

- [Architecture Overview](docs/architecture.md)
- [Skill Development Guide](docs/skills.md)
- [Environment Configuration](docs/configuration.md)
- [API Reference](docs/api.md)

## =, Research

This project is inspired by several key papers:
- [Voyager: An Open-Ended Embodied Agent](https://voyager.minedojo.org/)
- [Large Language Models as Tool Makers](https://arxiv.org/abs/2305.17126)
- [ReAct: Synergizing Reasoning and Acting](https://arxiv.org/abs/2210.03629)

## =� License

MIT License - see [LICENSE](LICENSE) for details.

## =O Acknowledgments

- OpenAI Gym/Gymnasium for the RL framework
- Solana Labs for the blockchain infrastructure
- Anthropic/OpenAI for LLM capabilities
- The DeFi community for building amazing protocols

---

**Ready to teach AI to DeFi?** Star P this repo and join us in building the future of autonomous finance!

*"The best way to predict the future is to invent it."* - Alan Kay