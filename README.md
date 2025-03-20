# PyInstaller Guide: Converting Tkinter/CustomTkinter Projects to Executable Files

This guide walks through the process of using PyInstaller to convert your Tkinter or CustomTkinter Python projects into standalone executable files.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Basic Usage](#basic-usage)
- [Handling Tkinter Applications](#handling-tkinter-applications)
- [Handling CustomTkinter Applications](#handling-customtkinter-applications)
- [Common Options](#common-options)
- [Troubleshooting](#troubleshooting)
- [Advanced Configuration](#advanced-configuration)

## Prerequisites

- Python 3.6+ installed
- Your Python project with Tkinter or CustomTkinter
- pip (Python package manager)
- Administrator access (may be required for some installations)

## Installation

Install PyInstaller using pip:

```bash
pip install pyinstaller
```

## Basic Usage

To create a basic executable from your Python script:

```bash
pyinstaller your_script.py
```

This will create:
- A `build` directory containing working files
- A `dist` directory containing the executable and supporting files

## Handling Tkinter Applications

Tkinter is part of the Python standard library, but PyInstaller sometimes needs help finding all dependencies:

```bash
# For basic Tkinter applications:
pyinstaller --hidden-import=tkinter your_tkinter_app.py

# For more complex Tkinter apps with additional modules:
pyinstaller --hidden-import=tkinter --hidden-import=tkinter.filedialog --hidden-import=tkinter.messagebox your_tkinter_app.py
```

## Handling CustomTkinter Applications

CustomTkinter requires additional configuration to package correctly:

```bash
pyinstaller --noconfirm --onedir --windowed --add-data "<path_to_customtkinter>/customtkinter;customtkinter/" your_customtkinter_app.py
```

To find the CustomTkinter package path:
```python
import customtkinter
print(customtkinter.__path__)
```

### Alternative Approach for CustomTkinter

Create a spec file for more control:

```bash
# First generate a spec file
pyinstaller --name "YourAppName" your_customtkinter_app.py

# Edit the generated .spec file to include CustomTkinter resources
# Then build using the spec file
pyinstaller YourAppName.spec
```

Add this to your spec file:
```python
import customtkinter
from PyInstaller.utils.hooks import collect_data_files

datas = collect_data_files("customtkinter")
```

## Common Options

- `--onefile`: Create a single executable file that includes all dependencies
- `--windowed` or `-w`: Hide the console window (good for GUI applications)
- `--icon=icon.ico`: Add a custom icon to your executable
- `--name "AppName"`: Set a custom name for the output executable
- `--add-data "source;destination"`: Include additional files/folders your app needs
- `--hidden-import=module`: Include Python modules that aren't automatically detected

Example with common options:
```bash
pyinstaller --onefile --windowed --icon=app_icon.ico --name "MyTkinterApp" your_script.py
```

## Troubleshooting

### Missing Modules
If you get "ModuleNotFoundError" when running the executable:
```bash
pyinstaller --hidden-import=missing_module your_script.py
```

### Missing Images or Resources
If your app uses images or other resource files:
```bash
pyinstaller --add-data "images;images" --add-data "resources;resources" your_script.py
```

### Tkinter Issues
For Tkinter-specific issues:
```bash
pyinstaller --hidden-import=tkinter --hidden-import=_tkinter your_script.py
```

### CustomTkinter Theme Issues
If CustomTkinter themes aren't appearing properly:
```bash
pyinstaller --add-data "<path_to_customtkinter>/customtkinter/assets;customtkinter/assets/" your_customtkinter_app.py
```

## Advanced Configuration

For more complex applications, create and customize a spec file:

```bash
# Generate a spec file
pyinstaller --name "YourAppName" your_script.py
# Edit the .spec file as needed, then:
pyinstaller YourAppName.spec
```

### Example Spec File for CustomTkinter

```python
# YourAppName.spec
block_cipher = None

import sys
import os
from PyInstaller.utils.hooks import collect_data_files, collect_submodules

# Get CustomTkinter data files
customtkinter_datas = collect_data_files("customtkinter")

a = Analysis(
    ['your_script.py'],
    pathex=[],
    binaries=[],
    datas=customtkinter_datas + [
        # Add your additional data files here
        ('images/', 'images/'),
    ],
    hiddenimports=collect_submodules('customtkinter'),
    hookspath=[],
    hooksconfig={},
    runtime_hooks=[],
    excludes=[],
    win_no_prefer_redirects=False,
    win_private_assemblies=False,
    cipher=block_cipher,
    noarchive=False,
)

pyz = PYZ(a.pure, a.zipped_data, cipher=block_cipher)

exe = EXE(
    pyz,
    a.scripts,
    [],
    exclude_binaries=True,
    name='YourAppName',
    debug=False,
    bootloader_ignore_signals=False,
    strip=False,
    upx=True,
    console=False,  # Set to True if you want to show console
    icon='icon.ico',  # Path to your icon
)

coll = COLLECT(
    exe,
    a.binaries,
    a.zipfiles,
    a.datas,
    strip=False,
    upx=True,
    upx_exclude=[],
    name='YourAppName',
)
```

## Example Project Structure

A well-organized project makes packaging easier:

```
your_project/
├── main.py             # Your main script
├── assets/             # Images, icons, etc.
├── resources/          # Other resources
├── requirements.txt    # Dependencies
└── build_script.py     # Optional script to automate builds
```

Example `build_script.py`:
```python
import os
import PyInstaller.__main__
import customtkinter

# Get CustomTkinter path
customtkinter_path = os.path.dirname(customtkinter.__file__)

PyInstaller.__main__.run([
    'main.py',
    '--name=YourAppName',
    '--onefile',
    '--windowed',
    f'--add-data={customtkinter_path};customtkinter',
    '--add-data=assets;assets',
    '--icon=assets/icon.ico',
])
```

## Testing Your Executable

Always test your executable in a clean environment (a different computer if possible) to ensure all dependencies are properly packaged.
