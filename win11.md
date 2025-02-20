# Windows 11 Setup

## Using System Python (CPU Only)
1. Download and install [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) and make sure **Desktop development with C++** workload is selected and installed. This is needed to build `chroma-hnswlib` (as of this writing a pre-built wheel only exists for Python 3.11 and below). It is also needed if you need to build `llama-cpp-python` instead of installing a prebuilt wheel (as we do below). If you have issues, you might also try downloading and installing the [Microsoft Visual C++ Redistributable](https://aka.ms/vs/17/release/vc_redist.x64.exe) to ensure `onnxruntime` can be imported, as described in [this issue](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/16342#discussioncomment-10279473). However the Redistributable should be automatically installed with Build Tools.
2. Install Python 3.12:  Open "cmd" as administrator and type python to trigger Microsoft Store installation in Windows 11.
3. Create virtual environment: `python -m venv .venv`
4. Activate virtual environment: `.venv\Scripts\activate`. You can optionally append `C:\Users\<username\.venv\Scripts` to `Path` environment variable, so that you only need to type `activate` to enter virtual environment in the future.
5. Install PyTorch:
   - For CPU: `pip install torch torchvision torchaudio`
   - For GPU (if you do happen to have installed an NVIDIA driver): `pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124`
     - Run the following at Python prompt to verify things are working. (The PyTorch binaries ship with all CUDA runtime dependencies and you don't need to locally install a CUDA toolkit or cuDNN, as long as NVIDIA driver is installed.)
	   ```python
	   In [1]: import torch
	
	   In [2]: torch.cuda.is_available()
	   Out[2]: True
	
	   In [3]: torch.cuda.get_device_name()
	   Out[3]: 'NVIDIA RTX A1000 6GB Laptop GPU'
	   ```
7. Install llama-cpp-python using pre-built wheel: `pip install llama-cpp-python==0.2.90 --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu`
8. Install OnPrem.LLM: `pip install onprem `
11. [OPTIONAL] Add REQUESTS_BUNDLE to environment variable and point it to certs for your organization if behind a corporate, so hugging face models can be downloaded. Without this steup, you will need to use the `--trusted-host` option
12. [OPTIONAL] Enable long paths if you get an error indicating you do:  https://stackoverflow.com/questions/72352528/how-to-fix-winerror-206-the-filename-or-extension-is-too-long-error/76452218#76452218
13. Try onprem to make sure it works:
     ```python
     from onprem import LLM
     llm = LLM()
     llm.prompt('List three cute names for a cat.')
     ```

## Using uv instead of System Python (CPU Only)
1. Install Python 3.12:  Open "cmd" as adminstrator and type python to trigger installation prompt from Windows 11. Needed to install `uv`.
2. Open new `cmd` (not as Administrator) and run `pip install uv`
3. Add to `PATH` environment varialbe (for ipython, uv, etc.):  `C:\Users\amaiya\AppData\Local\Packages\PythonSoftwareFoundation.Python.3.12_qbz5n2kfra8p0\LocalCache\local-packages\Python312\Scripts`
4. Enable long paths:  https://stackoverflow.com/questions/72352528/how-to-fix-winerror-206-the-filename-or-extension-is-too-long-error/76452218#76452218
5. Run `uv venv --python 3.11 --seed --trusted-host github.com` # using 3.11 because prebuilt wheel 3.12 files for `chroma-hnswlib` do not yet exist as of this writing
6. Add to `Path` environment variable (and to BEFORE the entry aded in step 3): `C:\Users\amaiya\.venv\Scripts`
7. Install packages you want: `uv pip install cowsay --trusted-host pypi.org --trusted-host files.pythonhosted.org`
8. Install PyTorch: `uv pip install torch torchvision torchaudio`
9. Install prebuilt llama-cpp-python: `uv pip install llama-cpp-python==0.2.90 --extra-index-url https://abetlen.github.io/llama-cpp-python/whl/cpu`
10. Install pdfplumber due to [uv bug](https://github.com/Unstructured-IO/unstructured-inference/issues/368): `uv pip install pdfplumber`
11. Install onprem: `uv pip install onprem`
12. Install latest Visual C++ Redistributable. (See As described in [this issue](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/16342#discussioncomment-10279473), there is an issue with importing `onnxruntime`, which requires Visual C++ Redistributable.)  [Direct Link to EXE](https://aka.ms/vs/17/release/vc_redist.x64.exe).
13. Try onprem:
     ```python
     from onprem import LLM
     llm = LLM()
     llm.prompt('List three cute names for a cat.')
     ```





## Using Conda/Mamba (with CUDA Support)

1. Download the latest Windows 11 NVIDIA driver for your graphics card from [here](https://www.nvidia.com/Download/index.aspx). On June 1st, 2024, the latest driver version was 552.22. (You may also specifically need the CUDA toolkit from NVIDIA.)
2. Install [Miniconda](https://docs.anaconda.com/free/miniconda/). (Install for all users and set as system python. See STEP 10.)
3. If you're behind a corporate firewall, run this `conda config --set ssl_verify path/to/ca-bundle.crt` (where `ca-bundle.crt` contains certificates for your company.)
4. Go to the [PyTorch - Get Started](https://pytorch.org/get-started/locally/) page and run the recommended command (e.g., `conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia`).
5. Run this at a Python prompt to verify things are working:
   ```python
   In [1]: import torch

   In [2]: torch.cuda.is_available()
   Out[2]: True

   In [3]: torch.cuda.get_device_name()
   Out[3]: 'NVIDIA RTX A1000 6GB Laptop GPU'
   ```
6. Download and install [Visual Studio Community Edition](https://visualstudio.microsoft.com/vs/community/) and make sure you select the following options:
    1. Desktop development with C++
	2. Python development
	3. Linux embedded development with C++

7. Install `pip` if it is not already installed: `conda install pip`
8. The `nvcc` package is required to build `llama-cpp-python` with cuda support: `conda install cuda -c nvidia`
9. Install `llama-cpp-python` by running the following in a command prompt:
    ```sh
	# Windows
	$env:CMAKE_ARGS = "-DLLAMA_CUDA=ON"
	pip install --upgrade --force-reinstall llama-cpp-python --no-cache-dir
	```
10. If the previous command, returns an error that says "No Cuda Toolset found", try the solution [here](https://github.com/NVlabs/tiny-cuda-nn/issues/164#issuecomment-1280749170)


## Additional Tips

### Error

```sh
$ sudo apt-get update
Err:1 http://archive.ubuntu.com/ubuntu focal InRelease
Temporary failure in name rerolution

$ host google.com
;; connection timed out; no servers could be reached
```

### Solution
The ***/etc/resolv.conf*** is the main configuration file for the DNS name resolver library. It was automatically generated by WSL. Some time there was a problem with that DNS.

1. To stop automatic generation of ***resolv.conf***, add the following entry to ***/etc/wsl.conf***:

```sh
$ sudo cat << EOF > /etc/wsl.conf
[network]
generateResolvConf = false
EOF
```

2. In a cmd/powershell window, run:
```ps
> wsl --shutdown
```
or:
```ps
> wsl --terminate <Distro>
```

3. Restart WSL
4. Create a file: ***/etc/resolv.conf***. If it exists (even a link), replace existing one with new file.

```sh
sudo cat << EOF > /etc/resolv.conf
# Use one or many DNS servers you like
# nameserver 192.168.1.1
nameserver 8.8.8.8
nameserver 1.1.1.1
EOF
```

5. Shutdown and restart WSL again.

### Still not working
cmd/powershell as admin:

```ps
> wsl --shutdown
> netsh winsock reset
> netsh int ip reset all
> netsh winhttp reset proxy
> ipconfig /flushdns
```

Restart Windows.

### Ref:
- Colten Krauter: [Fix DNS resolution in WSL2](https://gist.github.com/coltenkrauter/608cfe02319ce60facd76373249b8ca6)
- RedHat: [Chapter 27. Manually configuring the /etc/resolv.conf file](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/configuring_and_managing_networking/manually-configuring-the-etc-resolv-conf-file_configuring-and-managing-networking)
- StackExchange: [How do I set my DNS when resolv.conf is being overwritten?](https://unix.stackexchange.com/questions/128220/how-do-i-set-my-dns-when-resolv-conf-is-being-overwritten)
- TechMint: [How To Set Permanent DNS Nameservers in Ubuntu and Debian](https://www.tecmint.com/set-permanent-dns-nameservers-in-ubuntu-debian/)
- rescenic: [No network connection in any distribution under WSL 2](https://github.com/microsoft/WSL/issues/5336#issuecomment-653881695)
- cudatoolkit: https://stackoverflow.com/questions/61533291/is-it-still-necessary-to-install-cuda-before-using-the-conda-tensorflow-gpu-pack/61538568#61538568



