# ConfineClaw-Releases

Release artifacts and operational notes for ConfineClaw.

## Configure OpenClaw

### Models

#### Online Model

Enter the container:
```bash
ctr -n default task exec -t --exec-id configure gateway /bin/bash
```

You can also configure the model and API_KEY (obtain it from the provider) without entering the container:
```bash
ctr -n default run --rm -t --net-host \
  --env OPENCLAW_STATE_DIR=/home/node/state \
  --mount type=bind,src=/root/openclaw/state,dst=/home/node/state,options=rbind:rw \
  --mount type=bind,src=/root/openclaw/workspace,dst=/home/node/workspace,options=rbind:rw \
  ghcr.io/openclaw/openclaw:latest init \
  /bin/sh -c '
  set -e
  openclaw config set env.OPENROUTER_API_KEY "API_KEY"
  openclaw config set agents.defaults.model.primary "openrouter/MODEL_ID"
  '
```

#### Local Model

##### Ollama

Start the Ollama service:
```bash
OLLAMA_HOST=0.0.0.0 OLLAMA_KEEP_ALIVE=1h CUDA_VISIBLE_DEVICES=0 ollama serve
```

Configure the Ollama model:
```bash
ctr -n default run --rm -t --net-host \
  --env OPENCLAW_STATE_DIR=/home/node/state \
  --mount type=bind,src=/root/openclaw/state,dst=/home/node/state,options=rbind:rw \
  --mount type=bind,src=/root/openclaw/workspace,dst=/home/node/workspace,options=rbind:rw \
  ghcr.io/openclaw/openclaw:latest init \
  /bin/sh -c '
  set -e

  cat <<EOF > /tmp/batch.json
[
  {
    "path": "models.providers.ollama",
    "value": {
      "api": "ollama",
      "apiKey": "ollama-local",
      "baseUrl": "http://172.20.40.79:11434",
      "models": [
        {
          "id": "gemma4:26b",
          "name": "GEMMA4 26B",
          "reasoning": false,
          "input": ["text"],
          "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
          "contextWindow": 16384,
          "maxTokens": 163840
        }
      ]
    }
  },
  {
    "path": "agents.defaults.model.primary",
    "value": "ollama/gemma4:26b"
  }
]
EOF

  openclaw config set --batch-file /tmp/batch.json
  '
```

##### llama.cpp

Start the llama.cpp service:
```bash
CUDA_VISIBLE_DEVICES=0 ./llama-server --model /data3/buffalo/models/unsloth/Qwen3.5-27B-GGUF/Qwen3.5-27B-UD-Q4_K_XL.gguf --host 0.0.0.0 --port 9090 --reasoning off
```

Configure the llama.cpp model:
```bash
ctr -n default run --rm -t --net-host \
  --env OPENCLAW_STATE_DIR=/home/node/state \
  --mount type=bind,src=/root/openclaw/state,dst=/home/node/state,options=rbind:rw \
  --mount type=bind,src=/root/openclaw/workspace,dst=/home/node/workspace,options=rbind:rw \
  ghcr.io/openclaw/openclaw:latest init \
  /bin/sh -c '
  set -e

  cat <<EOF > /tmp/batch.json
[
  {
    "path": "models.providers.llamacpp",
    "value": {
      "api": "openai-completions",
      "apiKey": "sk-local",
      "baseUrl": "http://172.20.40.79:9090/v1",
      "models": [
        {
          "id": "Qwen3.5-27B-UD-Q4_K_XL.gguf",
          "name": "Qwen3.5 27B GGUF",
          "reasoning": false,
          "input": ["text"],
          "cost": {"input": 0, "output": 0, "cacheRead": 0, "cacheWrite": 0},
          "contextWindow": 131072,
          "maxTokens": 8192
        }
      ]
    }
  },
  {
    "path": "agents.defaults.model.primary",
    "value": "llamacpp/Qwen3.5-27B-UD-Q4_K_XL.gguf"
  }
]
EOF

  openclaw config set --batch-file /tmp/batch.json
  '
```

### Channels

#### Feishu

Set up the channel:
```bash
ctr -n default run --rm -t --net-host \
  --env OPENCLAW_STATE_DIR=/home/node/state \
  --mount type=bind,src=/root/openclaw/state,dst=/home/node/state,options=rbind:rw \
  --mount type=bind,src=/root/openclaw/workspace,dst=/home/node/workspace,options=rbind:rw \
  ghcr.io/openclaw/openclaw:latest init \
  /bin/sh -c '
  set -e
  openclaw config set channels.feishu.enabled true --json
  openclaw config set channels.feishu.defaultAccount "main"
  openclaw config set channels.feishu.dmPolicy "pairing"
  openclaw config set channels.feishu.accounts.main.appId ""
  openclaw config set channels.feishu.accounts.main.appSecret ""
  openclaw config set channels.feishu.accounts.main.botName ""
  '
```

Approve the pairing request sent by the Feishu bot:
```bash
ctr -n default run --rm -t --net-host \
  --env OPENCLAW_STATE_DIR=/home/node/state \
  --mount type=bind,src=/root/openclaw/state,dst=/home/node/state,options=rbind:rw \
  --mount type=bind,src=/root/openclaw/workspace,dst=/home/node/workspace,options=rbind:rw \
  ghcr.io/openclaw/openclaw:latest init \
  /bin/sh -c '
  openclaw pairing approve feishu <CODE>
  '
```

### Tools and Plugins

#### Cron

Add a cron job:
```bash
openclaw cron add \
  --name "data_pipeline" \
  --every 1h \
  --message "Run the data_pipeline skill to process new input data." \
  --session isolated \
  --timeout-seconds 3600 \
  --announce \
  --channel feishu \
  --to "USER_ID"
```

#### Plugins

Place custom plugins in the `extensions` directory, then set permissions:
```bash
chmod 755 extensions
find extensions -type d -exec chmod 755 {} \;
find extensions -type f -exec chmod 644 {} \;
find extensions -name "*.py" -exec chmod 755 {} \;
```

Then copy `extensions` into the root filesystem:
```bash
mkdir /tmp/rootfs
sudo mount ./vm/rootfs.ext4 /tmp/rootfs/
sudo cp -r extensions /tmp/rootfs/root/openclaw/state
sudo chown -R 1000:1000 /tmp/rootfs/root/openclaw/state/extensions
sudo umount /tmp/rootfs
```

### Reincarnation / State Migration

The soul of OpenClaw is stored under `/root/openclaw`. When upgrading the body, you can transfer the soul from the old body to the new body and continue its life.
```bash
mount /path/to/old/rootfs.ext4 /tmp/rootfs_prev
mount /path/to/new/rootfs.ext4 /tmp/rootfs
sudo rsync -av /tmp/rootfs_prev/root/openclaw/ /tmp/rootfs/root/openclaw/
sudo rm /tmp/rootfs/root/openclaw/state/.initialized
umount /tmp/rootfs_prev
umount /tmp/rootfs
```
