# AI Toolkit Setup for Runpod

## Step 1: Create a Runpod Pod

1. Go to [Runpod.io](https://runpod.io)
2. Create a new GPU pod (recommend 24GB+ VRAM for Qwen Image training)
3. **Choose template:** PyTorch or CUDA (or use custom Docker image: `ostris/aitoolkit:latest`)

## Step 2: Network Volume Setup (Recommended)

### Option A: Using Network Volume (Persistent Storage)
1. Create a Network Volume in Runpod (if you haven't already)
2. Attach it to your pod
3. **Volume Mount Path:** `/workspace`
4. Your data will persist between pod sessions

### Option B: Using Pod Storage (Temporary)
- Data is stored on the pod itself
- Will be lost when pod is terminated
- Faster but not persistent

## Step 3: Upload Your Files to Runpod

### Via Runpod Web Terminal:
```bash
cd /workspace
git clone https://github.com/ostris/ai-toolkit.git
cd ai-toolkit
```

### Or upload your dataset manually:
```bash
# Create dataset directory
mkdir -p /workspace/ai-toolkit/datasets/tifolora

# Upload files using Runpod's file manager or via scp/rsync
```

## Step 4: Copy Your Dataset

If you're using the network volume, copy your `tifolora` dataset to:
```
/workspace/ai-toolkit/datasets/tifolora/
```

Your directory structure should be:
```
/workspace/ai-toolkit/
├── datasets/
│   └── tifolora/
│       ├── 1.jpg
│       ├── 1.txt
│       ├── 2.jpg
│       ├── 2.txt
│       └── ... (all your images and captions)
├── config/
│   └── examples/
│       └── train_lora_qwen_image_24gb.yaml
└── output/
```

## Step 5: Install AI Toolkit (if not using Docker image)

```bash
cd /workspace/ai-toolkit
pip install -r requirements.txt
```

## Step 6: Configure Your Training

Your config file should use **relative paths**:

```yaml
datasets:
  - folder_path: "datasets/tifolora"  # ✓ Correct
    # NOT: "C:\\Users\\..." ✗ Wrong for Runpod
```

## Step 7: Run Training

### If using the ostris/aitoolkit Docker image:
The UI should be available at `http://your-pod-ip:8675`

### If running directly:
```bash
cd /workspace/ai-toolkit
python run.py config/examples/train_lora_qwen_image_24gb.yaml
```

## Port Forwarding

If using the web UI, expose port 8675:
- Runpod will automatically expose it
- Access via: `https://your-pod-id-8675.proxy.runpod.net`

## Tips

1. **Use relative paths** in all configs (e.g., `datasets/tifolora` not absolute Windows paths)
2. **Save outputs to network volume** so you don't lose your trained models
3. **Monitor GPU usage** in Runpod dashboard
4. **Stop pod when not training** to save credits
5. **Download your models** from `/workspace/ai-toolkit/output/` when done

## Troubleshooting

### "No such file or directory" error:
- Check your files are in `/workspace/ai-toolkit/datasets/tifolora/`
- Verify config uses relative path: `"datasets/tifolora"`
- Run `ls -la /workspace/ai-toolkit/datasets/tifolora/` to confirm files exist

### Out of memory:
- Reduce batch size
- Ensure quantization is enabled
- Use a pod with more VRAM (24GB minimum recommended)
