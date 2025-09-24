# MiniIOT - Real-Time IoT Dashboard

A comprehensive IoT data collection and visualization system built with Laravel 12, designed to receive and display real-time data from Arduino Nano 33 IoT and ESP32 devices via TCP connections.

## 🚀 Features

### ✅ **Dual Device Support**

-   **Arduino Nano 33 IoT**: Accelerometer data (X, Y, Z axes) using built-in LSM6DS3 sensor
-   **ESP32**: Temperature and humidity monitoring using DHT22 sensor
-   Automatic device type detection based on data format
-   Real-time device status tracking (online/offline)

### ✅ **Real-Time Data Processing**

-   **TCP Server**: Receives data from both devices on port 9000
-   **Background Processing**: Automatic log processing every second
-   **JSON Data Handling**: Robust parsing with trailing comma tolerance
-   **Duplicate Prevention**: Smart device creation based on IP + device type

### ✅ **Interactive Dashboard**

-   **Live Sensor Cards**: Real-time temperature, humidity, and accelerometer readings
-   **Chart.js Visualizations**: Interactive charts with real-time updates
-   **Auto-Refresh**: Dashboard updates every 5 seconds automatically
-   **Device Status**: Live online/offline indicators
-   **Responsive Design**: Works on desktop and mobile devices

### ✅ **Advanced Architecture**

-   **Separate Device Logic**: ESP32 (temperature/humidity) + Arduino Nano (accelerometer)
-   **Smart Data Processing**: Only creates sensor readings for data actually sent
-   **Background Services**: Continuous TCP server + log processing
-   **RESTful API**: Clean endpoints for data retrieval and device management

## 🏗️ System Architecture

```
┌─────────────────┐    TCP:9000     ┌──────────────────┐
│   Arduino Nano  │ ──────────────► │                  │
│  (Accelerometer)│                 │   Laravel TCP    │
└─────────────────┘                 │     Server       │
                                    │   (server.php)   │
┌─────────────────┐    TCP:9000     │                  │
│      ESP32      │ ──────────────► │                  │
│  (Temp/Humidity)│                 └──────────────────┘
└─────────────────┘                           │
                                             │ Log Files
                                             ▼
                                   ┌──────────────────┐
                                   │  Background      │
                                   │  Processing      │
                                   │ (process-logs)   │
                                   └──────────────────┘
                                             │
                                             ▼ Database
                                   ┌──────────────────┐
                                   │   Dashboard      │
                                   │   (Chart.js)     │
                                   │  Real-time UI    │
                                   └──────────────────┘
```

## 📊 Current Data Flow

### Arduino Nano 33 IoT → TCP

```json
{ "x": 0.011, "y": -0.027, "z": 1.015 }
```

-   **Frequency**: Every 0.5 seconds
-   **Sensors**: Built-in LSM6DS3 accelerometer
-   **Connection**: WiFiNINA to TCP port 9000

### ESP32 → TCP

```json
{ "temperature": 20.8, "humidity": 58.6, "device_type": "esp32" }
```

-   **Frequency**: Every 5 seconds
-   **Sensors**: DHT22 temperature/humidity sensor
-   **Connection**: WiFi to TCP port 9000

## 🔧 Installation & Setup

### 1. Laravel Application

```bash
# Navigate to project directory
cd d:\Work\MiniIOT

# Install dependencies (if needed)
composer install

# Set up database
php artisan migrate

# Start Laravel Herd or development server
# Dashboard available at: http://miniiot.test
```

### 2. Start Background Services

#### Option A: Start TCP Server

```bash
php server.php
```

#### Option B: Start Background Log Processing

```powershell
# Run the PowerShell script for continuous processing
.\process-logs-loop.ps1
```

#### Option C: Use Laravel Scheduler (Alternative)

```bash
php artisan schedule:work
```

### 3. Hardware Setup

#### Arduino Nano 33 IoT Code

```cpp
#include <WiFiNINA.h>
#include <Arduino_LSM6DS3.h>

const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";
const char* serverHost = "192.168.1.100";  // Your server IP
const int serverPort = 9000;

// Complete code available in: Simple_Nano_Accelerometer.ino
```

#### ESP32 Code

```cpp
#include <WiFi.h>
#include <DHT.h>

#define DHT_PIN 4
#define DHT_TYPE DHT22

const char* ssid = "YOUR_WIFI_SSID";
const char* password = "YOUR_WIFI_PASSWORD";
const char* tcpServer = "192.168.1.100";  // Your server IP
const int tcpPort = 9000;

// Complete code available in: ESP32_TCP_Temperature_Humidity.ino
```

## 📡 API Endpoints

### Live Data (Dashboard)

```http
GET /api/live-data
```

**Response:**

```json
{
    "temperature": "20.80",
    "humidity": "58.60",
    "x": "0.011",
    "y": "-0.027",
    "z": "1.015",
    "timestamp": "20:10:21"
}
```

### Chart Data

```http
GET /api/chart-data?sensor_type=temperature&hours=24
```

### Device Status

```http
GET /api/devices
```

## 🗄️ Database Structure

### Devices Table

```sql
- id (Primary Key)
- name (Device Name)
- type ('esp32' | 'arduino_nano')
- ip_address (Current IP)
- status ('online' | 'offline')
- last_seen_at (Last Communication)
- location (Physical Location)
- created_at, updated_at
```

### Sensor Readings Table

```sql
- id (Primary Key)
- device_id (Foreign Key → devices.id)
- sensor_type ('temperature' | 'humidity' | 'accelerometer_x' | 'accelerometer_y' | 'accelerometer_z')
- value (Sensor Reading)
- unit ('°C' | '%' | 'm/s²')
- created_at (Reading Timestamp)
```

## 🏃‍♂️ Running the Complete System

### 1. Start All Services

```bash
# Terminal 1: Start TCP Server
php server.php

# Terminal 2: Start Background Processing
powershell -File process-logs-loop.ps1

# Terminal 3: Access Dashboard
# Open browser: http://miniiot.test
```

### 2. Power On Devices

-   **Arduino Nano**: Will start sending accelerometer data every 0.5s
-   **ESP32**: Will start sending temperature/humidity every 5s
-   **Dashboard**: Will show both devices as "online" within seconds

### 3. Monitor Live Data

-   Dashboard auto-refreshes every 5 seconds
-   Real-time charts update automatically
-   Device status indicators show live connection status

## 🔧 Configuration Files

### Key Files in Project:

-   `server.php` - TCP server receiving device data
-   `process-logs-loop.ps1` - Background log processing script
-   `app/Console/Commands/ProcessLogDataCommand.php` - Data processing logic
-   `app/Http/Controllers/DashboardController.php` - Dashboard API
-   `resources/views/dashboard.blade.php` - Real-time dashboard UI

## 🚨 Troubleshooting

### 1. Dashboard Not Updating

-   ✅ **Solution**: Ensure background processing is running
-   Check: `Get-Process | Where-Object {$_.ProcessName -like "*powershell*"}`
-   Restart: `.\process-logs-loop.ps1`

### 2. Device Shows Offline

-   ✅ **Check**: TCP server is running on port 9000
-   ✅ **Verify**: Device can reach server IP address
-   ✅ **Test**: `netstat -an | findstr ":9000"`

### 3. Only One Device Working

-   ✅ **Cause**: Trailing comma in JSON (ESP32) - **Fixed**
-   ✅ **Cause**: Default sensor values overwriting data - **Fixed**
-   ✅ **Solution**: Updated ProcessLogDataCommand to handle device-specific sensors

### 4. JSON Parsing Errors

-   ✅ **Fixed**: Server now handles trailing commas automatically
-   ✅ **Fixed**: Robust JSON validation and error logging

## ⚡ Performance Stats

-   **Data Processing**: ~1000+ sensor readings processed seamlessly
-   **Real-time Updates**: Dashboard refreshes every 5 seconds
-   **Device Response**: Sub-second data processing
-   **Background Processing**: Handles 515+ log entries per batch
-   **Concurrent Devices**: 2 devices sending data simultaneously without conflicts

## 🎯 Current Status: ✅ FULLY OPERATIONAL

-   ✅ **Arduino Nano**: Sending accelerometer data via TCP
-   ✅ **ESP32**: Sending temperature/humidity via TCP
-   ✅ **TCP Server**: Receiving and logging all data
-   ✅ **Background Processing**: Running automatically
-   ✅ **Dashboard**: Displaying real-time data from both devices
-   ✅ **Device Management**: Both devices showing as online
-   ✅ **API**: Returning live data with current timestamps

## 🔮 Future Enhancements

-   **HTTP Endpoint**: Alternative to TCP for Arduino Nano
-   **Device Authentication**: MAC address based device security
-   **Data Export**: CSV/JSON export functionality
-   **Historical Analysis**: Long-term data trends and analytics
-   **Mobile App**: React Native companion app
-   **Alerting System**: Email/SMS notifications for sensor thresholds

## 📝 License

This project is open source and available under the [MIT License](LICENSE).


