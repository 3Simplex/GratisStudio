ThiloteE — 4/2/2024 (Original notes, guided me through his process.)  
3Simplex - 4/4/2024 (Step by step walkthrough with all requirements, syntax for powershell method with detailed explainations.)  
# Guide for Quantizing with llama.cpp (updated 8/8/2024)

The following guide was written for Windows users, which was derived from [https://github.com/ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp?tab=readme-ov-file#description)  
There you will find detailed information for users of all supported Operating systems.  
Linux users may be interested in an adaptation by rizitis - 5/11/2024 (Linux script https://github.com/rizitis/Quantizing_with_LlamaCpp)

### Reqirements to follow guide.
- Git (git clone)  
https://git-scm.com/  

- Python (tools)
	- CMake (build llama.cpp)  
	https://pypi.org/project/cmake/  
	- pyenv-win (safely install and use python versions with venv)  
	https://github.com/pyenv-win/pyenv-win  

- LlaMA.Cpp (Quantization)  
https://github.com/ggerganov/llama.cpp  

- LLM files (Used in video)  
https://huggingface.co/NousResearch/Meta-Llama-3-8B-Instruct  

- Vulkan SDK (AMD GPU Support)  
https://vulkan.lunarg.com/sdk/home  
Windows download link - https://sdk.lunarg.com/sdk/download/latest/windows/vulkan-sdk.exe  

- Cuda toolkit (Nvidia GPU Support)  
https://developer.nvidia.com/cuda-downloads  

- GPT4All (LLM environment)  
Easy Installer https://gpt4all.io/index.html  (minutes)  
Instructions for self compiled version https://github.com/nomic-ai/gpt4all/blob/main/gpt4all-chat/build_and_run.md  (hours/days)  
## Install git (if not already installed)
Using powershell (the easy way)  
```
winget install --id Git.Git -e --source winget
```
## Install pyvenv (if not already installed)  
This is an easy and useful python managment tool which will save your sanity.  
```
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```
- Configure pyenv.  
	Check a list of Python versions available.  
	```
	pyenv install -l 
	```
	Install python 3.11 (for this project)  
	```
	pyenv install 3.11
	```
	Set a Python version as the system wide default (Best practice, if python was already installed set that to global)
	```  
	pyenv global <your prefered version>  
	```
	Whenever you need a new version of python for a specific project, you will set that project directory with a local version.
	```
	pyenv local <required local version>
	```
	Always "rehash" after changes, like installing a new python or running pip. (keeps your Python environment in sync)
	```
	pyenv rehash
	```
## Install CMake (if not already installed)
```
pip install cmake
pyenv rehash
```   
## Install a GPU sdk (if not already installed)
Install Vulkan (if using AMD) https://sdk.lunarg.com/sdk/download/latest/windows/vulkan-sdk.exe  
Install Cuda (if using Nvidia) https://developer.nvidia.com/cuda-downloads  

## Clone Llama.cpp into your home directory.
While in powershell enter the home directory (the easy way)  
```
cd ~
```
Clone the repo.  
```
git clone --recurse-submodules https://github.com/ggerganov/llama.cpp
```
[Optional] How to update the repo to get changes made after cloning. (always build after updating)
```
cd ~\llama.cpp
git pull origin
```  
### CPU instruction sets Powershell command.
To find the instruction sets compatible with your CPU. (GPT4All uses AVX/AVX2)  
Look for the asterisk * to denote the instruction sets your CPU has.  
```
cd ~
mkdir coreinfo
cd ~\coreinfo\
Invoke-WebRequest -Uri 'https://download.sysinternals.com/files/Coreinfo.zip' -OutFile 'Coreinfo.zip'
Expand-Archive -Path 'Coreinfo.zip' -DestinationPath . -Force
.\coreinfo.exe
```
## Build Llama.cpp:  (build each time you update the repo)
- Build llama.cpp  
	Auto select applicable CPU instruction sets (AVX, AVX2)  
	This flag will only use your native instructions "-DGGML_NATIVE=ON" (not recomended if you intend to use the output on another computer)  
	This flag will set the number of build jobs to use, ``-j #`` set this to the number of cores your CPU has.  
	- ``[Environment]::ProcessorCount / 2`` (powershell command to find number of cores)  

	Use Vulkan for AMD GPU
	```
	cd ~\llama.cpp
	mkdir build
	cd .\build
	cmake .. -DGGML_VULKAN=ON -DGGML_NATIVE=ON
	cmake --build . --config Release -j 8
	```
	or  
	
	Use Cuda for Nvidia GPU
	```
	cd ~\llama.cpp
	mkdir build
	cd .\build
	cmake .. -DGGML_CUDA=ON -DGGML_NATIVE=ON
	cmake --build . --config Release -j 8
	```  
	or  
	
	Use CPU without GPU
	```
	cd ~\llama.cpp
	mkdir build
	cd .\build
	cmake .. -DGGML_NATIVE=ON
	cmake --build . --config Release -j 8
	```  
## Place the model and configuration files into the correct folder.
### Git Clone into the models folder. (or move them there)
	cd ~\llama.cpp\models
	git clone [URL]  
[Optional] How to update model if you used git clone. (To get changes only)
List the model directory you previously cloned.
```
ls ~\llama.cpp\models
```
Enter that directory then pull changes.
```
cd ~\llama.cpp\models\model_dir
git pull origin
```
## Install Python dependencies with python virtual environment.
Enter the llama.cpp folder.  
```
cd ~\llama.cpp  
```
Set local python environment. (run only the first time)  
```
pyenv local 3.11
```
Create a virtual environment. (run only the first time)
```
python -m venv venv
```  
Activate the Virtual environment. (use this 'venv' from now on)
```
.\venv\Scripts\activate
```
Install Python dependencies within the virtual environment. (run only the first time)
```
python -m pip install -r requirements.txt
pyenv rehash
```
## Conversion to gguf.
While in python VENV enter \Llama.cpp directory.  
```
cd ~\llama.cpp
```
Available convert scripts...  
- [Recomended] convert_hf_to_gguf.py handles most models. (Huggingface)
- convert_legacy_llama.py is intended for llama 1-2 and mistral only.
- convert_llama_ggml_to_gguf.py converts old .ggml llama models.

Show the model directories you have now. You will replace `model_dir` with a name in this list. (Models recieved via git clone or download)  
```
Get-ChildItem -Path ".\models" | Where-Object { $_.Attributes -eq "Directory" }
``` 
Convert the model will default to FP16 which preserves the model while reducing size. (using the recomended script)  
```
python convert_hf_to_gguf.py models\model_dir\
```
- For llama 1-2 and mistral only ```python examples\convert_legacy_llama.py models\model_dir\```
- For models using Byte-Pair Encoding (BPE) tokenizers add the folowing tag. ```--vocab-type bpe```
- For ggml llama models ```python convert_llama_ggml_to_gguf.py models\model_dir\```

## Do the quantization in the llama.cpp\build\bin\Release directory.
Quantization is not done in python VENV. ('deactivate' the venv)  
List the models in the model directory (copy the name for the `model_dir`)  
```
deactivate
Get-ChildItem -Path ".\models" | Where-Object { $_.Attributes -eq "Directory" }
```
Put the model to be quantized into the folder containing quantize.exe (change `model_dir` name as needed, move the new guff)  
```
mv ~\llama.cpp\models\model_dir\ggml-model-f16.gguf ~\llama.cpp\build\bin\Release\ggml-model-f16.gguf
```
Change to the bin directory where llama-quantize.exe is at. (cd to the \build\bin\Release)  
```
cd ~\llama.cpp\build\bin\Release
```
Verify you are in the correct folder and understand various options.  
```
.\llama-quantize.exe --help
```	
Quantize the model to Q4_0. (Vulkan GPU support in GPT4All uses Q4_0 or Q4_1, CUDA works with any quantization.)  
```
.\llama-quantize.exe ggml-model-f16.gguf ggml-model-Q4_0.gguf Q4_0
```	
[Optional] to update the gguf to current version if older version is unsupported.  
```
.\llama-quantize.exe ggml-model-Q4_0.gguf ggml-model-Q4_0-v2.gguf COPY
```
Rename ggml-model-Q4_0.gguf (Copy `model_dir` name)  
```
ls ~\llama.cpp\models  
ren ggml-model-Q4_0.gguf  PASTE_NEW_NAME-Q4_0.gguf
```
---
Put the new quantized model in your LLM environment directory, run the environment, configure it and enjoy!  
