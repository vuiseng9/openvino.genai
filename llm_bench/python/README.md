# Benchmarking script for large language models

This script provides unified approach to estimate performance for LLM. It is based on pipelines provided by Optimum-Intel and allows to estimate performance for pytorch and openvino models using almost the same code and precollected models.

## Usage
(1) Need to start the virtual environment of python
``` bash
python3 -m venv python-env
source python-env/bin/activate
pip install update --upgrade
pip install -r requirements.txt
```
(2) Convert model to IRs
## Conversion script for preparing benchmarking models

`convert.py` allow to reproduce IRs stored on shared drive.

Prerequisites:
install conversion dependencies using requirements.txt

Usage:

```bash
python convert.py --model_id <model_id_or_path> --output_dir <out_dir>
```

Paramters:
* `--model_id` - model_id for downloading from huggngface_hub (https://huggingface.co/models) or path with directory where pytorch model located.
* `--output_dir` - output directory for saving OpenVINO model
* `--precision` - (optional, default FP32), precision for model conversion FP32 or FP16
* `--save_orig` - flag for saving original pytorch model, model will be located in `<output_dir>/pytorch` subdirectory.
* `--compress_weights` - flag for saving model with compressed weights.
* `--compress_weights_backends` - (optional, default openvino) backends for weights compression, this option has an effect only with `--compress_weights`. For `openvino` backend, model will be located in `<output_dir>/pytorch/dldt/INT8_compressed_weights` subdirectory. For `pytorch` backend, model will be located in `<output_dir>/pytorch/dldt/PT_compressed_weights` subdirectory. You can specify multiple backends separated by a space.

Usage example:
```bash
python convert.py --model_id meta-llama/Llama-2-7b-chat-hf --output_dir models/llama-2-7b-chat
```

the result of running command will have following file structure:

    |-llama-2-7b-chat
      |-pytorch
        |-dldt
           |-FP32
              |-openvino_model.xml
              |-openvino_model.bin
              |-config.json
              |-added_tokens.json
              |-tokenizer_config.json
              |-tokenizer.json
              |-tokenizer.model
              |-special_tokens_map.json

(3) Bechmarking
Prerequisites:
install benchmarking dependencies using requirements.txt

``` bash
pip install -r requirements.txt
```
notes: **You can specify the installed openvino version through pip install**
``` bash
# e.g. 
pip install openvino==2023.2.0
```

(4) Run the following command to test the performance of one LLM model
``` bash
python benchmark.py -m <model> -d <device> -r <report_csv> -f <framework> -p <prompt text> -n <num_iters>
```x
# e.g.
python benchmark.py -m models/llama-2-7b-chat/pytorch/dldt/FP32 -n 2
python benchmark.py -m models/llama-2-7b-chat/pytorch/dldt/FP32 -p "What is openvino?" -n 2
python benchmark.py -m models/llama-2-7b-chat/pytorch/dldt/FP32 -pf prompts/llama-2-7b-chat_l.jsonl -n 2
```
Parameters:
* `-m` - model path
* `-d` - inference device (default=cpu)
* `-r` - report csv
* `-f` - framework (default=ov)
* `-p` - interactive prompt text
* `-pf` - path of JSONL file including interactive prompts
* `-n` - number of benchmarking iterations, if the value greater 0, will exclude the first iteration. (default=0)

``` bash
python ./benchmark.py -h # for more information
```

## Running `torch.compile()`

The option `--torch_compile_backend` uses `torch.compile()` to speed up the PyTorch code by compiling it into optimized kernels using a selected backend.

Prerequisites: install benchmarking dependencies using requirements.txt

``` bash
pip install -r requirements/requirements.txt
```

In order to run the `torch.compile()` on CUDA GPU, install additionally the nightly PyTorch version:

```bash
pip install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/cu118
```

Add the option `--torch_compile_backend` with the desired backend: `pytorch` or `openvino` (default) while running the benchmarking script:

```bash
python ./benchmark.py -m models/llama-2-7b-chat/pytorch -d CPU --torch_compile_backend openvino
```