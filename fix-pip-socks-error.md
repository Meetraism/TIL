**TIL — Fixing "Missing dependencies for SOCKS support" when installing requests in Python 3.12 venv**

**The Problem**

While working on a SQL injection automation script in Python, I created a virtual environment using:
```
python3.12 -m venv venv
source venv/bin/activate
```
Inside this venv, I tried to install the requests library using:
```
pip install requests
```
But I kept hitting this frustrating error:
```
ERROR: Could not install packages due to an OSError: Missing dependencies for SOCKS support.
```
I also tried:
```
pip install "requests[socks]"
pip install PySocks
pip install requests --no-deps
# and installing python3-socks, python3-pysocks, and more...
```
Nothing worked.
Even ```--break-system-packages``` didn’t help. The venv was completely stuck.

**Cause (from what I learned)**

- The requests package optionally installs PySocks to support SOCKS proxies.
- PySocks includes native (C-based) components that must be compiled.
- My venv was missing build tools and dev headers like libffi-dev, libssl-dev, or python3-dev.
- It wasn’t requests that was broken — it was my environment setup.

**What About Installing Globally?**

Out of curiosity, I also tried installing requests globally (outside the venv):
```
pip install requests
```
But this time, I ran into a different kind of error:
```
error: externally-managed-environment
× This environment is externally managed
╰─> To install Python packages system-wide, try apt install python3-xyz
```
Apparently, my system’s Python 3.12 environment is protected.
Ubuntu (and other Linux distros) now follow PEP 668, which restricts direct pip installations into the global environment — to avoid accidentally breaking system tools.

Since I didn’t want to override system protection with --break-system-packages, I went back to working inside venvs.
✅ The Fix (Workaround That Worked)

Here’s what finally worked for me:
```
# Remove the broken venv:
rm -rf venv

# Install Python 3.10 via deadsnakes PPA:
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt update
sudo apt install python3.10 python3.10-venv python3.10-dev

# Create a new venv with Python 3.10:
python3.10 -m venv venv
source venv/bin/activate

# Manually install requests from source:
wget https://github.com/psf/requests/archive/refs/tags/v2.31.0.zip
unzip v2.31.0.zip
cd requests-2.31.0
python setup.py install

# Verify:
python -c "import requests; print(requests.__version__)"
# ✅ Output: 2.31.0
```

***Reflection***

_This was a really interesting issue that started with a vague error message and turned out to be a mix of environment, versioning, and system policy.

TIL: Sometimes, your code isn’t the problem — your environment is.
Learning to navigate pip, Python versions, and system policies is just as important as writing Python itself._
