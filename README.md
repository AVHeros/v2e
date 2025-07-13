# v2e - Video to Events Converter

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Python torch + opencv code to convert conventional video frames into realistic synthetic DVS (Dynamic Vision Sensor) event streams with high temporal precision. This repository has been modified to be compatible with Cyfronet Athena GPU infrastructure.

![v2e-tennis-example](media/v2e-tennis-split-screen.gif)

## Overview

v2e includes:
- Finite intensity-dependent photoreceptor bandwidth
- Gaussian pixel-to-pixel event threshold variation
- Noise 'leak' events
- SuperSloMo frame interpolation for high temporal resolution

## Installation on Cyfronet AGH Servers (Athena)

### Prerequisites

First, set up your environment directories:

```bash
# Create necessary directories in your PLG storage
export V2E=$PLG_GROUPS_STORAGE/plggwie/plg{username}
mkdir -p $V2E/cache $V2E/config $V2E/tmp

# Export environment variables
export TMPDIR=$V2E/tmp
export XDG_CACHE_HOME=$V2E/cache
export XDG_CONFIG_HOME=$V2E/config
```

### Installation Steps

1. **Load CUDA module:**
   ```bash
   module load CUDA/12.4.0
   ```

2. **Create conda environment:**
   ```bash
   conda create -n v2e python=3.10
   conda activate v2e
   ```

3. **Install PyTorch with CUDA support:**
   ```bash
   pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
   ```

4. **Clone and install v2e:**
   ```bash
   git clone https://github.com/SensorsINI/v2e
   cd v2e
   python -m pip install -e .
   ```

5. **Download SuperSloMo model:**
   Download the pre-trained model checkpoint [SuperSloMo39.ckpt](https://drive.google.com/file/d/1ETID_4xqLpRBrRo1aOT7Yphs3QqWR_fx/view?usp=sharing) (151 MB) and save it to the `input` folder.

### GPU Access with GUI

To access GPU resources with graphical interface:

```bash
# Load visualization module
module load pro-viz

# Start GPU session
pro-viz start -N 1 -t 36:00:00 -M 30GB -g 1 -A plgdyplomanci6-gpu-a100 -p plgrid-gpu-a100

# List active sessions
pro-viz list

# Get password for session
pro-viz password {JobID}

# Stop session when done
pro-viz stop {JobID}
```

## Quick Start

### Basic Usage
```bash
python v2e.py -i input/tennis.mov --overwrite --timestamp_resolution=.003 --auto_timestamp_resolution=False --dvs_exposure duration 0.005 --output_folder=output/tennis --overwrite --pos_thres=.15 --neg_thres=.15 --sigma_thres=0.03 --dvs_aedat2 tennis.aedat --output_width=346 --output_height=260 --stop_time=3 --cutoff_hz=15 
```

### Sample Data
Download sample input videos from [v2e-sample-input-data](https://drive.google.com/drive/folders/1oWxTB9sMPp6UylAdxg5O1ko_GaIXz7wo?usp=sharing).

## Documentation

For detailed usage instructions, command-line options, and advanced features, see:
- [Detailed Usage Guide](docs/detailed_usage.md)
- [Original v2e repository](https://github.com/SensorsINI/v2e)
- [v2e home page](https://sites.google.com/view/video2events/home)

## Performance Notes

- **GPU Recommended**: v2e runs 50-200X slower than real time on GPU, much slower on CPU
- **Memory Requirements**: Use batch sizes 8-16 for SuperSloMo depending on GPU memory
- **Timestamp Resolution**: Higher resolution (e.g., 1ms) requires more computation time
- Use `--stop` option for trial runs before long conversions

## Citation

If you use v2e, please cite:

```bibtex
@inproceedings{hu2021v2e,
  title={v2e: From Video Frames to Realistic DVS Events},
  author={Hu, Yuhuang and Liu, Shih-Chii and Delbruck, Tobi},
  booktitle={2021 IEEE/CVF Conference on Computer Vision and Pattern Recognition Workshops (CVPRW)},
  year={2021},
  url={https://arxiv.org/abs/2006.07722}
}
```

## Contact

- Yuhuang Hu (yuhuang.hu@ini.uzh.ch)
- Tobi Delbruck (tobi@ini.uzh.ch)

## License

MIT License - see [LICENSE](LICENSE) file for details.

---

**Note**: This repository has been modified for compatibility with Cyfronet Athena GPU infrastructure. For the original repository, visit [SensorsINI/v2e](https://github.com/SensorsINI/v2e).
