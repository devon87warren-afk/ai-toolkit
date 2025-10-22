# Runpod Network Volume Setup - Step by Step

## Part 1: Create Network Volume

### Step 1: Go to Network Volumes
1. Log into [Runpod.io](https://runpod.io)
2. Click **"Storage"** in the left sidebar
3. Click **"Network Volumes"** tab

### Step 2: Create New Volume
1. Click **"+ Network Volume"** button (top right)
2. **Choose a datacenter:**
   - Select the same region where you'll create your pod
   - Recommended: `US East` or `EU` (whichever is closer/cheaper)
   - ⚠️ **IMPORTANT:** You can only attach volumes to pods in the SAME datacenter

### Step 3: Configure Volume Settings
Fill in the form:

- **Volume Name:** `ai-toolkit-data` (or whatever you prefer)
- **Size:**=   
  - Minimum: `20 GB` (for basic datasets)
  - Recommended: `50-100 GB` (for models + datasets + outputs)
  - Your choice based on how much data you'll store
- **Datacenter:** (should already be selected from Step 2)

### Step 4: Create
1. Click **"Create"** button
2. Wait for volume to be created (usually instant)
3. You'll see it listed with status "Ready"

---

## Part 2: Create GPU Pod and Link Volume

### Step 1: Go to GPU Pods
1. Click **"Pods"** in the left sidebar (or "GPU Pods")
2. Click **"+ Deploy"** button

### Step 2: Select GPU
1. **Filter by GPU type:**
   - Minimum: RTX 3090 (24GB VRAM)
   - Recommended: RTX 4090 or A40 (24GB+ VRAM)
   - Budget option: RTX 3090
2. Click **"Deploy"** on your chosen GPU

### Step 3: Configure Pod Template

#### Container Image:
- **Option A (Recommended):** Use Docker image
  ```
  ostris/aitoolkit:latest
  ```

- **Option B:** Use base image and install manually
  ```
  runpod/pytorch:2.1.0-py3.10-cuda11.8.0-devel-ubuntu22.04
  ```

#### Container Disk:
- Set to: `20 GB` (minimum, just for container)
- Your data will be on the network volume

#### Volume Mount Configuration:
1. **Select Volume to Mount:**
   - Click the dropdown under "Network Volume"
   - Select: `ai-toolkit-data` (the volume you created in Part 1)

2. **Volume Mount Path:**
   ```
   /workspace
   ```
   ⚠️ **CRITICAL:** Must be exactly `/workspace` (this is the standard)

#### Expose Ports:
Add these ports:
- **HTTP Ports:** `8675` (for AI Toolkit web UI)
- Click **"+ Add Port"** if you need more

#### Environment Variables (Optional):
Click **"Edit Template"** then add:
```
AI_TOOLKIT_AUTH=password
```

### Step 4: Deploy Pod
1. Review your settings:
   - ✓ GPU selected
   - ✓ Network volume attached (`ai-toolkit-data`)
   - ✓ Mount path: `/workspace`
   - ✓ Ports exposed: `8675`
2. Click **"Deploy On-Demand"** or **"Deploy Spot"**
   - **On-Demand:** More expensive, won't shut down unexpectedly
   - **Spot:** Cheaper, but can be interrupted

### Step 5: Wait for Pod to Start
- Status will change from "Provisioning" → "Running"
- Usually takes 1-3 minutes

---

## Part 3: Access and Setup

### Step 1: Connect to Pod
1. Click **"Connect"** button on your running pod
2. Choose connection method:
   - **Web Terminal** (easiest)
   - **SSH** (if you prefer)
   - **Jupyter** (if available)

### Step 2: Verify Volume is Mounted
In the terminal, run:
```bash
ls -la /workspace
```

You should see an empty directory (or previous files if you've used this volume before).

### Step 3: Setup AI Toolkit
```bash
# Navigate to workspace
cd /workspace

# Clone AI Toolkit (if not using Docker image)
git clone https://github.com/ostris/ai-toolkit.git
cd ai-toolkit

# Create necessary directories
mkdir -p datasets/tifolora
mkdir -p output
mkdir -p config
```

### Step 4: Upload Your Dataset

#### Option A: Using Runpod File Manager
1. Click the **"Files"** button on your pod
2. Navigate to `/workspace/ai-toolkit/datasets/tifolora/`
3. Click **"Upload"** and select all your images and .txt files

#### Option B: Using rsync/scp from your local machine
```bash
# Get your pod's SSH connection string from Runpod
# Then from your local machine:
scp -r C:\Users\User\Documents\GitHub\ai-toolkit\datasets\tifolora/* \
  root@your-pod-ssh-address:/workspace/ai-toolkit/datasets/tifolora/
```

#### Option C: Copy from local files already in the repo
If you're using the Docker image:
```bash
# Inside the pod terminal
cp -r /app/ai-toolkit/datasets/tifolora /workspace/ai-toolkit/datasets/
```

### Step 5: Verify Files
```bash
ls -la /workspace/ai-toolkit/datasets/tifolora/
```

You should see:
```
1.jpg
1.txt
2.jpg
2.txt
... (all your files)
```

---

## Part 4: Run Training

### If using ostris/aitoolkit Docker image:

1. **Access Web UI:**
   - Look for the pod's URL in Runpod dashboard
   - Should be something like: `https://xxxxxxxx-8675.proxy.runpod.net`
   - Or click the **"Connect to HTTP Service [8675]"** button

2. **Upload your config** or create a new job in the UI

### If running via command line:

```bash
cd /workspace/ai-toolkit
python run.py config/examples/train_lora_qwen_image_24gb.yaml
```

---

## Important Notes

### ✓ Volume Persistence
- **Network volume data persists** even when you stop/terminate the pod
- **Pod storage is temporary** - only files in `/workspace` (network volume) persist
- Always save outputs to `/workspace/ai-toolkit/output/`

### ✓ Datacenter Matching
- Network volumes can ONLY be attached to pods in the **same datacenter**
- If you get "no volumes available", check you're deploying in the same region

### ✓ Billing
- **Network volumes** are charged per GB per hour (even when not attached)
- **Pods** are charged per hour while running
- **Stop pods** when not training to save money
- **Data transfer** may have charges for large uploads/downloads

### ✓ Config File Paths
Your config must use relative paths:
```yaml
folder_path: "datasets/tifolora"  # ✓ Correct
```

NOT:
```yaml
folder_path: "C:\\Users\\..."  # ✗ Wrong - Windows path won't work
folder_path: "/workspace/ai-toolkit/datasets/tifolora"  # ✗ Wrong - absolute path not needed
```

---

## Quick Reference

**Volume Name:** `ai-toolkit-data` (your choice)
**Mount Path:** `/workspace`
**Dataset Location:** `/workspace/ai-toolkit/datasets/tifolora/`
**Output Location:** `/workspace/ai-toolkit/output/`
**Web UI Port:** `8675`
**Config Path:** Use relative: `"datasets/tifolora"`

---

## Troubleshooting

### "Volume not available when creating pod"
- Check you're deploying in the **same datacenter** as the volume
- Volume must be in "Ready" state

### "No such file or directory" error during training
- Verify files are in `/workspace/ai-toolkit/datasets/tifolora/`
- Run: `ls -la /workspace/ai-toolkit/datasets/tifolora/`
- Check config uses relative path: `"datasets/tifolora"`

### "Cannot access /workspace"
- Volume mount path must be exactly `/workspace`
- Check pod was created with volume attached
- Run `df -h` to see mounted volumes

### Files disappeared after restarting pod
- Files in pod storage (not network volume) are lost on restart
- Always work in `/workspace/` to use network volume
- Move important files: `mv /root/file /workspace/`


















































































                    