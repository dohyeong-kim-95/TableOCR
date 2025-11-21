# Development Setup Guide

This guide will help you set up your development environment for TableOCR.

## Prerequisites

Before you begin, ensure you have the following installed:

- **Python 3.8 or higher** ([Download](https://www.python.org/downloads/))
- **pip** (usually comes with Python)
- **git** (for version control)

### Verify Prerequisites

```bash
# Check Python version (should be 3.8+)
python --version
# or
python3 --version

# Check pip
pip --version
# or
pip3 --version

# Check git
git --version
```

## Step-by-Step Setup

### 1. Clone the Repository

```bash
git clone https://github.com/dohyeong-kim-95/TableOCR.git
cd TableOCR
```

### 2. Create Virtual Environment

**Why?** Virtual environments isolate project dependencies from your system Python installation.

#### On Linux/macOS:
```bash
python3 -m venv venv
source venv/bin/activate
```

#### On Windows:
```bash
python -m venv venv
venv\Scripts\activate
```

**Verification**: Your terminal prompt should now show `(venv)` at the beginning.

### 3. Upgrade pip

```bash
pip install --upgrade pip
```

### 4. Install Dependencies

```bash
pip install -r requirements.txt
```

**Note**: This will take several minutes as it downloads and installs:
- PaddlePaddle framework (~200MB)
- PaddleOCR (~50MB)
- PDF processing libraries
- Image processing libraries

### 5. Verify Installation

#### Check Installed Packages

```bash
pip list
```

You should see packages including:
- `pdfplumber`
- `PyMuPDF`
- `paddlepaddle`
- `paddleocr`
- `opencv-python`
- `Pillow`

#### Quick Test (Once Code is Available)

```bash
# This will work once main.py is implemented
python main.py --help
```

### 6. First-Time Model Download

PaddleOCR automatically downloads required models on first use:

```python
# Test in Python interactive shell
python3
>>> from paddleocr import PPStructure
>>> engine = PPStructure(lang='korean')
# Models will download here (first time only)
# - Table detection: ~10MB
# - Structure recognition (SLANet): ~6MB
# - Korean OCR: ~12MB
# - English OCR: ~10MB
>>> exit()
```

**Model Location**: Models are cached in `~/.paddleocr/` (Linux/macOS) or `C:\Users\<username>\.paddleocr\` (Windows).

**First Run Time**: 2-5 minutes depending on internet speed.

## Development Tools (Optional but Recommended)

### Install Development Dependencies

```bash
pip install pytest pytest-cov black flake8 mypy
```

### IDE Setup

#### Visual Studio Code
1. Install Python extension
2. Select virtual environment:
   - `Cmd/Ctrl + Shift + P`
   - Type "Python: Select Interpreter"
   - Choose `./venv/bin/python`

3. Recommended extensions:
   - Python (Microsoft)
   - Pylance
   - Python Test Explorer

#### PyCharm
1. Open project folder
2. Go to Settings → Project → Python Interpreter
3. Click gear icon → Add → Existing Environment
4. Select `./venv/bin/python` (Linux/macOS) or `.\venv\Scripts\python.exe` (Windows)

## Verify Your Setup

### Run Tests (Once Available)

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src tests/

# Run specific test file
pytest tests/unit/test_pdf_processor.py
```

### Code Quality Checks

```bash
# Format code with Black
black src/ tests/

# Lint with flake8
flake8 src/ tests/

# Type checking with mypy
mypy src/
```

## Common Issues and Solutions

### Issue 1: Python Version Too Old

**Error**: `python: command not found` or version < 3.8

**Solution**:
- Install Python 3.8+ from [python.org](https://www.python.org/downloads/)
- On Linux: `sudo apt install python3.8` (Ubuntu/Debian) or `sudo yum install python3.8` (CentOS/RHEL)
- On macOS: `brew install python@3.8`

### Issue 2: Virtual Environment Activation Fails (Windows)

**Error**: `cannot be loaded because running scripts is disabled`

**Solution**:
```powershell
# Run PowerShell as Administrator
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# Then try activating again
venv\Scripts\activate
```

### Issue 3: PaddlePaddle Installation Fails

**Error**: `ERROR: Could not find a version that satisfies the requirement paddlepaddle`

**Solution**:
```bash
# Use alternative installation method
pip install paddlepaddle==2.5.0 -i https://pypi.tuna.tsinghua.edu.cn/simple

# Or for specific platform
# Linux CPU:
pip install paddlepaddle==2.5.0 -i https://mirror.baidu.com/pypi/simple

# macOS:
pip install paddlepaddle==2.5.0
```

### Issue 4: Model Download Fails

**Error**: `URLError` or connection timeout during model download

**Solution**:
1. Check internet connection
2. Try manual download:
   ```bash
   # Create model directory
   mkdir -p ~/.paddleocr/whl/

   # Download models manually from PaddleOCR GitHub releases
   # Then place in ~/.paddleocr/whl/
   ```

3. Or specify model path:
   ```python
   from paddleocr import PPStructure
   engine = PPStructure(
       table_model_dir='path/to/table/model',
       lang='korean'
   )
   ```

### Issue 5: OpenCV Import Error

**Error**: `ImportError: libGL.so.1: cannot open shared object file`

**Solution** (Linux):
```bash
sudo apt-get install libgl1-mesa-glx
# or
sudo yum install mesa-libGL
```

### Issue 6: Memory Error During Installation

**Error**: `MemoryError` or system freezes

**Solution**:
```bash
# Install packages one by one with --no-cache-dir
pip install --no-cache-dir paddlepaddle
pip install --no-cache-dir paddleocr
pip install --no-cache-dir pdfplumber PyMuPDF opencv-python Pillow
```

### Issue 7: Permission Denied

**Error**: `Permission denied` when installing packages

**Solution**:
```bash
# Use --user flag (not recommended with venv)
pip install --user -r requirements.txt

# Or use sudo (Linux/macOS, only if not using venv)
sudo pip3 install -r requirements.txt

# Better: Ensure venv is activated
source venv/bin/activate  # Linux/macOS
venv\Scripts\activate     # Windows
```

## Environment Variables (Optional)

You can customize behavior with environment variables:

```bash
# Linux/macOS
export PADDLEOCR_MODEL_DIR=/path/to/models
export TABLEOCR_OUTPUT_DIR=/path/to/output

# Windows
set PADDLEOCR_MODEL_DIR=C:\path\to\models
set TABLEOCR_OUTPUT_DIR=C:\path\to\output
```

Add to shell configuration for persistence:
- Linux/macOS: `~/.bashrc` or `~/.zshrc`
- Windows: Environment Variables in System Properties

## Quick Start After Setup

```bash
# Activate virtual environment
source venv/bin/activate  # Linux/macOS
venv\Scripts\activate     # Windows

# Run the application (once implemented)
python main.py sample.pdf --output result.html

# Run tests
pytest

# Deactivate virtual environment when done
deactivate
```

## Project Directory Overview

After setup, your directory should look like:

```
TableOCR/
├── venv/                    # Virtual environment (git-ignored)
├── src/                     # Source code
├── tests/                   # Test files
├── docs/                    # Documentation
├── examples/                # Sample files
├── output/                  # Generated outputs (git-ignored)
├── requirements.txt         # Python dependencies
├── .gitignore              # Git ignore rules
├── README.md               # Project readme
└── main.py                 # Entry point (to be created)
```

## System Requirements

### Minimum:
- **CPU**: Dual-core 2.0 GHz
- **RAM**: 4 GB
- **Disk**: 2 GB free space
- **OS**: Windows 10, macOS 10.14+, or Linux (Ubuntu 18.04+)

### Recommended:
- **CPU**: Quad-core 2.5 GHz+
- **RAM**: 8 GB+
- **Disk**: 5 GB free space (for models and output)
- **OS**: Windows 11, macOS 12+, or Linux (Ubuntu 20.04+)

## Next Steps

1. ✅ Environment set up
2. ✅ Dependencies installed
3. ✅ Models downloaded (on first use)
4. 📝 Read [ARCHITECTURE.md](ARCHITECTURE.md) to understand the system design
5. 📝 Read [API_SPECIFICATION.md](API_SPECIFICATION.md) for development guidelines
6. 🛠️ Start developing modules (see architecture for order)
7. ✅ Write tests as you develop
8. 🚀 Run integration tests before committing

## Getting Help

If you encounter issues not covered here:

1. Check the [troubleshooting section](#common-issues-and-solutions) above
2. Review [GitHub Issues](https://github.com/dohyeong-kim-95/TableOCR/issues)
3. Check PaddleOCR documentation: https://github.com/PaddlePaddle/PaddleOCR
4. Open a new issue with:
   - Your OS and Python version
   - Full error message
   - Steps to reproduce

## Updating Dependencies

```bash
# Update all packages
pip install --upgrade -r requirements.txt

# Update specific package
pip install --upgrade paddleocr

# Check for outdated packages
pip list --outdated
```

## Uninstall/Clean Up

```bash
# Deactivate virtual environment
deactivate

# Remove virtual environment
rm -rf venv/  # Linux/macOS
rmdir /s venv  # Windows

# Remove downloaded models
rm -rf ~/.paddleocr/  # Linux/macOS
rmdir /s %USERPROFILE%\.paddleocr  # Windows

# Remove output files
rm -rf output/
```

---

You're now ready to start developing! 🚀
