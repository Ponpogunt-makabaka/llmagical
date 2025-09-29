[README.md](https://github.com/user-attachments/files/22587427/README.md)
# LLMagical: Machine Generated Analog IC Layout with AI

LLMagical is an open-source automated analog integrated circuit (IC) layout synthesis framework designed for analog and mixed-signal circuit design. The framework combines machine learning techniques with traditional EDA algorithms and **AI-powered natural language automation** to provide a comprehensive solution for analog circuit placement and routing.

## Overview

LLMagical provides an end-to-end automated flow for analog IC layout generation, including:

- **Constraint Generation**: Automatic extraction and generation of layout constraints
- **Device Generation**: Automated analog device layout synthesis
- **Placement**: Hierarchical analog placement with symmetry and matching constraints
- **Routing**: Analog-aware routing with parasitic-conscious algorithms
- **LLM Integration**: Natural language interface for circuit design automation with multiple AI providers

## 🐳 Quick Start with Docker (Recommended)

The easiest way to get started with LLMagical's AI-enhanced features is using our pre-configured Docker environment:

### Step 1: Pull and Run Docker Container
```bash
docker run -it --rm \
  --network host \
  -v $(pwd)/flow/python/llm_controller:/install/MAGICAL/flow/python/llm_controller \
  -v $(pwd)/examples:/install/MAGICAL/examples \
  -v $(pwd)/llm_config.yaml:/install/MAGICAL/llm_config.yaml \
  magical-llm:latest bash
```

### Step 2: Configure Environment Inside Container
```bash
# Set LLM configuration path
export LLM_CONFIG_PATH=/install/MAGICAL/llm_config.yaml

# Configure Python paths
export PYTHONPATH=/install/MAGICAL/flow/python:/install/MAGICAL:$PYTHONPATH

# Verify setup
python -c "from llm_controller.orchestrator.llm_engine import LLMEngine; print('✓ LLMagical AI ready!')"
```

### Step 3: Run AI-Enhanced Circuit Analysis
```bash
cd /install/MAGICAL/examples/ota1
python /install/MAGICAL/flow/python/llm_controller/magical_llm.py ota1.json
```

## Project Structure

```
LLMagical/
├── llm_config.yaml         # 🆕 Consolidated LLM configuration
├── ConstGen/               # Constraint generation engine
├── IdeaPlaceEx/            # Analog placement engine
├── anaroute/               # Analog routing engine
├── device_generation/      # Device layout generation
├── flow/                   # Main Python flow control
│   ├── cpp/               # C++ backend bindings
│   └── python/            # Python flow orchestration
│       └── llm_controller/  # 🆕 AI-driven automation
├── examples/               # Example circuits and test cases
│   ├── ota1/              # Operational amplifier examples
│   ├── ota2/
│   ├── ota3/
│   ├── adc1/              # ADC examples
│   ├── adc2/
│   ├── comp/              # Comparator examples
│   └── mockPDK/           # Mock process design kit
└── build_*.sh             # Build scripts
```

## 🤖 AI Configuration

LLMagical supports multiple AI providers for natural language circuit design automation:

### Supported AI Providers
- **Anthropic Claude** (claude-3-opus, claude-3-sonnet, claude-3-haiku)
- **OpenAI GPT** (gpt-4-turbo, gpt-4, gpt-3.5-turbo)
- **Ollama** (Local models: llama2, codellama, mixtral, qwen3, deepseek-coder, gpt-oss)
- **Local Servers** (Custom model deployments)
- **Mock Mode** (Development and testing)

### Current Configuration
The system is pre-configured to use **Ollama with Qwen3-30B** for local AI processing:

```yaml
# In llm_config.yaml
active_provider: "ollama"
current_model:
  provider: "ollama"
  model: "qwen3:30b"
```

### Switching AI Providers
Edit `llm_config.yaml` to change providers:

```yaml
# For Anthropic Claude
current_model:
  provider: "anthropic"
  model: "claude-3-sonnet"

# For OpenAI GPT
current_model:
  provider: "openai"
  model: "gpt-4-turbo"

# For local Ollama models
current_model:
  provider: "ollama"
  model: "codellama:latest"
```

## Key Components

### 1. AI Controller (NEW)
Natural language interface for circuit design:
- Circuit analysis from text descriptions
- Automated constraint generation
- Multi-provider AI model support
- Local model integration via Ollama
- Interactive design optimization

### 2. Constraint Generation (ConstGen)
Automatically extracts layout constraints from circuit netlists:
- Symmetry constraints
- Matching requirements
- Proximity constraints
- Parasitic-sensitive net identification

### 3. Placement Engine (IdeaPlaceEx)
Hierarchical analog placement engine featuring:
- Symmetry-aware placement
- Matching device grouping
- Thermal and electrical optimization
- Support for complex analog topologies

### 4. Routing Engine (anaroute)
Analog-specific routing with:
- Symmetric routing for matched devices
- Parasitic-aware routing algorithms
- Multi-layer routing optimization
- Shielding and guard ring insertion

## Installation

### Prerequisites

- Docker (recommended) OR:
- Python 3.9+
- CMake 3.14+
- C++ compiler with C++17 support
- Boost libraries (≥1.74)
- Eigen3
- pybind11

### Option 1: Docker Installation (Recommended)

```bash
# Pull the pre-configured image
docker pull magical-llm:latest

# Run with volume mounts for development
docker run -it --rm \
  --network host \
  -v $(pwd):/workspace \
  magical-llm:latest bash
```

### Option 2: Local Installation

#### Dependencies

The project requires several external libraries:
- [Boost](https://www.boost.org) (≥1.74)
- [LIMBO](https://github.com/limbo018/Limbo) - Layout optimization library
- [Lemon](https://lemon.cs.elte.hu) - Graph algorithms library
- [Eigen](http://eigen.tuxfamily.org) - Linear algebra library
- [pybind11](https://github.com/pybind/pybind11) - Python-C++ bindings

#### Build from Source

1. **Clone the repository:**
```bash
git clone https://github.com/magical-eda/LLMagical.git
cd LLMagical
git submodule update --init --recursive
```

2. **Set up environment:**
```bash
conda env create -f environment.yml
conda activate llmagical
```

3. **Install AI dependencies:**
```bash
pip install anthropic openai pyyaml requests jsonschema
```

4. **Build C++ components (optional):**
```bash
# Minimal build (recommended for AI features)
./build_cpp_minimal.sh

# Full build with all optimizations
./build_cpp_only.sh
```

## Usage

### AI-Enhanced Flow (Recommended)

1. **Configure your AI provider** in `llm_config.yaml`

2. **Set API keys** (if using cloud providers):
```bash
export ANTHROPIC_API_KEY="your_key_here"
export OPENAI_API_KEY="your_key_here"
```

3. **Run AI-enhanced analysis:**
```bash
cd examples/ota1
PYTHONPATH=../../flow/python python ../../flow/python/llm_controller/magical_llm.py ota1.json
```

4. **Interactive circuit design:**
```python
from llm_controller.magical_llm import MagicalLLM

# Natural language circuit design
magical = MagicalLLM(
    "ota1.json",
    natural_language_input="Design a two-stage operational amplifier with rail-to-rail output and low offset"
)

# Run automated flow with AI guidance
magical.run()
```

### Traditional Flow

1. **Prepare input files:**
   - SPICE netlist (`.sp`)
   - Technology files (`.techfile`, `.lef`)
   - Configuration JSON

2. **Run traditional flow:**
```bash
cd examples/ota1
python ../../flow/python/Magical.py ota1.json
```

### Example Configuration

```json
{
    "spectre_netlist": "ota1.sp",
    "resultDir": "./",
    "techfile": "../mockPDK/mock.techfile",
    "simple_tech_file": "../mockPDK/techfile.simple",
    "lef": "../mockPDK/mock.lef"
}
```

## Examples

The `examples/` directory contains several reference designs:

- **OTA1-3**: Various operational amplifier topologies with AI analysis
- **ADC1-2**: Analog-to-digital converter designs including CTDSM
- **COMP**: Comparator circuits
- **mockPDK**: Example process design kit for testing

Each example includes:
- SPICE netlist
- Configuration files
- AI analysis capabilities
- Expected output references
- Run scripts

## 🔧 AI Development

### Local Model Setup with Ollama

1. **Install Ollama:**
```bash
curl -fsSL https://ollama.ai/install.sh | sh
```

2. **Pull desired models:**
```bash
ollama pull qwen3:30b
ollama pull codellama:latest
ollama pull mixtral:latest
```

3. **Configure LLMagical:**
```yaml
# In llm_config.yaml
current_model:
  provider: "ollama"
  model: "qwen3:30b"
```

### Adding Custom AI Providers

1. Extend `llm_engine.py` with new provider initialization
2. Add provider configuration to `llm_config.yaml`
3. Implement API calls in the AI engine

## Development

### Testing AI Features

```bash
# Test configuration loading
python -c "from llm_controller.orchestrator.llm_engine import LLMEngine; engine = LLMEngine(); print('✓ Config loaded')"

# Test circuit analysis
cd examples/ota1
PYTHONPATH=../../flow/python python -c "
import sys; sys.path.append('../../flow/python/llm_controller')
from orchestrator.llm_engine import LLMEngine
engine = LLMEngine()
result = engine.generate_circuit_analysis('Analyze this OTA circuit')
print('Circuit type:', result.get('circuit_type'))
"
```

### Docker Development

```bash
# Development with live code mounting
docker run -it --rm \
  --network host \
  -v $(pwd):/workspace \
  -v $(pwd)/llm_config.yaml:/install/MAGICAL/llm_config.yaml \
  magical-llm:latest bash
```

### Contributing

1. Fork the repository
2. Create a feature branch
3. Add tests for AI features
4. Submit a pull request

#### Code Structure

- C++ components use CMake build system
- Python components follow PEP 8 style
- AI integration uses modern async patterns
- All public APIs include documentation

## Troubleshooting

### Common Issues

**"No module named 'yaml'"**
```bash
pip install pyyaml
```

**"magicalFlow not found"**
- The C++ backend is optional for AI features
- Use mock mode: set `provider: "mock"` in `llm_config.yaml`

**Ollama connection errors**
```bash
# Check Ollama is running
ollama list
# Start Ollama service
ollama serve
```

**Docker network issues**
- Use `--network host` for local model access
- Ensure Ollama is accessible on `localhost:11434`

**YAML configuration errors**
```bash
# Fix tab characters in config file
python3 -c "
with open('llm_config.yaml', 'r') as f: content = f.read()
with open('llm_config.yaml', 'w') as f: f.write(content.replace('\t', '    '))
"
```

## Performance

### AI Response Times
- **Ollama (Local)**: 2-10 seconds depending on model size
- **Cloud APIs**: 1-3 seconds (with network latency)
- **Mock Mode**: <1 second (for testing)

### Recommended Models
- **Development**: Mock mode or small local models
- **Production**: Claude-3-Sonnet or GPT-4-Turbo
- **Offline**: Ollama with CodeLlama or Qwen3

## License

LLMagical is released under the MIT License. See individual component directories for specific licensing information.

## Publications

If you use LLMagical in your research, please cite:

```bibtex

@inproceedings{magical2019,
  title={MAGICAL: Toward Fully Automated Analog IC Layout Leveraging Human and Machine Intelligence},
  author={Authors},
  booktitle={Proceedings of IEEE/ACM International Conference on Computer-Aided Design},
  year={2019}
}
```

## Support

- **Documentation**: [Project Wiki](https://github.com/magical-eda/LLMagical/wiki)
- **Issues**: [GitHub Issues](https://github.com/magical-eda/LLMagical/issues)
- **Discussions**: [GitHub Discussions](https://github.com/magical-eda/LLMagical/discussions)
- **AI Features**: See `flow/python/llm_controller/README.md`

## Acknowledgments

LLMagical development is supported by various academic and industry collaborations. The project builds upon decades of analog IC design automation research and modern AI advances.

The AI integration leverages state-of-the-art language models to make analog circuit design more accessible through natural language interfaces, representing the next evolution in EDA tools.

---

**Note**: This project includes mock PDK files for demonstration purposes only. For production use, obtain appropriate process design kits from your foundry partner.

🎯 **LLMagical: Where Traditional EDA Meets Modern AI**
