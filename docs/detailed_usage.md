# Detailed v2e Usage Guide

## Render emulated DVS events from conventional video

_v2e.py_ reads a standard video (e.g. in .avi, .mp4, .mov, or .wmv), or a folder of images, and generates emulated DVS events at upsampled timestamp resolution.

Don't be intimidated by the huge number of options. Running _v2e.py_ with no arguments sets reasonable values and opens a file browser to let you select an input video. Inspect the logging output for hints.

**Hint:** Note the options _[--dvs128 | --dvs240 | --dvs346 | --dvs640 | --dvs1024]_; they set output size and width to popular DVS cameras.

**On headless platforms**, with no graphics output, use --no_preview option to suppress the OpenCV windows.

## Command Line Options

```
usage: v2e.py [-h] [-o OUTPUT_FOLDER] [--avi_frame_rate AVI_FRAME_RATE]
              [--output_in_place [OUTPUT_IN_PLACE]] [--overwrite]
              [--unique_output_folder [UNIQUE_OUTPUT_FOLDER]]
              [--skip_video_output]
              [--auto_timestamp_resolution [AUTO_TIMESTAMP_RESOLUTION]]
              [--timestamp_resolution TIMESTAMP_RESOLUTION]
              [--dvs_params DVS_PARAMS] [--pos_thres POS_THRES]
              [--neg_thres NEG_THRES] [--sigma_thres SIGMA_THRES]
              [--cutoff_hz CUTOFF_HZ] [--leak_rate_hz LEAK_RATE_HZ]
              [--shot_noise_rate_hz SHOT_NOISE_RATE_HZ]
              [--photoreceptor_noise]
              [--leak_jitter_fraction LEAK_JITTER_FRACTION]
              [--noise_rate_cov_decades NOISE_RATE_COV_DECADES]
              [--refractory_period REFRACTORY_PERIOD]
              [--dvs_emulator_seed DVS_EMULATOR_SEED]
              [--show_dvs_model_state SHOW_DVS_MODEL_STATE [SHOW_DVS_MODEL_STATE ...]]
              [--save_dvs_model_state]
              [--record_single_pixel_states RECORD_SINGLE_PIXEL_STATES]
              [--output_height OUTPUT_HEIGHT] [--output_width OUTPUT_WIDTH]
              [--dvs128 | --dvs240 | --dvs346 | --dvs640 | --dvs1024]
              [--disable_slomo] [--slomo_model SLOMO_MODEL]
              [--batch_size BATCH_SIZE] [--vid_orig VID_ORIG]
              [--vid_slomo VID_SLOMO] [--slomo_stats_plot] [-i INPUT]
              [--input_frame_rate INPUT_FRAME_RATE]
              [--input_slowmotion_factor INPUT_SLOWMOTION_FACTOR]
              [--start_time START_TIME] [--stop_time STOP_TIME] [--crop CROP]
              [--hdr] [--synthetic_input SYNTHETIC_INPUT]
              [--dvs_exposure DVS_EXPOSURE [DVS_EXPOSURE ...]]
              [--dvs_vid DVS_VID] [--dvs_vid_full_scale DVS_VID_FULL_SCALE]
              [--no_preview] [--ddd_output] [--dvs_h5 DVS_H5]
              [--dvs_aedat2 DVS_AEDAT2] [--dvs_text DVS_TEXT]
              [--label_signal_noise] [--cs_lambda_pixels CS_LAMBDA_PIXELS]
              [--cs_tau_p_ms CS_TAU_P_MS] [--scidvs]
```

### Key Parameters

#### Output Options
- `-o OUTPUT_FOLDER, --output_folder OUTPUT_FOLDER`: folder to store outputs
- `--avi_frame_rate AVI_FRAME_RATE`: frame rate of output AVI video files
- `--overwrite`: overwrites files in existing folder
- `--skip_video_output`: Skip producing video outputs

#### DVS Model Parameters
- `--dvs_params DVS_PARAMS`: Easy optional setting ('clean', 'noisy', or None)
- `--pos_thres POS_THRES`: threshold for positive events (default: log_e intensity change)
- `--neg_thres NEG_THRES`: threshold for negative events
- `--sigma_thres SIGMA_THRES`: threshold variation
- `--cutoff_hz CUTOFF_HZ`: photoreceptor lowpass filter cutoff frequency
- `--leak_rate_hz LEAK_RATE_HZ`: leak event rate per pixel in Hz
- `--shot_noise_rate_hz SHOT_NOISE_RATE_HZ`: temporal noise rate

#### Camera Sizes
- `--dvs128`: Set size for 128x128 DVS
- `--dvs240`: Set size for 240x180 DVS (DAVIS240)
- `--dvs346`: Set size for 346x260 DVS (DAVIS346)
- `--dvs640`: Set size for 640x480 DVS (DAVIS640)
- `--dvs1024`: Set size for 1024x768 DVS

#### SloMo Upsampling
- `--disable_slomo`: Disables slomo interpolation
- `--slomo_model SLOMO_MODEL`: path of slomo_model checkpoint
- `--batch_size BATCH_SIZE`: Batch size for SuperSloMo (8-16 recommended)
- `--timestamp_resolution TIMESTAMP_RESOLUTION`: Desired DVS timestamp resolution in seconds

## Example Usage

### Basic conversion
```bash
python v2e.py -i input/tennis.mov -o output/tennis --dvs346
```

### With custom parameters
```bash
python v2e.py -i input/video.mp4 -o output/events \
    --dvs346 \
    --timestamp_resolution 0.001 \
    --pos_thres 0.2 \
    --neg_thres 0.2 \
    --dvs_params clean
```

### Synthetic input generation
```bash
python v2e.py --synthetic_input moving_dot \
    --dvs346 \
    -o output/synthetic
```

## Performance Notes

- We recommend running _v2e_ on a CUDA GPU or it will be very slow, particularly when using SuperSloMo upsampling
- Even with a low-end GTX-1050, _v2e_ runs about 50-200X slower than real time using 10X slowdown factor and 346x260 video
- Conversion speed depends linearly on the reciprocal of the desired DVS timestamp resolution
- Use the _--stop_ option for trial runs before starting long conversions

## Output Formats

v2e can output events in several formats:
- HDF5 (.h5) - recommended for large datasets
- AEDAT2 (.aedat) - compatible with jAER
- Text (.txt) - human readable
- DDD format - for specific datasets

## Advanced Features

### Noise Modeling
v2e includes several noise sources:
- Shot noise: `--shot_noise_rate_hz`
- Leak events: `--leak_rate_hz`
- Photoreceptor noise: `--photoreceptor_noise`
- Threshold variation: `--sigma_thres`

### State Recording
You can record and visualize internal DVS model states:
```bash
python v2e.py --show_dvs_model_state all --save_dvs_model_state
```

### Single Pixel Analysis
Record states of a specific pixel:
```bash
python v2e.py --record_single_pixel_states 100,100
```

For more detailed information, see the [v2e home page](https://sites.google.com/view/video2events/home) and the original [GitHub repository](https://github.com/SensorsINI/v2e).