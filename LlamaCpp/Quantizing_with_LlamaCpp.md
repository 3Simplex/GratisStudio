ThiloteE — 4/2/2024  
3Simplex - 4/4/2024 (modified for video, included: PreReqs, coreinfo, syntax for powershell and cmd.)  
# Notes for Quantizing with llama.cpp documentation:

### Reqirements to follow video.
QT creator (CMake) "https://github.com/nomic-ai/gpt4all/blob/main/gpt4all-chat/build_and_run.md"  
Vulkan SDK (AMD GPU Support) "https://vulkan.lunarg.com/sdk/home"  
LlaMA.Cpp (Quantization) "https://github.com/ggerganov/llama.cpp"  
LLM files (Used in video) "https://huggingface.co/WhiteRabbitNeo/Trinity-13B/tree/main"  
GPT4All (LLM environment) Easy Installer "https://gpt4all.io/index.html" or compiled "https://github.com/nomic-ai/gpt4all/blob/main/gpt4all-chat/build_and_run.md"  

### CPU instruction sets Powershell command.
Find the instruction sets compatible with your CPU, open powershell and enter this script.  
Look for the asterisk * to denote the instruction sets your CPU has.  

	Invoke-WebRequest -Uri 'https://download.sysinternals.com/files/Coreinfo.zip' -OutFile 'Coreinfo.zip'
	Expand-Archive -Path 'Coreinfo.zip' -DestinationPath . -Force
	.\coreinfo.exe


### Short Example:
	cd .\llama.cpp
    python convert.py models\mymodel\pytorch_model.bin --outtype f16

    .\quantize.exe ggml-model-f16.gguf ggml-model-Q4_0.gguf Q4_0

	
### Long Guide:


This guide was derrived from https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#prepare-and-quantize

## Place the model and configuration files into the correct folder

### Git Clone and place them in the models folder (within the llama.cpp directory)
	cd .\llama.cpp\models
	git clone [URL]
	
### [Optional] for models using BPE tokenizers
	cd .\llama.cpp\models
	<folder containing weights and tokenizer json> vocab.json
	
### [Optional] for PyTorch .bin models like Mistral-7B
	cd .\llama.cpp\models
	<folder containing weights and tokenizer json>


## Install Python dependencies
Enter the llama.cpp folder with administrator command prompt.  
	cd .\llama.cpp

## [Suggested] Install Python dependencies with python virtual environment
	
	Create a virtual environment
	python -m venv venv
		
	Activate the Virtual environment
	.\venv\Scripts\activate.ps1 (using powershell)
	.\venv\Scripts\activate (using cmd)

	Install Python dependencies within the virtual environment
	python -m pip install -r requirements.txt

## Carry out the initial conversion to gguf
While in python VENV enter \Llama.cpp directory.

	Verify you are in the correct folder, the convert script is there and understand the various options:
	python convert.py --help

	Convert the model to ggml FP16 format
	python convert.py models\mymodel\ --outtype f16

	[Optional] for models Byte-Pair Encoding (BPE) tokenizers
	python convert.py models\mymodel\ --vocab-type bpe

## Carry out the quantization in the build-llama.cpp directory.
This is not done in python VENV.

Prerequisites:  
- Build llama.cpp as explained at https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#build  
- Open a new commandline in the bin folder of the build directory.  
- Put the model to be quantized into the folder containing quantize.exe  

Verify you are in the correct folder and understand various options  
.\quantize.exe --help

quantize the model to 4-bits (using Q4_0 method)  
.\quantize.exe ggml-model-f16.gguf ggml-model-Q4_0.gguf Q4_0

[Optional] to update the gguf to current version if older version is now unsupported  
.\quantize.exe ggml-model-Q4_0.gguf ggml-model-Q4_0-v2.gguf COPY
