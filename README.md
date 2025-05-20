# IOT_Threat_Protector
This project uses an ESP32 microcontroller to collect temperature and humidity data from a DHT11 sensor and send it to a Django server for storage and display on a user dashboard. The ESP32 connects to a WiFi network, syncs time with an NTP server, and posts sensor data to a REST API endpoint every 15 seconds.

## Table of Contents

- Features
- Hardware Requirements
- Software Requirements
- Setup Instructions
  - Hardware Setup
  - ESP32 Firmware
  - Django Server
- Configuration
- Usage
- Troubleshooting
- License

## Features

- Collects temperature and humidity data using a DHT11 sensor.
- Connects to a WiFi network for internet access.
- Syncs time with an NTP server for accurate timestamps.
- Sends sensor data to a Django server via a REST API with token authentication.
- Retries WiFi connection and handles network errors gracefully.
- Configurable time zone offset (default: UTC+5:30 for India).
- Alert User and admin through email and disconnect the device from the Wifi-Networks if any Cyber attack happen

## Hardware Requirements

- ESP32 development board (e.g., ESP32-WROOM-32).
- DHT11 temperature and humidity sensor.
- Jumper wires.
- Breadboard (optional).
- USB cable for programming the ESP32.

## Software Requirements

- **ESP32**:
  - MicroPython firmware installed on the ESP32.
  - Python libraries
- **Django Server**:
  - Python 3.x with Django and Django REST Framework.
  - `rest_framework.authtoken` for token authentication.
  - `django-cors-headers` (optional, for CORS support).
- **Development Tools**:
  - Thonny, uPyCraft, or another MicroPython IDE for uploading code to the ESP32.
  - Postman or `curl` for testing the API.

## Setup Instructions

### Hardware Setup

1. Connect the DHT11 sensor to the ESP32:
   - **VCC**: 3.3V or 5V pin on ESP32.
   - **GND**: GND pin on ESP32.
   - **Data**: GPIO 4 (configurable in code via `DHT_PIN`).
   - Add a 4.7k–10k pull-up resistor between VCC and the Data pin if not built into the DHT11 module.
2. Power the ESP32 via USB or an external power source.

### ESP32 Firmware

1. Install MicroPython on the ESP32:

   - Download the latest MicroPython firmware from micropython.org.
   - Flash the firmware using `esptool.py` or a similar tool.

2. Install required libraries:

   - Ensure `dht` and `ntptime` are available (usually included in MicroPython builds).

3. Upload the main script (`main.py`) to the ESP32 using Thonny 

### Django Server

1. Set up a Django project with Django REST Framework:

   ```bash
   pip install django djangorestframework django-cors-headers
   ```

2. Configure `settings.py`:

   ```python
   INSTALLED_APPS = [
       ...,
       'rest_framework',
       'rest_framework.authtoken',
       'corsheaders',
   ]
   
   REST_FRAMEWORK = {
       'DEFAULT_AUTHENTICATION_CLASSES': [
           'rest_framework.authentication.TokenAuthentication',
       ],
       'DEFAULT_PERMISSION_CLASSES': [
           'rest_framework.permissions.IsAuthenticated',
       ],
   }
   
   MIDDLEWARE = [
       ...,
       'corsheaders.middleware.CorsMiddleware',
       'django.middleware.common.CommonMiddleware',
       ...,
   ]
   
   CORS_ALLOW_ALL_ORIGINS = True  # Restrict in production
   ```

3. Create a view for the `/api/sensor-data/` endpoint (example in `views.py`):

   ```python
   from rest_framework.decorators import api_view, authentication_classes, permission_classes
   from rest_framework.authentication import TokenAuthentication
   from rest_framework.permissions import IsAuthenticated
   from rest_framework.response import Response
   
   @api_view(['POST'])
   @authentication_classes([TokenAuthentication])
   @permission_classes([IsAuthenticated])
   def sensor_data(request):
       data = request.data
       # Process and save data to the database
       return Response({"message": "Data received"}, status=201)
   ```

4. Map the URL in `urls.py`:

   ``` python
   from django.urls import path
   from . import views
   
   ```

5. Generate an API token for the ESP32:

   Copy the token for use in the ESP32 code.

## Configuration

Edit the following variables in `main.py`:

- `WIFI_SSID`: Your WiFi network name (e.g., `"Pass@2025"`).
- `WIFI_PASS`: Your WiFi password (e.g., `"2025@Password"`).
- `SERVER_URL`: Your Django server URL (e.g., `"http://Use Ipconfig:8000"`).
- `API_TOKEN`: The token from the Django server (e.g., `"a847caa237ceec51f1ba79bbac6d2266fb65ba1b"`).
- `DEVICE_ID`: Unique identifier for the ESP32 (e.g., `"ESP001"`).
- `DHT_PIN`: GPIO pin for the DHT11 sensor (default: `4`).
- `TIMEZONE_OFFSET`: Time zone offset in seconds (e.g., `19800` for UTC+5:30).

## Usage

1. Power on the ESP32 and ensure it’s connected to the same network as the Django server.

2. Run the Django server:

   ```bash
   python manage.py runserver 0.0.0.0:8000
   ```

3. The ESP32 will:

   - Connect to WiFi.
   - Sync time with `pool.ntp.org`.
   - Read DHT11 sensor data every 15 seconds.
   - Send data to `/api/sensor-data/` with the configured token.

4. Monitor the ESP32’s serial output (e.g., via Thonny) for debug messages.

5. Check the Django server logs or database for received data.

## Troubleshooting

- **ESP32 fails to connect to WiFi**:

  - Verify `WIFI_SSID` and `WIFI_PASS`.
  - Ensure the ESP32 is within WiFi range.

- **Authentication fails (403 error)**:

  - Confirm the `API_TOKEN` matches the one in Django’s admin panel.

  - Test the endpoint with Postman:

  - Check Django’s `settings.py` and view for correct authentication settings.

- **No data received on server**:

  - Ensure the server is running and accessible at `Use Ipconfig:8000`.
  - Verify CORS settings if cross-origin issues arise.

- **Invalid sensor readings**:

  - Check DHT11 wiring and pull-up resistor.
  - Ensure `DHT_PIN` matches the GPIO used.

## License

This project is licensed under the MIT License. See the LICENSE file for details.
