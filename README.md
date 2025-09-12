# Parakeet-Silero Wyoming Wrapper

A high-performance Wyoming Protocol ASR server combining **NVIDIA Parakeet v3** speech recognition with **Silero VAD** for intelligent voice activity detection. Designed as a drop-in replacement for Faster-Whisper in Home Assistant voice pipelines.

## ✨ Features

- 🚀 **35x Faster**: ~0.06s inference after model loading (vs 20-30s for Whisper)
- 🧠 **Intelligent VAD**: Silero VAD filters noise and stops listening intelligently
- 🌍 **25 Languages**: Hungarian, English, German, French, Spanish, Italian, Polish, Dutch, Czech, Slovak, and 15+ more
- 🔌 **Drop-in Replacement**: Full Wyoming Protocol compatibility for Home Assistant
- 💾 **Low VRAM**: Only 2.8GB vs 6-8GB for Whisper Large
- 🎯 **Production Ready**: Systemd service, auto-restart, log rotation

## 🏠 Perfect for Home Assistant

This wrapper provides a complete Wyoming ASR service that Home Assistant can discover and use just like the official Whisper containers, but with dramatically better performance for multilingual voice commands.

## 📊 Performance Comparison

| Metric | Faster-Whisper | Parakeet Wyoming Wrapper |
|--------|----------------|---------------------------|
| **First Request** | ~20-30s | ~20s (model loading) |
| **Subsequent Requests** | ~2-5s | **~0.06s** ⚡ |
| **VRAM Usage** | 6-8GB | **2.8GB** 💾 |
| **False Transcriptions** | Many (processes all audio) | **Few** (VAD filtered) 🎯 |
| **Languages** | 100+ | **25 European** (optimized) 🌍 |
| **Home Assistant Discovery** | ✅ | **✅ (Fixed protocol compliance)** |

## 🚀 Quick Start

### Prerequisites
- Python 3.8+
- NVIDIA GPU with CUDA support
- 4GB+ VRAM available
- Linux/Ubuntu system

### Installation

1. **Clone the repository**:
```bash
git clone <your-repo-url>
cd parakeet-wyoming-wrapper
```

2. **Install dependencies**:
```bash
pip install -r requirements.txt
```

3. **Test the service**:
```bash
chmod +x manage.sh
./manage.sh test
```

4. **Start the service**:
```bash
./manage.sh start
```

The service will be available at `tcp://localhost:10300` and compatible with Home Assistant Wyoming Protocol integration.

## ⚙️ Configuration

Edit `config.yaml` to customize:

```yaml
# ASR Backend ('parakeet' recommended for speed)
asr_backend: 'parakeet'

# VAD Settings (higher = more strict filtering)
vad_threshold: 0.7
buffer_timeout: 2.0

# Performance Tuning
performance:
  max_audio_length: 8.0  # Seconds
  enable_caching: true
```

## 🛠️ Management Commands

```bash
# Basic Operations
./manage.sh start           # Start the server
./manage.sh stop            # Stop the server  
./manage.sh restart         # Restart the server
./manage.sh status          # Show status
./manage.sh test            # Test Wyoming protocol

# Production Setup
./manage.sh install-service # Install systemd service (auto-start/restart)
./manage.sh setup-logs      # Configure log rotation and cleanup

# Maintenance
./manage.sh cleanup-logs    # Clean old logs
./manage.sh uninstall-service # Remove systemd service
```

## 🏠 Home Assistant Integration

### Add to Home Assistant

1. **Settings** → **Devices & Services** → **Add Integration**
2. Search for **"Wyoming Protocol"**
3. Configure:
   - **Host**: Your server IP (e.g., `10.40.80.175`)
   - **Port**: `10300`

### Configuration.yaml Example

```yaml
# Speech-to-Text with Parakeet + VAD
stt:
  - platform: wyoming
    uri: tcp://10.40.80.175:10300

# Text-to-Speech (use your existing TTS)
tts:
  - platform: wyoming
    uri: tcp://10.40.80.175:10200

# Wake Word Detection (use your existing wake word service)
wake_word:
  - platform: wyoming
    uri: tcp://10.40.80.175:10400
```

## 🧪 Testing

### Protocol Compliance Test
```bash
./manage.sh test
```

### Manual Testing
```bash
# Test service discovery
python3 wyoming_test_client.py --uri tcp://localhost:10300

# Test with audio file
python3 wyoming_test_client.py --uri tcp://localhost:10300 --audio-file test.wav
```

## 🔧 Advanced Configuration

### Systemd Service (Recommended for Production)

```bash
# Install as system service
./manage.sh install-service

# Manage with systemctl
sudo systemctl status parakeet-wyoming
sudo systemctl restart parakeet-wyoming
journalctl -u parakeet-wyoming -f
```

### GPU Optimization

For even better performance, install CUDA Python:
```bash
pip install cuda-python>=12.3
```

### Log Management

```bash
# Setup automatic log rotation and cleanup
./manage.sh setup-logs

# Manually clean logs
./manage.sh cleanup-logs
```

## 📁 Architecture

```
Audio Input → Silero VAD → Speech Detection → Parakeet v3 ASR → Wyoming Protocol → Home Assistant
```

### Key Components

- **`wyoming_vad_asr_server.py`**: Main Wyoming server with VAD + ASR integration
- **`wyoming_test_client.py`**: Test client for protocol validation
- **`config.yaml`**: Configuration file for all settings
- **`manage.sh`**: Management script for all operations
- **`parakeet-wyoming.service`**: Systemd service definition

## 🌍 Language Support

Optimized for **25 European languages**:
- Hungarian (hu) - Primary
- English (en)
- German (de), French (fr), Spanish (es), Italian (it)
- Polish (pl), Dutch (nl), Czech (cs), Slovak (sk)
- Croatian (hr), Slovenian (sl), Bulgarian (bg)
- Estonian (et), Latvian (lv), Lithuanian (lt), Maltese (mt)
- Danish (da), Swedish (sv), Finnish (fi)
- Greek (el), Romanian (ro)
- Russian (ru), Ukrainian (uk)

## 🐛 Troubleshooting

### Service Not Discovered by Home Assistant

1. Check service status: `./manage.sh status`
2. Test protocol: `./manage.sh test`
3. Verify port 10300 is accessible from Home Assistant
4. Check Home Assistant logs for connection errors

### Slow Performance

1. **Ensure models stay loaded**: Don't restart the service frequently
2. **Adjust VAD settings**: Higher threshold = faster cutoff
3. **Check GPU utilization**: `nvidia-smi`
4. **Install CUDA Python**: `pip install cuda-python>=12.3`

### Audio Not Transcribed

1. **Check VAD sensitivity**: Lower `vad_threshold` in config
2. **Test without VAD**: Set `vad_enabled: false` temporarily
3. **Check audio format**: Service expects 16kHz, mono, 16-bit PCM

## 📈 Performance Tuning

### For Speed (Default Configuration)
```yaml
vad_threshold: 0.7          # Aggressive filtering
buffer_timeout: 2.0         # Quick processing
max_audio_length: 8.0       # Limit long recordings
```

### For Accuracy
```yaml  
vad_threshold: 0.3          # Less filtering
buffer_timeout: 5.0         # More patience
max_audio_length: 15.0      # Allow longer speech
```

### For Low Resource Usage
```yaml
asr_backend: 'parakeet'     # Lower VRAM
performance:
  enable_caching: true      # Cache results
  max_audio_length: 5.0     # Shorter processing
```

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Test thoroughly
4. Submit a pull request

## 📄 License

MIT License - feel free to use in your projects!

## 🙏 Acknowledgments

- **NVIDIA NeMo** - Parakeet v3 ASR model
- **Silero Team** - Voice Activity Detection
- **Rhasspy Project** - Wyoming Protocol specification
- **Home Assistant** - Voice assistant platform

---

## 🚀 Ready to Deploy?

1. `./manage.sh install-service` - Install for production
2. `./manage.sh setup-logs` - Configure log management  
3. Add to Home Assistant Wyoming integration
4. Enjoy 35x faster voice recognition! 🎉
=======
# wyoming-parakeet-silero-wrapper
Wyoming wrapper for Parakeet and Silero - a drop-in replacement for faster-whisper in Home Assistant
