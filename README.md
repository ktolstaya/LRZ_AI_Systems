# LRZ_AI_Systems
A short instruction how to start computations using GPU and conda on Reibniz Rechnenzentrum (LRZ) AI Systems

It is convenient to use MobaXTerm for accessing Linux drive, it has explorer window, and terminal window.
To use GPU machines via SLURM you need first to submit a ticket to LRZ Helpdesk, to get access:
https://doku.lrz.de/3-access-and-getting-started-10746642.html


**0.** Go to your home directory:
type in windows command window (or some terminal window on you system): ssh <user_id2>@login.ai.lrz.de
enter password, when prompted
you are in your linux home directory

**1.** Install miniconda (https://docs.anaconda.com/miniconda/):

```
$ mkdir -p ~/miniconda3
$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
$ bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
$ rm ~/miniconda3/miniconda.sh
```

**2.** Create conda env. (first add path to conda to PATH variable):
````
$ export PATH="<home_folder>/miniconda3/bin:$PATH"
$ conda create -n py39 python=3.9
````

**3.** Install packages and run test.py (test.py will output, that CUDA is not available):
````
$ export PATH="<home_folder>/miniconda3/bin:$PATH"
$ source deactivate
$ source activate py39
$ conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
$ conda install -c menpo opencv
$ conda install pandas seaborn tqdm openpyxl scikit-learn onnx h5py monai
$ python3 test.py
````
**4.** test.py contents:
````
print('Hello, world')
import sys
print(sys.executable)
import torch
print('Cuda is available: ', torch.cuda.is_available(), ' number of devices: ', torch.cuda.device_count())
for i in range(torch.cuda.device_count()):
   print(torch.cuda.get_device_properties(i).name)
````

**5a.** Resources allocation (For example, allocation within the lrz-v100x2 partition with single GPU), 
    interactive session (this session time is shorter, than session, allocated with sbatch, so for 
    long computations sbatch is preferred way). This time test.py must say, that CUDA is available, 
    and list available GPU devives:
````
$ salloc -p lrz-v100x2 --gres=gpu:1
$ srun --pty bash
$ conda activate py39
$ python3 test.py
````
**5b.** Run test script using sbatch command, without prior resource allocation
    (this will put the job in queue and execute it, output will be in log file "my_test.log"):
````
$ sbatch test_script.sh
````

**6.** Content of test_scrpit.sh (please, change the address of home directory <home_folder>) to run script on lrz-v100x2 machine (for example):
````
#!/bin/bash
#SBATCH --job-name=my_test
#SBATCH --output=my_test.log
#SBATCH --error=my_test.log
#SBATCH --nodes 1
#SBATCH --ntasks=1
#SBATCH --gres=gpu:1
#SBATCH -p lrz-v100x2
#SBATCH --time=0:10:00
echo "################################"
echo SLURM_PROCID=${SLURM_PROCID}
export PATH="<home_folder>/miniconda3/bin:$PATH"
echo on
nvidia-smi
source deactivate
source activate py39
python3 test.py
echo "################################"
````



**7.** It is possible to setup Python remote interpreter in PyCharm (or Microsoft VSCode), using host name login.ai.lrz.de and user_id

To read further about SLURM: https://blog.ronin.cloud/slurm-intro/
