# Installing c++ support into jupyter notebooks

## Windows

Download Miniconda https://docs.conda.io/en/latest/miniconda.html

Check hash with Powershell 
Get-FileHash "C:\Users\erici\Downloads\Miniconda3-latest-Windows-x86_64.exe" -Algorithm SHA256

Open the Anaconda Prompt (MiniConda3) 
conda

Instructions here: https://www.learnopencv.com/xeus-cling-run-c-code-in-jupyter-notebook/
Run 

conda create -n xeus-cling
source activate xeus-cling
conda install -c conda-forge xeus-cling