# Create python environment

## For GPU
1. python -m venv llm_pd_env
2. source llm_pd_env/bin/activate      # Linux/macOS
3. llm_pd_env\Scripts\activate       # Windows
4. python -m pip install --upgrade pip
5. pip install torch torchvision torchaudio
6. pip install transformers datasets trl peft accelerate bitsandbytes scikit-learn pandas numpy matplotlib
7. If you use Jupyter:
8. pip install notebook ipykernel
9. python -m ipykernel install --user --name llm_pd_env --display-name "Python (llm_pd_env)"

## Check and fix compatible version to run code in GPU
Install the same PyTorch version/build that works outside the venv.

### First compare both environments.
In system Python: python3 -c "import torch; print(torch.__version__); print(torch.version.cuda)"

In your venv: 
1. source llm_pd_env/bin/activate
2. python3 -c "import torch; print(torch.__version__); print(torch.version.cuda)"

If the system one shows an older CUDA build like 11.8, 12.1, or 12.4, install that same build in the venv.

### Recommended repair steps

Inside llm_pd_env:

1. Remove the incompatible torch install: pip uninstall -y torch torchvision torchaudio
2. Install a compatible official wheel

A good conservative choice is CUDA 12.1 or CUDA 11.8, because official previous-version wheels exist for both across multiple PyTorch releases.

Example with PyTorch 2.5.1 + CUDA 12.1:

pip install torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu121

Official PyTorch also lists 2.5.1 wheels for CUDA 11.8, 12.1, and 12.4.

If cu121 still fails, go one step lower:

pip install torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu118

That build is also officially available.

If you want an older release family, PyTorch also provides official commands such as:

pip install torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 --index-url https://download.pytorch.org/whl/cu121

and

pip install torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 --index-url https://download.pytorch.org/whl/cu118

### Verify in the venv
python3 -c "import torch; print(torch.__version__); print(torch.version.cuda); print(torch.cuda.is_available()); print(torch.cuda.device_count()); print(torch.cuda.get_device_name(0))"

You want: torch.cuda.is_available() → True

### Reconnect Jupyter to the repaired env
After fixing torch in the venv, reinstall the kernel entry just to be safe:

1. pip install ipykernel
2. python -m ipykernel install --user --name llm_pd_env --display-name "Python (llm_pd_env)"

Then:

1. stop the notebook kernel
2. restart Jupyter
3. reopen the notebook
4. select Python (llm_pd_env) again

Notebook kernels can keep old imports alive until restarted.

5. Verify inside Jupyter

Run:

import torch
print(torch.__version__)
print(torch.version.cuda)
print("CUDA available:", torch.cuda.is_available())
print("CUDA device count:", torch.cuda.device_count())
if torch.cuda.is_available():
    print("GPU name:", torch.cuda.get_device_name(0))

### Why this happened

Your first install likely used the newest default PyTorch wheel, and the current stable selector offers builds including CUDA 12.6 and CUDA 12.8. If your server driver is older, that newer build can fail, while an older compatible wheel still works.

What I would do in your case

### I would try this first inside llm_pd_env:

1. pip uninstall -y torch torchvision torchaudio
2. pip install torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu121

### If CUDA is still unavailable, then switch to:

1. pip uninstall -y torch torchvision torchaudio
2. pip install torch==2.5.1 torchvision==0.20.1 torchaudio==2.5.1 --index-url https://download.pytorch.org/whl/cu118

Then restart the Jupyter kernel.

Paste the output of this after reinstall:

python3 -c "import torch; print(torch.__version__); print(torch.version.cuda); print(torch.cuda.is_available()); print(torch.cuda.device_count())"

and I’ll tell you whether your environment is fully ready for training.

## If mismatched found in torch version
If you confirmed it: the system Python has a CUDA 12.8 build, while your virtual environment has a CUDA 13.0 build.
### What to do

The safest fix is to make llm_pd_env use the same working Torch family as the system Python.

Inside llm_pd_env:

1. pip uninstall -y torch torchvision torchaudio
2. pip cache purge
3. pip install torch==2.9.1 torchvision torchaudio --index-url https://download.pytorch.org/whl/cu128

### Then verify

Still inside llm_pd_env:

python3 -c "import torch; print(torch.__version__); print(torch.version.cuda); print(torch.cuda.is_available()); print(torch.cuda.device_count())"

You want something like:

2.9.1+cu128
12.8
True
2

### Reconnect Jupyter kernel

After that, reinstall the kernel entry and restart the notebook kernel:

1. pip install ipykernel
2. python -m ipykernel install --user --name llm_pd_env --display-name "Python (llm_pd_env)"



