# LLMagaical Getting Started Guide

**Version**: v3.0.0
**Last Updated**: 2025-09-30
**Target Audience**: Engineers, Students, Researchers

---

## 🚀 Quick Start (5 Minutes)

### Step 1: Installation

```bash
# Clone the repository (if you haven't already)
git clone https://github.com/yourusername/llmagaical.git
cd llmagaical

# Install Python dependencies
pip install -e .
```

### Step 2: Verify Installation

```bash
# Run basic tests
python test_basic.py
```

**Expected output**:
```
🎉 All tests passed!
✓ Passed: 16
⊘ Skipped: 2  (C++ backend not built)
```

### Step 3: Run Your First Example

```bash
# Generate 5 common analog circuits
python examples/draw_analog_circuits.py
```

**You should see**:
```
🎨 Drawing Analog Circuits Example

1. Generating current mirror...
   ✅ Done! 2 devices, 0.029s
   📦 Output: output/analog_circuits/current_mirror/

2. Generating differential pair...
   ✅ Done! 3 devices, 0.022s
   📦 Output: output/analog_circuits/differential_pair/

... (5 circuits total)

🎉 All done!
📁 Output: output/analog_circuits/
📊 Total: 24 individual GDS files + 5 merged versions
⏱️  Time: 0.106s
```

### Step 4: View the Results

```bash
# List generated GDS files
ls output/analog_circuits/current_mirror/gds/

# Output:
# M1.gds  M2.gds  current_mirror_final_version.gds
```

**Congratulations!** You've just generated your first IC layout! 🎉

---

## 📖 Understanding the Basics

### Output Structure

Every design generates the following structure:

```
output/my_circuit/
├── gds/                          # Layout files
│   ├── M1.gds                    # Individual device 1
│   ├── M2.gds                    # Individual device 2
│   └── xxx_final_version.gds     # Complete circuit (recommended)
├── reports/
│   └── design_report.txt         # Design report
└── temp/                         # Temporary files
```

**Key file**: `xxx_final_version.gds` - This is the complete circuit layout containing all devices, ready to open in KLayout or other GDS viewers.

### Basic Concepts

#### 1. CircuitSpec
Defines your circuit at a high level:
- Circuit type (OTA, current mirror, etc.)
- List of devices
- Placement and routing strategies

#### 2. DeviceSpec
Defines individual devices:
- Name (e.g., "M1", "M2")
- Type (NMOS, PMOS, capacitor, resistor)
- Dimensions (width, length)
- Multiplier (for parallel devices)

#### 3. EDA Flow
The automated process:
1. Device generation (create device layouts)
2. Placement (position devices)
3. Routing (connect devices)
4. GDS output (generate files)

---

## 🎯 Tutorial 1: Using the Command Line

### Basic Usage

```bash
# Design with mock provider (no API key needed)
llmagaical design "Design a current mirror" --provider mock

# Specify output name
llmagaical design "Design an operational amplifier" --name my_ota --provider mock

# Check system status
llmagaical check
```

### Using Real LLMs

```bash
# Set your API key
export ANTHROPIC_API_KEY="your_key_here"

# Design with Claude
llmagaical design "Design a low-noise two-stage OTA with Miller compensation" --provider anthropic

# Design with GPT
export OPENAI_API_KEY="your_key_here"
llmagaical design "Design a bandgap reference" --provider openai
```

---

## 🎯 Tutorial 2: Using the Python API

### Example 1: Simple Current Mirror

Create a file `my_design.py`:

```python
from pathlib import Path
from llmagaical.core.config import LLMagaicalConfig
from llmagaical.core.types import (
    CircuitSpec, DeviceSpec, DeviceType, CircuitType
)
from llmagaical.eda import EDAFlowController

# 1. Create configuration
config = LLMagaicalConfig.get_default()

# 2. Define circuit specification
circuit_spec = CircuitSpec(
    circuit_type=CircuitType.CURRENT_MIRROR,
    natural_input="Simple current mirror",
    devices=[
        DeviceSpec(
            name="M1",
            device_type=DeviceType.PMOS,
            width=10.0,        # Width in µm
            length=0.5,        # Length in µm
            multiplier=2       # 2 parallel devices
        ),
        DeviceSpec(
            name="M2",
            device_type=DeviceType.PMOS,
            width=10.0,
            length=0.5,
            multiplier=2
        ),
    ]
)

# 3. Run EDA flow
output_dir = Path("./output/my_design")
controller = EDAFlowController(config.eda, output_dir)
result = controller.run_flow(circuit_spec)

# 4. Check results
if result.success:
    print(f"✅ Design successful!")
    print(f"   Devices: {result.metrics['total_devices']}")
    print(f"   GDS files: {result.metrics['gds_files']}")
    print(f"   Total area: {result.metrics['total_area']:.2f} µm²")
    print(f"   Time: {result.metrics['flow_time']:.3f}s")
else:
    print(f"❌ Design failed: {result.error}")
```

Run it:
```bash
python my_design.py
```

### Example 2: Two-Stage OTA

```python
circuit_spec = CircuitSpec(
    circuit_type=CircuitType.OTA,
    natural_input="Two-stage operational amplifier",
    devices=[
        # Input differential pair
        DeviceSpec("M1", DeviceType.NMOS, width=20.0, length=0.5, multiplier=4),
        DeviceSpec("M2", DeviceType.NMOS, width=20.0, length=0.5, multiplier=4),
        # Active load (current mirror)
        DeviceSpec("M3", DeviceType.PMOS, width=10.0, length=0.5, multiplier=2),
        DeviceSpec("M4", DeviceType.PMOS, width=10.0, length=0.5, multiplier=2),
        # Second stage
        DeviceSpec("M5", DeviceType.NMOS, width=40.0, length=0.5, multiplier=8),
        DeviceSpec("M6", DeviceType.PMOS, width=20.0, length=0.5, multiplier=4),
        # Tail current source
        DeviceSpec("M7", DeviceType.NMOS, width=15.0, length=1.0, multiplier=2),
        # Miller compensation capacitor
        DeviceSpec("CC", DeviceType.CAPACITOR, width=25.0, length=25.0),
    ]
)

result = controller.run_flow(circuit_spec)
```

---

## 🎯 Tutorial 3: Custom Circuit Design

### Step-by-Step Circuit Design

#### 1. Define Your Requirements

Example: **Differential Amplifier**
- Input: Differential pair (2 NMOS)
- Load: Active load (2 PMOS)
- Bias: Tail current source (1 NMOS)
- Total: 5 MOSFETs

#### 2. Choose Device Parameters

Consider:
- **Width (W)**: Affects current capability and transconductance
- **Length (L)**: Affects output resistance and matching
- **Multiplier (M)**: Number of parallel devices

Guidelines:
```python
# For high current: Large W, small L
DeviceSpec("M1", DeviceType.NMOS, width=40.0, length=0.5, multiplier=8)

# For high output resistance: Small W, large L
DeviceSpec("M2", DeviceType.NMOS, width=10.0, length=2.0, multiplier=1)

# For matching: Same dimensions, use multiplier
DeviceSpec("M3", DeviceType.PMOS, width=20.0, length=0.5, multiplier=4)
DeviceSpec("M4", DeviceType.PMOS, width=20.0, length=0.5, multiplier=4)
```

#### 3. Create the Circuit

```python
from llmagaical.core.types import CircuitSpec, DeviceSpec, DeviceType, CircuitType

circuit_spec = CircuitSpec(
    circuit_type=CircuitType.DIFFERENTIAL_AMPLIFIER,
    natural_input="Custom differential amplifier",
    devices=[
        # Input pair (matched)
        DeviceSpec("M1", DeviceType.NMOS, width=20.0, length=0.5, multiplier=4),
        DeviceSpec("M2", DeviceType.NMOS, width=20.0, length=0.5, multiplier=4),

        # Active load (matched)
        DeviceSpec("M3", DeviceType.PMOS, width=15.0, length=0.5, multiplier=3),
        DeviceSpec("M4", DeviceType.PMOS, width=15.0, length=0.5, multiplier=3),

        # Tail current source
        DeviceSpec("M5", DeviceType.NMOS, width=12.0, length=1.0, multiplier=2),
    ]
)
```

#### 4. Add Placement Constraints (Optional)

```python
from llmagaical.core.types import PlacementConstraints, SymmetryConstraint

constraints = PlacementConstraints(
    symmetry_pairs=[
        SymmetryConstraint(devices=["M1", "M2"], axis="vertical"),
        SymmetryConstraint(devices=["M3", "M4"], axis="vertical"),
    ]
)

circuit_spec.placement_constraints = constraints
```

#### 5. Generate and Verify

```python
from pathlib import Path
from llmagaical.eda import EDAFlowController
from llmagaical.core.config import LLMagaicalConfig

config = LLMagaicalConfig.get_default()
output_dir = Path("./output/custom_diffamp")
controller = EDAFlowController(config.eda, output_dir)

result = controller.run_flow(circuit_spec)

if result.success:
    print(f"✅ Success!")
    print(f"📦 Output: {output_dir}/gds/")
    print(f"📊 Area: {result.metrics['total_area']:.2f} µm²")
```

---

## 🎯 Tutorial 4: Importing SPICE Netlists

### Supported SPICE Syntax

LLMagaical can parse standard SPICE netlists:

**Example: simple_ota.sp**
```spice
* Simple OTA
.TITLE Simple Operational Transconductance Amplifier

* Input differential pair
M1 out1 in_p tail vss nch w=20u l=0.5u
M2 out2 in_n tail vss nch w=20u l=0.5u

* Active load
M3 out1 out1 vdd vdd pch w=10u l=0.5u
M4 out2 out1 vdd vdd pch w=10u l=0.5u

* Tail current source
M5 tail vbias vss vss nch w=15u l=1u

* Miller compensation
CC out2 out1 5p

.END
```

### Import and Generate Layout

```python
from llmagaical.netlist import parse_spice_file, convert_to_circuit_spec
from llmagaical.eda import EDAFlowController
from llmagaical.core.config import LLMagaicalConfig
from pathlib import Path

# 1. Parse SPICE file
netlist = parse_spice_file("simple_ota.sp")

# 2. Convert to CircuitSpec (circuit type auto-detected)
circuit_spec = convert_to_circuit_spec(netlist)

# 3. Generate layout
config = LLMagaicalConfig.get_default()
output_dir = Path("./output/spice_import")
controller = EDAFlowController(config.eda, output_dir)
result = controller.run_flow(circuit_spec)

# 4. Check result
print(f"Circuit type detected: {circuit_spec.circuit_type}")
print(f"Devices imported: {len(circuit_spec.devices)}")
print(f"Layout generated: {result.success}")
```

---

## ⚙️ Configuration and Parameters

### Global Configuration

Configuration is managed through `LLMagaicalConfig`:

```python
from llmagaical.core.config import LLMagaicalConfig

# Get default configuration
config = LLMagaicalConfig.get_default()

# Customize LLM settings
config.llm.provider = "anthropic"
config.llm.model = "claude-3-5-sonnet-20241022"
config.llm.temperature = 0.1
config.llm.max_tokens = 4096

# Customize EDA settings
config.eda.enable_cpp_backend = True
config.eda.parallel_device_gen = True
config.eda.max_workers = 8

# Customize PDK
config.pdk.name = "tsmc180nm"
config.pdk.foundry = "TSMC"

# Save configuration
config.save_to_file("my_config.yaml")

# Load configuration
config = LLMagaicalConfig.from_file("my_config.yaml")
```

### EDA Flow Parameters

#### Device Generation

```python
from llmagaical.eda.device_gen import MOSFETGenerator

# Customize device generation
generator = MOSFETGenerator(pdk_config=config.pdk)

device = generator.generate(
    name="M1",
    device_type=DeviceType.NMOS,
    width=10.0,           # µm
    length=0.5,           # µm
    multiplier=2,         # parallel devices
    finger_width=5.0      # µm per finger (optional)
)
```

#### Placement Parameters

```python
from llmagaical.eda.backends.placement import PlacementBackend, PlacementStrategy

backend = PlacementBackend(config.eda.backend_config)

result = backend.place(
    devices=devices,
    constraints=placement_constraints,
    strategy=PlacementStrategy.SYMMETRY_AWARE,  # Choose strategy
    grid_size=0.005,                            # µm
    spacing_factor=1.5                          # multiplier for min spacing
)
```

#### Routing Parameters

```python
from llmagaical.eda.backends.routing import RoutingBackend, RoutingStrategy

backend = RoutingBackend(config.eda.backend_config)

result = backend.route(
    netlist=netlist,
    positions=positions,
    constraints=routing_constraints,
    strategy=RoutingStrategy.PARASITIC_AWARE,  # Choose strategy
    min_width=0.3,                             # µm
    min_spacing=0.5                            # µm
)
```

### PDK Configuration

PDK settings are stored in YAML files under `configs/pdk/`:

**Example: configs/pdk/tsmc180nm.yaml**
```yaml
pdk:
  name: "tsmc180nm"
  node: "180nm"
  foundry: "TSMC"

  layers:
    active:
      gds_layer: 1
      gds_datatype: 0
    poly:
      gds_layer: 2
      gds_datatype: 0
    contact:
      gds_layer: 3
      gds_datatype: 0
    metal1:
      gds_layer: 4
      gds_datatype: 0

  design_rules:
    min_width:
      active: 0.22
      poly: 0.18
      metal1: 0.23

    min_spacing:
      active: 0.27
      poly: 0.22
      metal1: 0.28

    min_area:
      active: 0.0484
      metal1: 0.0529
```

---

## 🔧 Advanced Features

### 1. Parallel Processing

```python
from llmagaical.utils.parallel import parallel_device_generation

# Enable parallel device generation
devices = parallel_device_generation(
    device_specs=circuit_spec.devices,
    device_library=device_library,
    max_workers=8  # Use 8 CPU cores
)

# 4-8x speedup depending on number of cores
```

### 2. Caching

```python
from llmagaical.utils.cache import SimpleCache, DiskCache

# Memory cache
memory_cache = SimpleCache(max_size=1000)

# Disk cache for persistent storage
disk_cache = DiskCache(cache_dir="./cache")
disk_cache.set("expensive_result", large_data)
data = disk_cache.get("expensive_result")
```

### 3. C++ Backend (Optional)

For 10-100x performance improvement, build the C++ backends:

```bash
# Install system dependencies
./INSTALL_DEPS.sh

# Build anaroute (routing backend)
./scripts/build_anaroute.sh

# Verify installation
python test_basic.py
```

**Performance Comparison**:

| Circuit Size | Python | C++ | Speedup |
|--------------|--------|-----|---------|
| Small (2-5 devices) | 0.5s | 0.05s | **10x** |
| Medium (5-10 devices) | 1.5s | 0.15s | **10x** |
| Large (20+ devices) | 3-5s | 0.3-0.5s | **10x** |

### 4. Custom Device Types

Add new device types by extending `BaseDevice`:

```python
from llmagaical.eda.device_gen.base import BaseDevice

class InductorGenerator(BaseDevice):
    """Custom inductor generator."""

    def _generate_layout(self):
        # Implement inductor spiral layout
        pass

    def _calculate_dimensions(self):
        # Calculate inductor dimensions
        pass

# Register the generator
from llmagaical.eda.device_gen import DEVICE_GENERATORS
DEVICE_GENERATORS[DeviceType.INDUCTOR] = InductorGenerator
```

---

## 📊 Viewing Results

### Method 1: KLayout (Recommended)

```bash
# Install KLayout
sudo apt install klayout  # Ubuntu/Debian
brew install --cask klayout  # macOS

# Open GDS file
klayout output/my_circuit/gds/xxx_final_version.gds
```

**KLayout Features**:
- View all layers
- Zoom and pan
- Measure dimensions
- DRC checking

### Method 2: Python (gdspy)

```python
import gdspy

# Read GDS file
lib = gdspy.GdsLibrary(infile="output/my_circuit/gds/xxx_final_version.gds")

# Get top-level cell
top_cell = lib.top_level()[0]

# Print info
print(f"Library: {lib.name}")
print(f"Cells: {len(lib.cells)}")
print(f"Bounding box: {top_cell.get_bounding_box()}")

# Get layer statistics
for layer in range(1, 7):
    polygons = top_cell.get_polygons(by_spec=(layer, 0))
    if polygons is not None and len(polygons) > 0:
        print(f"Layer {layer}: {len(polygons)} polygons")
```

---

## 💡 Best Practices

### 1. Device Sizing

**For current mirrors**:
```python
# Use matching multipliers for symmetry
DeviceSpec("M1", DeviceType.PMOS, width=10.0, length=0.5, multiplier=2)
DeviceSpec("M2", DeviceType.PMOS, width=10.0, length=0.5, multiplier=2)
```

**For differential pairs**:
```python
# Larger W/L for input devices
DeviceSpec("M1", DeviceType.NMOS, width=20.0, length=0.5, multiplier=4)
DeviceSpec("M2", DeviceType.NMOS, width=20.0, length=0.5, multiplier=4)

# Smaller W/L for tail current source
DeviceSpec("M3", DeviceType.NMOS, width=10.0, length=1.0, multiplier=2)
```

### 2. Layout Strategy

**Use symmetry constraints** for matched devices:
```python
from llmagaical.core.types import SymmetryConstraint

constraints = PlacementConstraints(
    symmetry_pairs=[
        SymmetryConstraint(devices=["M1", "M2"], axis="vertical"),
    ]
)
```

**Specify proximity** for critical connections:
```python
from llmagaical.core.types import ProximityConstraint

constraints.proximity_groups = [
    ProximityConstraint(devices=["M1", "M2"], max_distance=50.0)
]
```

### 3. Verification

Always verify your design:

```python
# Check metrics
print(f"Total area: {result.metrics['total_area']:.2f} µm²")
print(f"Device count: {result.metrics['total_devices']}")
print(f"GDS files: {result.metrics['gds_files']}")

# Verify GDS files exist
from pathlib import Path
gds_dir = Path(output_dir) / "gds"
assert gds_dir.exists()
assert len(list(gds_dir.glob("*.gds"))) > 0
```

---

## 🐛 Troubleshooting

### Common Issues

#### 1. Import Error

```bash
# Error: ModuleNotFoundError: No module named 'llmagaical'
# Solution: Install the package
pip install -e .
```

#### 2. Missing Dependencies

```bash
# Error: No module named 'pydantic'
# Solution: Install dependencies
pip install -r requirements.txt
```

#### 3. C++ Backend Not Available

```bash
# Warning: C++ backend 'anaroute' not available, using Python fallback
# This is normal! Python fallback works perfectly.
# To use C++ backend (optional):
./INSTALL_DEPS.sh
./scripts/build_anaroute.sh
```

#### 4. API Key Issues

```bash
# Error: Missing API key for provider 'anthropic'
# Solution: Set environment variable
export ANTHROPIC_API_KEY="your_key_here"

# Or use mock provider (no API key needed)
llmagaical design "your circuit" --provider mock
```

---

## 📚 Next Steps

### Learn More

1. **README.md** - Project overview and features
2. **SUMMARIZED.md** - Technical architecture details
3. **docs/** - Detailed documentation
4. **examples/** - More example scripts

### Practice Projects

**Beginner**:
1. Design a 3-device current mirror
2. Create an RC filter

**Intermediate**:
3. Build a complete differential amplifier
4. Import and modify a SPICE netlist

**Advanced**:
5. Batch design optimization
6. Add a new device type
7. Create a custom PDK configuration

---

## 🎉 You're Ready!

You now know:
- ✅ How to install and verify LLMagaical
- ✅ How to run examples and view results
- ✅ How to design custom circuits
- ✅ How to configure parameters
- ✅ How to troubleshoot common issues

**Start designing!** 🚀

---

**LLMagaical v3.0.0** - Making IC design accessible through natural language

*Design smarter, not harder.* ✨