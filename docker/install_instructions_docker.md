# Usage: 
git clone https://github.com/ReddysLab/RFdiffusion.git
cd RFdiffusion
docker build -f docker/Dockerfile -t rfdiffusion .

# Download models and example inputs in RFdiffusion directory
mkdir $(pwd)/inputs $(pwd)/outputs $(pwd)/models
bash scripts/download_models.sh $(pwd)/models
wget -P $(pwd)/inputs https://files.rcsb.org/view/5TPN.pdb

# Run docker (assumes you are in RFdiffusion directory as pwd)
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

# Example of running RFdiffusion with cyclic contigs
# This example uses GPU 0 and assumes you have a PDB file at the specified path
# Adjust the paths and parameters as needed

```bash
# Ensure you have a PDB file at the specified path
# Define variables
prefix=$HOME/outputs/diffused_binder_cyclic2
pdb=$HOME/examples/input_pdbs/7zkr_GABARAP.pdb
num_designs=10

# Run the container using GPU 0
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
