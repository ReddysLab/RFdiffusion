# RFdiffusion Docker Setup Instructions

## Prerequisites

### 1. Install Docker
If Docker is not installed, run:
```bash
# Update package manager
sudo apt update

# Install Docker
sudo apt install -y docker.io

# Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to the docker group (requires logout/login or restart)
sudo usermod -aG docker $USER
```

### 2. Install NVIDIA Container Toolkit
For GPU support, you need the NVIDIA Container Toolkit:
```bash
# Add NVIDIA GPG key and repository
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

# Update and install nvidia-docker2
sudo apt update
sudo apt install -y nvidia-docker2

# Restart Docker daemon
sudo systemctl restart docker
```

### 3. Verify GPU Access
Test that Docker can access your GPUs:
```bash
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

## Setup and Usage

### 1. Clone and Build
```bash
git clone https://github.com/ReddysLab/RFdiffusion.git
cd RFdiffusion
docker build -f docker/Dockerfile -t rfdiffusion .
```

### 2. Download Models and Prepare Directories
```bash
# Create required directories
mkdir $(pwd)/inputs $(pwd)/outputs $(pwd)/models

# Download RFdiffusion models
bash scripts/download_models.sh $(pwd)/models

# Download example PDB file
wget -P $(pwd)/inputs https://files.rcsb.org/view/5TPN.pdb
```

### 3. Basic Usage Example
```bash
# Run RFdiffusion for motif scaffolding
docker run -it --rm --gpus all \
  -v $(pwd)/models:$HOME/models \
  -v $(pwd)/inputs:$HOME/inputs \
  -v $(pwd)/outputs:$HOME/outputs \
  rfdiffusion \
  inference.output_prefix=$HOME/outputs/motifscaffolding \
  inference.model_directory_path=$HOME/models \
  inference.input_pdb=$HOME/inputs/5TPN.pdb \
  inference.num_designs=3 \
  'contigmap.contigs=[10-40/A163-181/10-40]'
```

### 4. Advanced Example: Cyclic Contigs
For more complex protein design with cyclic contigs:
```bash
# Define variables for cyclic design
prefix=$HOME/outputs/diffused_binder_cyclic2
pdb=$HOME/examples/input_pdbs/7zkr_GABARAP.pdb
num_designs=10

# Run the container using specific GPU (device=0)
docker run -it --rm --gpus '"device=0"' \
  -v $(pwd)/scripts:/app/RFdiffusion/scripts \
  -v $(pwd)/examples:$HOME/examples \
  -v $(pwd)/inputs:$HOME/inputs \
  -v $(pwd)/outputs:$HOME/outputs \
  -v $(pwd)/models:$HOME/models \
  rfdiffusion \
  --config-name base \
  inference.output_prefix=$prefix \
  inference.num_designs=$num_designs \
  inference.model_directory_path=$HOME/models \
  'contigmap.contigs=[12-18 A3-117/0]' \
  inference.input_pdb=$pdb \
  inference.cyclic=True \
  diffuser.T=50 \
  inference.cyc_chains='a' \
  "ppi.hotspot_res=['A51','A52','A50','A48','A62','A65']"
```

## Troubleshooting

### Common Issues

1. **Permission denied errors**: Make sure you've added your user to the docker group and logged out/in or restarted.

2. **GPU not detected**: Verify NVIDIA drivers are installed and run `nvidia-smi` to check GPU status.

3. **Docker daemon not running**: Start with `sudo systemctl start docker`.

4. **Out of memory**: Reduce `num_designs` or use specific GPU with `--gpus '"device=X"'` instead of `--gpus all`.

