# ollama

## Windows

### [自定义Ollama安装路径](https://www.cnblogs.com/LaiYun/p/18696931)

```
OllamaSetup.exe /DIR=E:\MySoftware\Ollama
```

### 手动创建大模型存储目录

首先先在你想要存储的盘符下创建好存储目录，比如我想存到E盘下面，我这里就创建了该目录：

`E:\MySoftware\Ollama`

增加环境变量：`OLLAMA_MODELS`

内容指向：`E:\MySoftware\Ollama\models`

## Linux

https://github.com/ollama/ollama/blob/main/docs/linux.md

### **Install**

To install Ollama, run the following command:

```
curl -fsSL https://ollama.com/install.sh | sh
```

### **Manual install**

Note

If you are upgrading from a prior version, you should remove the old libraries with `sudo rm -rf /usr/lib/ollama` first.

Download and extract the package:

```
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
```

Start Ollama:

```
ollama serve
```

In another terminal, verify that Ollama is running:

```
ollama -v
```

**AMD GPU install**

If you have an AMD GPU, also download and extract the additional ROCm package:

```
curl -L https://ollama.com/download/ollama-linux-amd64-rocm.tgz -o ollama-linux-amd64-rocm.tgz
sudo tar -C /usr -xzf ollama-linux-amd64-rocm.tgz
```

**ARM64 install**

Download and extract the ARM64-specific package:

```
curl -L https://ollama.com/download/ollama-linux-arm64.tgz -o ollama-linux-arm64.tgz
sudo tar -C /usr -xzf ollama-linux-arm64.tgz
```

**Adding Ollama as a startup service (recommended)**

Create a user and group for Ollama:

```
sudo useradd -r -s /bin/false -U -m -d /usr/share/ollama ollama
sudo usermod -a -G ollama $(whoami)
```

Create a service file in `/etc/systemd/system/ollama.service`:

```
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=$PATH"

[Install]
WantedBy=default.target
```

Then start the service:

```
sudo systemctl daemon-reload
sudo systemctl enable ollama
```

**Install CUDA drivers (optional)**

[Download and install](https://developer.nvidia.com/cuda-downloads) CUDA.

Verify that the drivers are installed by running the following command, which should print details about your GPU:

```
nvidia-smi
```

**Install AMD ROCm drivers (optional)**

[Download and Install](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/quick-start.html) ROCm v6.

**Start Ollama**

Start Ollama and verify it is running:

```
sudo systemctl start ollama
sudo systemctl status ollama
```

Note

While AMD has contributed the `amdgpu` driver upstream to the official linux kernel source, the version is older and may not support all ROCm features. We recommend you install the latest driver from https://www.amd.com/en/support/linux-drivers for best support of your Radeon GPU.

**Customizing**

To customize the installation of Ollama, you can edit the systemd service file or the environment variables by running:

```
sudo systemctl edit ollama
```

Alternatively, create an override file manually in `/etc/systemd/system/ollama.service.d/override.conf`:

```
[Service]
Environment="OLLAMA_DEBUG=1"
```

**Updating**

Update Ollama by running the install script again:

```
curl -fsSL https://ollama.com/install.sh | sh
```

Or by re-downloading Ollama:

```
curl -L https://ollama.com/download/ollama-linux-amd64.tgz -o ollama-linux-amd64.tgz
sudo tar -C /usr -xzf ollama-linux-amd64.tgz
```

**Installing specific versions**

Use `OLLAMA_VERSION` environment variable with the install script to install a specific version of Ollama, including pre-releases. You can find the version numbers in the [releases page](https://github.com/ollama/ollama/releases).

For example:

```
curl -fsSL https://ollama.com/install.sh | OLLAMA_VERSION=0.5.7 sh
```

**Viewing logs**

To view logs of Ollama running as a startup service, run:

```
journalctl -e -u ollama
```

**Uninstall**

Remove the ollama service:

```
sudo systemctl stop ollama
sudo systemctl disable ollama
sudo rm /etc/systemd/system/ollama.service
```

Remove the ollama binary from your bin directory (either `/usr/local/bin`, `/usr/bin`, or `/bin`):

```
sudo rm $(which ollama)
```

Remove the downloaded models and Ollama service user and group:

```
sudo rm -r /usr/share/ollama
sudo userdel ollama
sudo groupdel ollama
```

Remove installed libraries:

```
sudo rm -rf /usr/local/lib/ollama
```











# Open WebUI

https://github.com/open-webui/open-webui

## standalone：

Open WebUI can be installed using pip, the Python package installer. Before proceeding, ensure you're using **Python 3.11** to avoid compatibility issues.

1. **Install Open WebUI**: Open your terminal and run the following command to install Open WebUI:

   ```
   pip install open-webui
   ```

2. **Running Open WebUI**: After installation, you can start Open WebUI by executing:

   ```
   open-webui serve
   ```

This will start the Open WebUI server, which you can access at [http://localhost:8080](http://localhost:8080/)

## docker：

### Installation with Default Configuration

- **If Ollama is on your computer**, use this command:

  ```
  docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
  ```

- **If Ollama is on a Different Server**, use this command:

  To connect to Ollama on another server, change the `OLLAMA_BASE_URL` to the server's URL:

  ```
  docker run -d -p 3000:8080 -e OLLAMA_BASE_URL=https://example.com -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
  ```

- **To run Open WebUI with Nvidia GPU support**, use this command:

  ```
  docker run -d -p 3000:8080 --gpus all --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:cuda
  ```

### Installation for OpenAI API Usage Only



- **If you're only using OpenAI API**, use this command:

  ```
  docker run -d -p 3000:8080 -e OPENAI_API_KEY=your_secret_key -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
  ```

### Installing Open WebUI with Bundled Ollama Support

This installation method uses a single container image that bundles Open WebUI with Ollama, allowing for a streamlined setup via a single command. Choose the appropriate command based on your hardware setup:

- **With GPU Support**: Utilize GPU resources by running the following command:

  ```
  docker run -d -p 3000:8080 --gpus=all -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
  ```

- **For CPU Only**: If you're not using a GPU, use this command instead:

  ```
  docker run -d -p 3000:8080 -v ollama:/root/.ollama -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:ollama
  ```

Both commands facilitate a built-in, hassle-free installation of both Open WebUI and Ollama, ensuring that you can get everything up and running swiftly.

After installation, you can access Open WebUI at [http://localhost:3000](http://localhost:3000/). Enjoy! 😄







