# MSML610 Tutorials – Assignment Setup Guide

This README documents the exact process I followed to build the MSML610 tutorial environment, modify the DevOps scripts, run Docker, and execute all Lesson 07 notebooks without errors.
It matches the workflow described in the assignment prompt and reflects all fixes verified in the final report .

---

## **Objective**

The goal of this assignment was to:

* Build a clean Docker-based environment for MSML610 tutorials using the `umd_classes` repository.
* Enable JupyterLab inside the container with all required helpers.
* Run Lesson 07 notebooks end to end, installing missing dependencies where needed.
* Fix import issues in `msml610_utils.py` and update Python paths to match the repo structure.

---

## **Repository Path**

All work happens inside:

```
umd_classes/msml610
```

This folder contains:

* Tutorials
* DevOps Docker scripts
* Helpers
* Build and run configurations

*(Shown in page 1, Fig. 1 of the report)* 

---

## **1. Environment Setup**

Before modifying Docker files, make sure your environment (thin_env or equivalent) is activated and you are inside:

```
cd umd_classes/msml610
```

---

## **2. Required Script Modifications**

Two files must be updated so Jupyter and helper imports work correctly inside the container.

### **2.1 Update `run_jupyter_server.sh`**

Locate this line:

```
echo "##> $FILE_NAME"
```

Add **these two lines directly below it**:

```
source /venv/bin/activate
cd /workspace
```

Reason:
This forces the container to use the Python environment located in `/venv` and ensures all notebooks run from `/workspace`.
(Page 1, Section 2.1) 

---

### **2.2 Update `install_python_packages.sh`**

Find this line:

```
poetry install --no-root
```

Add **the following right below it**:

```
pip install jupyter jupyterlab jupytext ipykernel
```

Reason:
These are not installed by default and are required to start JupyterLab.
(Page 1, Section 2.2) 

---

## **3. Build the Docker Image**

Run this command inside `msml610`:

```
docker build --no-cache \
  -f tutorials/devops/docker_build/dev.Dockerfile \
  --build-arg POETRY_MODE=update \
  -t msml610assignmentimage \
  .
```

Notes:

* `--no-cache` forces a clean rebuild.
* Keep the final period.
  (Page 1–2, Figures 2–3) 

---

## **4. Run the Docker Jupyter Server**

Move one folder above:

```
cd ..
```

Then launch the container:

```
docker run --rm \
  --entrypoint "" \
  -p 8888:8888 \
  -v $(pwd):/workspace \
  -e PORT=8888 \
  msml610assignmentimage \
  /workspace/msml610/tutorials/devops/docker_run/run_jupyter_server.sh
```

JupyterLab will appear at:
[http://127.0.0.1:8888/lab](http://127.0.0.1:8888/lab)

(Page 2, Figures 4–5) 

---

## **5. Notebook Pre-Execution Setup**

Before running any notebook, insert this **first cell** at the top:

```python
import sys, os
sys.path.append("/workspace")
sys.path.append("/workspace/helpers_root")
sys.path.append("/workspace/msml610/tutorials")

os.environ["CSFY_GIT_ROOT_PATH"] = "/workspace/msml610"
```

Purpose:

* Makes project helpers importable
* Sets required CSFY environment variables
  (Page 2, Figure 6) 

---

## **6. Fix Import Issue in `msml610_utils.py`**

Find this line:

```
import helpers.hdbg as hdbg
```

Replace it with:

```
import helpers_root.helpers.hdbg as hdbg
```

Reason:
The helpers actually live under `helpers_root`, not the old `helpers` path.
(Page 3, Section 6 + Figure 8) 

---

## **7. Executing the Notebooks**

Run all Lesson 07 notebooks from top to bottom.
If a notebook fails due to a missing module, install it inside `/venv` using:

```
!sudo /bin/bash -c "(source /venv/bin/activate; pip install --quiet PACKAGENAME)"
```

(Page 3, Figure 7) 

Repeat until all notebooks run without errors.

---

## **8. Challenges and Fixes**

As documented in the report:

* Several ModuleNotFound errors occurred across notebooks.
* The fourth notebook required the most debugging due to argument mismatches and outdated helper paths.
* Multiple Python files were edited to align with current repo structure.
  (Page 3, Challenges section) 

All final working fixes and outputs are available at:
[https://github.com/Akash-Kadali/MSML610-Assignment-2](https://github.com/Akash-Kadali/MSML610-Assignment-2)

---

## **9. Final Results**

By the end of the assignment:

* Docker environment was built cleanly with JupyterLab in `/venv`.
* DevOps scripts were updated and working as expected.
* All helper imports resolved correctly.
* Every Lesson 07 notebook executed successfully.
* Environment is fully reproducible inside Docker.
  (Page 3, “Final Result and Summary”) 
