ThiloteE — 4/2/2024  
3Simplex - 4/4/2024 (modified for video, included: PreReqs, coreinfo, syntax for powershell and cmd.)  
# Notes for Quantizing with llama.cpp documentation:

This guide was derived from https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#prepare-and-quantize


### Reqirements to follow video.
Git (git clone)  
https://git-scm.com/  
LlaMA.Cpp (Quantization)  
https://github.com/ggerganov/llama.cpp  
LLM files (Used in video)  
https://huggingface.co/WhiteRabbitNeo/Trinity-13B/tree/main  
Vulkan SDK (AMD GPU Support)  
https://vulkan.lunarg.com/sdk/home  
QT creator (CMake)  
https://www.qt.io/download-qt-installer-oss  
GPT4All (LLM environment)  
- Easy Installer https://gpt4all.io/index.html  
- Instructions for Compiled version https://github.com/nomic-ai/gpt4all/blob/main/gpt4all-chat/build_and_run.md  

## Clone Llama.cpp into your home directory.
```
git clone https://github.com/ggerganov/llama.cpp
```

## Build Llama.cpp:  
- Preinstall Vulkan (if using AMD)
- Preinstall Cuda (if using Nvidia)
- Build llama.cpp with your prefered method as explained at https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#build
	- We used cmake with QT creator.
 		- Selected applicable CPU instruction sets (AVX, AVX2)
   		- Selected Vulkan for AMD GPU
### CPU instruction sets Powershell command.
Find the instruction sets compatible with your CPU, open powershell and enter this script.  
Look for the asterisk * to denote the instruction sets your CPU has.  

	Invoke-WebRequest -Uri 'https://download.sysinternals.com/files/Coreinfo.zip' -OutFile 'Coreinfo.zip'
	Expand-Archive -Path 'Coreinfo.zip' -DestinationPath . -Force
	.\coreinfo.exe


## Place the model and configuration files into the correct folder.

### Git Clone and place them in the models folder. (within the llama.cpp directory)
	cd .\llama.cpp\models
	git clone [URL]

## Install Python dependencies with python virtual environment.
### Enter the llama.cpp folder with administrator command prompt.
	cd .\llama.cpp
	
Create a virtual environment.
```
python -m venv venv
```		
Activate the Virtual environment.
```
.\venv\Scripts\activate
```
Install Python dependencies within the virtual environment.
```
python -m pip install -r requirements.txt
```
## Do the initial conversion to gguf.
While in python VENV enter \Llama.cpp directory.

Verify you are in the correct folder, the convert script is there and understand the various options:
```
python convert.py --help
```
Convert the model to ggml FP16 format.
```
python convert.py models\mymodel\ --outtype f16
```
[Optional] for models Byte-Pair Encoding (BPE) tokenizers. (Llama 3 uses BPE)
```
python convert.py models\mymodel\ --vocab-type bpe
```
## Do the quantization in the build-llama.cpp directory.
This is not done in python VENV.
- Open a new commandline in the bin folder of the build directory.  
- Put the model to be quantized into the folder containing quantize.exe  
	
Verify you are in the correct folder and understand various options.  
```
.\quantize.exe --help
```	
Quantize the model to 4-bits. (using Q4_0 method)  
```
.\quantize.exe ggml-model-f16.gguf ggml-model-Q4_0.gguf Q4_0
```	
[Optional] to update the gguf to current version if older version is now unsupported.  
```
.\quantize.exe ggml-model-Q4_0.gguf ggml-model-Q4_0-v2.gguf COPY
```
  
---
Put the new quantized model in your LLM environment directory, run the environment, configure it and enjoy!  
