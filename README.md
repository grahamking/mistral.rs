# mistral.rs
[![Documentation](https://github.com/EricLBuehler/mistral.rs/actions/workflows/docs.yml/badge.svg)](https://ericlbuehler.github.io/mistral.rs/mistralrs/)

Mistral.rs is a LLM inference platform written in pure, safe Rust.

## Upcoming features
- Python bindings

## Description
- Lightweight OpenAI API compatible HTTP server.
- Fast performance with per-sequence and catch-up KV cache management technique.
- Continuous batching.
- First X-LoRA inference platform with first class support.
- 2-bit, 3-bit, 4-bit, 5-bit, 6-bit and 8-bit quantization for faster inference and optimized memory usage.
- Apple silicon support with the Metal framework.

**Supported models:**
- Mistral 7B
- Gemma
- Llama
- Mixtral 8x7B

|Model|GGUF|GGML|X-LoRA|X-LoRA+GGUF|X-LoRA+GGML|
|--|--|--|--|--|--|
|Mistral 7B |✅| |✅|✅| |
|Gemma| | |✅| | |
|Llama|✅|✅|✅|✅|✅|
|Mixtral 8x7B|✅| |✅|✅|✅|

Please note that when using a derivative model with a quantized architecture, it is important to specify the corresponding model ID for the tokenizer with `-t`:

`./mistralrs-server --port 1234 --log output.txt mistral-gguf -t HuggingFaceH4/zephyr-7b-beta -m TheBloke/zephyr-7B-beta-GGUF -f zephyr-7b-beta.Q5_0.gguf`

Failing to do so will result in a incorrect prompt templating.

**Library API**
- Rust multithreaded API for easy integration into any application: [docs](https://ericlbuehler.github.io/mistral.rs/mistralrs/). To use, add `mistralrs = { git = "https://github.com/EricLBuehler/mistral.rs.git" }` to the Cargo.toml.

**HTTP Server**
Mistral.rs provides an OpenAI API compatible API server. It is accessible through the command line when one builds mistral.rs.

Request docs can be found [here](https://ericlbuehler.github.io/mistral.rs/mistralrs_server/openai/index.html), and response docs 
[here](https://ericlbuehler.github.io/mistral.rs/mistralrs_core/index.html).

## Usage
### Build
To build mistral.rs, one should ensure they have Rust installed by following [this](https://rustup.rs/) link.
The Huggingface token should be provided in `~/.cache/huggingface/token`. 
- Using a script

    For an easy quickstart, the script below will 
    download an setup Rust and then build mistral.rs to run on the CPU.
    ```bash
    sudo apt update -y
    sudo apt install libssl-dev -y
    sudo apt install pkg-config -y
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
    source $HOME/.cargo/env

    git clone https://github.com/EricLBuehler/mistral.rs.git
    cd mistral.rs
    mkdir ~/.cache/huggingface
    touch ~/.cache/huggingface/token
    echo <HF_TOKEN_HERE> > ~/.cache/huggingface/token
    cargo build --release
    ```
- Manual build

    If Rust is installed and the Huggingface token is set, then one may build mistral.rs by executing the build command.
    `cargo build --release`.
The build process will output a binary `misralrs-server` at `./target/release/mistralrs-server`.
### Building for GPU, Metal or enabling other features
Rust uses a feature flag system during build to implement compile-time build options. As such, the following is a list of features
which may be specified using the `--features` command.
1) `cuda`
2) `metal`
3) `flash-attn`: Be sure to pass `--use_flash_attn`

### X-LoRA
**Preparing the X-LoRA Ordering File**
The X-LoRA ordering JSON file contains 2 parts. The first is the order of the adapters and the second, the layer ordering. The layer ordering has been automatically generated and should not be manipulated as it controls the application of scalings. However the order of adapter should be replaced by an array of strings of adapter names corresponding to the order the adapters were specified during training.

---

**Quantized X-LoRA models**
Mistral.rs supports running quantized models with X-LoRA. The X-LoRA layers will not be quantized, only the base model. Please note that
using a high quantization level (eg., 4-bit) can distort the signal and prevent the classifier from acting properly. Therefore, it is better to use slightly higher levels such as 8-bit.

**Supported X-LoRA quantized layers**
- model.layers.{layer_idx}.self_attn.q_proj
- model.layers.{layer_idx}.self_attn.k_proj
- model.layers.{layer_idx}.self_attn.v_proj
- model.layers.{layer_idx}.self_attn.o_proj
- model.layers.{layer_idx}.mlp.up_proj
- model.layers.{layer_idx}.mlp.down_proj
- model.layers.{layer_idx}.mlp.gate_proj

## Run

To start a server serving Mistral on `localhost:1234`, 
```bash
./mistralrs-server --port 1234 --log output.log mistral
```

Mistral.rs uses subcommands to control the model type. Please run `./mistralrs-server --help` to see the subcommands.

To start an X-LoRA server with the default weights, run the following after modifying or copying the ordering file as described [here](README.md#preparing-the-x-lora-ordering-file).

`./mistralrs-server --port 1234 x-lora-mistral -o ordering.json`

## Benchmarks
For the prompt "Tell me about the Rust type system in depth." and a maximum length of 256.

**A6000** Mistral + CUDA + FlashAttention
- 30.44 tok/s

**A6000** Mistral GGUF + CUDA
- 39.3 tok/s