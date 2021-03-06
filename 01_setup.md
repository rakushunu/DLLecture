# Set Up Deep Learning Research Environment

[TOC]

## 1. Remote Linux Server (DL-box)

### 1.1 Login without password

`{ }`means a placeholder in the following.

##### MacOS

1. Create a key file for ssh

   ```bash
   ssh-keygen
   ```

2. Install Homebrew

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

3. Add a public key to the remote server's authorized_keys file

```bash
brew install ssh-copy-id
ssh-copy-id {user name}@{remote IP}
```

##### Windows 10 ([Ubuntu subsystem](https://docs.microsoft.com/en-us/windows/wsl/install-win10))

Use the subsystem environment to invoke linux commands in the following.

```bash
ssh-keygen
ssh-copy-id {username}@{remote IP}
```

### 1.2 Log in to server

```
ssh -l{user name} {remote IP}
```

If you want to change the password, use

```bash
passwd
```

### 1.3 Mount remote drivers

#### MacOS

```bash
brew cask install osxfuse
brew install sshfs
mkdir -p ~/Mounts/{mount point}
sshfs {username}@{remote IP}:/home/{username} ~/Mounts/{mount point}
```

When you want to unmount a driver,

```bash
diskutil umount ~/Mounts/{mount point}
```

#### Windows

Install sshfs for Windows by following [this](https://github.com/feo-cz/win-sshfs).

---

_<u>Notice</u>_: The storage of `/home/*` in the remote server is shared by all users at this time. If you want to handle big data files (eg. >=1GB ), please consider to additionally mount `{data}` volumes for your usage. Let's take this as an exercise.

## 2. Python

### 2. 1. Pyenv

- To easily switch multiple versions of Python (2 or 3)

- To install required libraries for your projects without system privilege

- Reference: https://github.com/pyenv/pyenv

  ​ https://github.com/pyenv/pyenv-virtualenv

#### Installation

##### MacOS

```bash
brew install pyenv pyenv-virtualenv
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo 'if command -v pyenv 1>/dev/null 2>&1; ' \
'then eval "$(pyenv init -)"; fi' >> ~/.bash_profile
echo 'if which pyenv-virtualenv-init > /dev/null; ' \
'then eval "$(pyenv virtualenv-init -)"; fi' >> ~/.bash_profile
exec "$SHELL"
```

##### Linux or the subsystem in Windows 10

```bash
curl -L https://github.com/pyenv/pyenv-installer/raw/master/bin/pyenv-installer | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
source ~/.bash_profile
```

### 2.2. Anaconda

- To easily manage python libraries for data science and machine learning
- Install Anaconda 2&3 to adapt to different environments by various projects.
- ~~Choose version 4.3 for Anaconda 3 due to the issue (https://github.com/pyenv/pyenv-virtualenv/issues/246) of persistent virtualenv setting.~~

#### Installation

##### Anaconda 2

```bash
pyenv install miniconda2-latest
pyenv virtualenv miniconda2-latest DLLecture2
mkdir -p ~/Projects/DLLecture2
cd ~/Projects/DLLecture2
pyenv local DLLecture2
```

Current environment: DLLecture2 (miniconda2-latest/envs/DLLecture2)

```bash
conda install anaconda
```

##### Anaconda 3

Using miniconda 2 to create environments of Anaconda 3 can avoid the issue.

```bash
mkdir -p ~/Projects/DLLecture
cd ~/Projects/DLLecture
pyenv local miniconda2-latest
```

Current environment: miniconda2-latest

```bash
conda update conda
conda create -n DLLecture python==3.6.6      # Important!!
pyenv local miniconda2-latest/envs/DLLecture
```

Current environment: miniconda2-latest/envs/DLLecture

```bash
conda install anaconda
```

Check whether your virtualenv is set correctly.

```bash
python -c 'import sys; print(sys.path)';
```

The output may be similar to the following.

```
['', '{your home}/.pyenv/versions/miniconda2-latest/envs/DLLecture/lib/python36.zip', '{your home}/.pyenv/versions/miniconda2-latest/envs/DLLecture/lib/python3.6', '{your home}/.pyenv/versions/miniconda2-latest/envs/DLLecture/lib/python3.6/lib-dynload', '{your home}/.pyenv/versions/miniconda2-latest/envs/DLLecture/lib/python3.6/site-packages']
```

Let's use `(miniconda2-latest/envs/DLLecture)` as our default environment in the following.

## 3. DL Libraries

### 3.1. TensorFlow

Use v1.10.0.

#### For Mac or non-CUDA machine

```bash
conda install tensorflow==1.10.0
```

#### For CUDA environment (DL-box)

```bash
conda install tensorflow-gpu==1.10.0
```

### 3.2. Pytorch

Use v0.4.1.

```bash
pip install tensorboardX
conda install pytorch==0.4.1 torchvision -c pytorch
```

It may take a while to download...

## 4. Jupyter

- A good interactive notebook environment for research

### 4.1. Create a Notebook password

1. Open a python shell

```bash
cd ~/Projects/DLLecture
python
```

2. Generate pass code

```python
from notebook.auth import passwd; passwd()
```

3. Then copy the pass code `type:salt:hashed-password` after inputing and verifying your password.

4. Type `exit()` to quit the shell.

### 4.2. Configure Notebook

```bash
jupyter notebook --generate-config
vim ~/.jupyter/jupyter_notebook_config.py
```

- Three modes in `vim` : Command, Input, Line. The default mode is Command.
- In Command mode, type `/` and item string to search the item in `vim`.
- Type `i` to enter Input mode. you can modify the contents in Input mode.
- Press `esc` back to Command mode anytime.
- In Command mode, type `:wq` to save your change and quit.

These items need to be set. To set new value, delete the `#` in front of the items firstly.

```python
c.NotebookApp.ip = 'localhost' # '{remote IP}' for DL-box
c.NotebookApp.password = u'{paste the above pass code}'
c.NotebookApp.port = {your port} # 8888 for your PC or unused one in 1024~65535 for DL-box
c.NotebookApp.open_browser = False
```

### 4.3. Launch Notebook server

#### In your PC

```bash
jupyter notebook
```

Then open`http://localhost:8888/` in the browser and login with your password.

#### In remote server (DL-box)

```bash
byobu new -s jupyter
jupyter notebook
```

Press `F6` to detach from the session where the program will keep running even if you logout.

If you want to enter a session again, use `byobu attach -t {session name}` or just run `byobu` to select one. For more details about `byobu`, see http://byobu.co.

Now, open`http://{remote IP}:{your port}/` in the browser and login with your password.

### 4.4. Install Python kernel

- Install kernel to let the virtual environment work in Jupyter.

```bash
cd ~/Projects/DLLecture
python -m ipykernel install --user --name DLLecture
```

### 4.5. Launch TensorBoard server

#### In your PC

Open a new terminal.

```bash
cd ~/Projects/DLLecture
tensorboard --logdir ./runs
```

Then open`http://localhost:6006/` in the browser.

#### In remote server (DL-box)

```bash
byobu new -s tensorboard
cd ~/Projects/DLLecture
tensorboard --logdir ./runs --port {your another port}
```

Press `F6` to detach from the session.

Then open`http://{remote IP}:{your another port}/` in the browser.

## 5. Test on your settings

- The home of Jupyter is the location where `jupyter` was executed, i.e., `~/Projects/DLLecture` in this example.

- Click the `New` in the right top coner to create a new folder, then check the folder and rename it to `runs` for tensorboard's data loading.
- Click `New` to create a new `DLLecture` notebook.
- Copy the following code (edited from this [demo](https://github.com/lanpa/tensorboardX)) into the first shell, then press `shift`+`return` to run it,

```python
import torch
import torchvision.utils as vutils
import numpy as np
import torchvision.models as models
from torchvision import datasets
from tensorboardX import SummaryWriter

logdir = './runs'
resnet18 = models.resnet18(False)
writer = SummaryWriter(logdir)
sample_rate = 44100
freqs = [262, 294, 330, 349, 392, 440, 440, 440, 440, 440, 440]

for n_iter in range(100):

    dummy_s1 = torch.rand(1)
    dummy_s2 = torch.rand(1)
    # data grouping by `slash`
    writer.add_scalar('data/scalar1', dummy_s1[0], n_iter)
    writer.add_scalar('data/scalar2', dummy_s2[0], n_iter)

    writer.add_scalars('data/scalar_group', {'xsinx': n_iter * np.sin(n_iter),
                                             'xcosx': n_iter * np.cos(n_iter),
                                             'arctanx': np.arctan(n_iter)}, n_iter)

    dummy_img = torch.rand(32, 3, 64, 64)  # output from network
    if n_iter % 10 == 0:
        x = vutils.make_grid(dummy_img, normalize=True, scale_each=True)
        writer.add_image('Image', x, n_iter)

        dummy_audio = torch.zeros(sample_rate * 2)
        for i in range(x.size(0)):
            # amplitude of sound should in [-1, 1]
            dummy_audio[i] = np.cos(freqs[n_iter // 10] * np.pi * float(i) / float(sample_rate))
        writer.add_audio('myAudio', dummy_audio, n_iter, sample_rate=sample_rate)

        writer.add_text('Text', 'text logged at step:' + str(n_iter), n_iter)

        for name, param in resnet18.named_parameters():
            writer.add_histogram(name, param.clone().cpu().data.numpy(), n_iter)

        # needs tensorboard 0.4RC or later
        writer.add_pr_curve('xoxo', np.random.randint(2, size=100), np.random.rand(100), n_iter)

writer.add_graph(resnet18, torch.randn(1, 3, 224, 224))

dataset = datasets.MNIST('mnist', train=False, download=True)
images = dataset.test_data[:100].float()
label = dataset.test_labels[:100]

features = images.view(100, 784)
writer.add_embedding(features, metadata=label, label_img=images.unsqueeze(1))

# export scalar data to JSON for external processing
writer.export_scalars_to_json("./all_scalars.json")
writer.close()
```

Wait until it finishes, and see the result on TensorBoard.
