#  eSim 2.5 Installation Issues and Fixes on Ubuntu 25.xeSim 2.5 Installation Issues and Fixes on Ubuntu 25.x

## Introduction
eSim 2.5 is officially supported up to Ubuntu 24.04.
This report documents the issues encountered while installing eSim 2.5 on Ubuntu 25.10, an unsupported but forward-looking release, and the steps taken to analyze, modify, and partially fix the installation process.

The objective was not only to make the installer run, but to identify dependency breakages caused by OS upgrades, analyze installer scripts, and propose maintainable fixes.

---

##  System Details
- OS: Ubuntu 25.10 (Testing via VirtualBox)
- eSim version: 2.5
- Installation method: Official install-eSim.sh
- Architecture: x86_64

---

##  Issues

### Issue 1 : Unsupported Ubuntu Version Check (Fixed)

#### Problem

The install-eSim.sh script explicitly blocks unsupported Ubuntu versions:
```
Unsupported Ubuntu version: 25.10 ()
```
This prevents further dependency inspection and debugging.

#### Root Cause

The script uses a strict case statement on VERSION_ID, which was not updated for Ubuntu 25.x.

#### Fix Applied
Mapped Ubuntu 25.10 to the closest supported installer (24.04), allowing the installation to proceed for dependency analysis.
```
"24.04"|"25.10")
    SCRIPT="$SCRIPT_DIR/install-eSim-24.04.sh"
    ;;

```

#### Result

- Installer proceeds beyond OS check
- This change does not claim full compatibility, only controlled progression


---

### Issue 2: KiCad PPA Unsupported on Ubuntu 25.10 (Fixed)

#### Problem

The official KiCad PPA does not provide packages for Ubuntu 25.10, causing add-apt-repository failures.

#### Root Cause

PPA repositories lag behind Ubuntu development releases.

#### Fix Applied
- Skipped adding the unsupported PPA.
- Installed KiCad directly from Ubuntu repositories
- Documented Snap as an optional alternative
```
echo "Skipping PPA addition for KiCad..."
sudo apt install -y kicad
```

#### Result

- KiCad installed successfully
- Prevents installer crash
- Maintains forward compatibility

![KiCad screenshot](Ubuntu/error_images/kicad_error.jfif)


---

### Issue 3: NGHDL / Verilator Dependency Errors (Partially Addressed)

#### Problem

- Missing GTK-related dependencies.
- NGHDL archive extracted into an unexpected directory.
- Verilator dependency mismatch

![NGHDL LLVM error screenshot](Ubuntu/error_images/nghdl_error.jfif)
![GTK](Ubuntu/error_images/gtk_error.jfif)


#### Action Taken

- Non-essential GUI-related dependencies (libcanberra-gtk) were commented out to allow progress
- NGHDL archive extraction path corrected manually
  
```
tar xvf $ghdl.tar.gz
```

#### Justification

These components are not critical for core schematic and simulation workflows, and skipping them allows further debugging of higher-impact issues.

---

### Issue 4: GHDL Fails Due to LLVM 20 Incompatibility (Analyzed and identified root cause)

#### Problem

```
Unhandled version llvm 20.1.8
```

#### Root Cause

- Ubuntu 25.10 ships LLVM 20
- GHDL 4.1.0 officially supports LLVM ≤15
- Trying to install GHDL 4.1.0 (which is likely intended for Ubuntu 24.04 or older) on Ubuntu 25.10, but system has LLVM 20.1.8, which is not officially supported by that GHDL version.
Hence
```
Unhandled version llvm 20.1.8
```

![LLVM error screenshot](Ubuntu/error_images/LLVM_error.jfif)

#### Attempted Resolution

- Installed clang-15
- Proposed fixing LLVM detection by forcing LLVM 15 usage
- Created symlinks for clang++
  
```
sudo apt install clang-15 llvm-15
```

#### Status

Not fully resolved, but root cause clearly identified and mitigation strategy documented.


This issue requires:

- Rebuilding GHDL from source with LLVM 20 support
OR
- Using containerized / pinned LLVM environments

---

## Conclusion

This task highlights how OS upgrades break tightly coupled dependency chains in EDA tools like eSim.

### Key contributions:

- Enabled Ubuntu 25.x installation flow
- Fixed KiCad installation blocker
- Identified LLVM–GHDL incompatibility with actionable solutions
- Provided maintainable, well-documented script changes

While full compatibility on Ubuntu 25.x requires upstream support, this work establishes a clear path forward.


