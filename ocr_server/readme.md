# OCR Server – Home Assistant Add-on

An OCR server packaged as a Home Assistant Add-on for image recognition (meters, displays, PV, etc.).  
The server exposes an HTTP API, processes images (OCR / TFLite or tesseract), and can optionally publish results via MQTT.

This repository contains **the Home Assistant Add-on only**, not the core OCR logic itself.  
The OCR logic lives in a separate repository and is included here as a Git submodule.

---

## ✨ Features

- HTTP API (Flask)
- OCR using:
  - Tesseract
  - TFLite (no full TensorFlow dependency)
- Configurable via:
  - **Add-on Options (UI)** for MQTT
  - **YAML file** for OCR layouts & models
- Optimized for HAOS (small image size, no TensorFlow)
- `amd64` architecture supported

---

## 📦 Installation (Home Assistant)

1. **Add the Add-on Repository**

   Home Assistant → Settings → Add-ons → Add-on Store  
   → ⋮ → Repositories  
   → Add repository URL:

   https://github.com/Plasmagun86/ha_repro

2. **Install the OCR Server Add-on**
3. **Configure the Add-on Options**
4. **Start the Add-on**

---

## ⚙️ Configuration

### Add-on Options (UI)

These values are configured via the Add-on UI and will automatically override the MQTT settings
in the server configuration file.

| Option | Description |
|------|-------------|
| config_path | Path to the OCR server configuration file |

Default value for config_path:

/config/ocr_server/config.yaml

---

## OCR Server Configuration File

The OCR logic is configured via a YAML file on the Home Assistant system:

/config/ocr_server/config.yaml

This file typically contains:

- OCR models
- Image regions (rects)
- Regex matching rules
- Transformations

---

## Repository Structure
```
ha_repro/
├── repository.yaml
└── ocr_server/
    ├── config.yaml        (Add-on manifest)
    ├── Dockerfile
    ├── start.sh
    └── src/
        └── ocr-server/    (OCR logic, submodule)
```
---

## Configuration Flow

1. Home Assistant writes Add-on options to:
   /data/options.json

2. start.sh:
   - reads options.json
   - loads /config/ocr_server/config.yaml
   - overrides only the mqtt block
   - writes final config to /app/config/config.yaml

3. The OCR server starts with the final configuration.

---

## Quick Test

Add-on logs may show the parsed options on startup.

```
curl -s -X POST http://<HA-IP>:5000/segment -F "identifier=watermeter" -F "image=@t4.jpg"
{"identifier":"watermeter","results":[{"id":"d1","value":"00016.0"},{"id":"a1","value":9.127448058121708},{"id":"a2","value":3.881748198574262},{"id":"a3","value":1.2311287412814798},{"id":"a4","value":3.345734090629815},{"id":"t1","value":"CH-MI001-10039"}]}
```

see also add-on log for receiving and handling the request. Use the t4.jpg from the ocr-server repo.

HTTP API test:
curl http://<HA-IP>:5000/

---

## Local Development

```
docker build -t ocr-addon-test ./ocr_server
docker run --rm -p 5000:5000 -v ./local_config:/config:ro ocr-addon-test
```

---

## Development Status

- HAOS add-on/app build
- HAOS configuration to set mqtt and config.yaml path (for picture segments)
- changes in ocr-server to add HA discovery for MQTT values
- BEWARE: still under preparation
  (complete testflow for example picture was up and running)

---

## License

See the respective repositories for license details.

---

## Contributing

Issues and pull requests are welcome.
Changes to OCR logic must be made in the submodule repository.
