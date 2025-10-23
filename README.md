#  Control de Ruido en Bibliotecas Universitarias

---

##  Descripción del Proyecto

El proyecto **Control de Ruido en Bibliotecas Universitarias** es un sistema IoT diseñado para **monitorear los niveles de ruido en tiempo real** dentro de espacios académicos, principalmente bibliotecas universitarias.  
Su propósito es **detectar niveles sonoros excesivos** y **emitir alertas automáticas**, permitiendo así mantener un ambiente óptimo para el estudio.

Este sistema combina **sensores simulados**, **mensajería MQTT**, **almacenamiento en InfluxDB** y **visualización en Grafana**, logrando una solución integral que refleja cómo se implementaría un entorno real de monitoreo inteligente.

---

##  Objetivos del Proyecto

- Monitorear continuamente los **niveles de ruido (dB)** en tiempo real.  
- Detectar y registrar **eventos de ruido anómalo o prolongado**.  
- Almacenar los datos en una base de tiempo (InfluxDB) para análisis histórico.  
- Mostrar los resultados y alertas en **Grafana** mediante paneles visuales.  
- Permitir **alertas automáticas y configuración dinámica de umbrales**.  

---

##  Arquitectura del Sistema

El sistema se compone de **tres módulos principales**, que trabajan de forma integrada:

1. **Simulador de Sensor de Ruido (`noise_sensor_simulator.py`)**  
   Genera datos sintéticos de niveles de ruido por hora del día y los publica en un **broker MQTT**.

2. **Bridge MQTT (`noise_bridge.py`)**  
   Actúa como **puente entre AWS IoT Core y el broker local** en la instancia EC2, gestionando la conexión segura y bidireccional.

3. **Conector InfluxDB (`mqtt_to_influx.py`)**  
   Suscribe los mensajes MQTT y los almacena en una base de datos **InfluxDB**, permitiendo su posterior análisis en **Grafana**.

   

---

## 🔄 Flujo General de Funcionamiento

1. El **sensor simulado** publica los niveles de ruido (en decibelios) a través de MQTT.  
2. El **bridge MQTT** retransmite los datos al servidor EC2 (o a AWS IoT Core, si se configura).  
3. El **script conector** los almacena en **InfluxDB**, registrando métricas y alertas.  
4. **Grafana** obtiene los datos y los muestra en paneles dinámicos en tiempo real.  

📡 **Flujo de datos:**  
Sensor → MQTT → Bridge → InfluxDB → Grafana  

---

##  Instalación y Configuración

###  Preparación del Entorno (Raspberry Pi o PC Local)

```bash
sudo apt update
sudo apt install mosquitto-clients python3-pip -y
pip3 install paho-mqtt
```
##  Instalación y Configuración

---

### 1️⃣ Preparación del Entorno (Raspberry Pi o PC Local)

```bash
sudo apt update
sudo apt install mosquitto-clients python3-pip -y
pip3 install paho-mqtt
```

##  Crear carpeta para certificados (si usas AWS IoT):
```bash
mkdir ~/certs
# Copiar tus archivos de seguridad:
# root-CA.crt, device-certificate.crt, private-key.key
```

### 2️⃣ Configuración AWS IoT Core (Opcional)

Crear una Thing en AWS IoT Core.

Generar y activar los certificados.

Adjuntar una política con permisos de conexión y publicación.

### 3️⃣ Configuración en Servidor EC2 (AWS)
```bash
Instalar servicios base:

sudo apt update && sudo apt upgrade -y
sudo apt install mosquitto mosquitto-clients -y
```
### Instalar InfluxDB v2

```bash
wget -q https://repos.influxdata.com/influxdata-archive.key
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
sudo apt update
sudo apt install influxdb2 -y

```
###  Instalar Grafana
```bash
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana -y
```
### Iniciar servicios
```bash
sudo systemctl enable mosquitto influxdb grafana-server
sudo systemctl start mosquitto influxdb grafana-server
```

### Configurar InfluxDB
```bash
sudo influx setup
# Usuario: admin
# Password: noise123
# Org: BibliotecaUniv
# Bucket: ruido
```
## Uso del 
###  Inicio Completo

| # | Proceso                    | Comando                         |
|---|-----------------------------|----------------------------------|
| 1 | Bridge MQTT (AWS ↔ EC2)     | `python3 noise_bridge.py`        |
| 2 | Conector InfluxDB           | `python3 mqtt_to_influx.py`      |
| 3 | Simulador de ruido          | `python3 noise_sensor_simulator.py` |

## 📜 Scripts Principales

### 1️⃣ `noise_sensor_simulator.py`

**Características:**
- Simula niveles de ruido por hora del día.  
- Umbrales de alerta configurables.  
- Envío MQTT cada 5 segundos.  
- Detección de picos y ruido continuo.  

**Alertas Implementadas:**

| Tipo de Alerta | Condición | Descripción |
|----------------|------------|--------------|
| 🔴 **RUIDO_EXCESIVO_CONTINUO** | >75 dB durante 30s | Alerta de ruido prolongado |
| 🟠 **PICO_DE_RUIDO** | >85 dB puntual | Pico breve de ruido |
| ⚠️ **ZONA_CRITICA** | >90 dB prolongado | Nivel crítico de ruido |

### 🧪 Ejemplo de Simulación

```python
if random.random() < 0.1:
    ruido = random.randint(85, 95)
else:
    ruido = random.randint(40, 70)
```
---

### 2️⃣ `noise_bridge.py`

**Funcionalidad:**
- Conexión bidireccional entre **AWS IoT** y **Broker Local**.  
- Reconexión automática en caso de fallo.  
- Soporte para múltiples *topics*:  
  - `biblioteca/ruido`  
  - `alertas/ruido`

---

### 3️⃣ `mqtt_to_influx.py`

**Características:**
- Procesa mensajes **MQTT** y los guarda en **InfluxDB**.  
- Registra campos dinámicos:
  - `nivel_ruido`
  - `alerta`
  - `ubicacion`
- Manejo robusto de errores.

---

## 📊 Configuración en Grafana

### 🔧 **Datasource**

| Parámetro | Valor |
|------------|--------|
| **Name** | InfluxDB-Ruido |
| **URL** | `http://localhost:8086` |
| **Organization** | BibliotecaUniv |
| **Bucket** | ruido |
| **Token** | `[tu token de InfluxDB]` |

---

### 📈 **Dashboard Sugerido**

- 🔹 **Nivel de Ruido** (línea en tiempo real)  
- 🔹 **Alertas** (tabla de eventos críticos)  
- 🔹 **Promedio por hora**  
- 🔹 **Alertas activas por zona**

---

### ⚠️ **Umbrales y Lógica de Detección**

| Alerta | Condición | Acción |
|---------|------------|--------|
| 🔴 **RUIDO_EXCESIVO_CONTINUO** | >75 dB por más de 30s | Registrar alerta |
| 🟠 **PICO_DE_RUIDO** | >85 dB puntual | Notificar instantáneamente |
| ⚠️ **ZONA_CRITICA** | >90 dB prolongado | Enviar alerta urgente |

---
---

## 🧠 Ejemplo de Flujo Completo ✅

### 1️⃣ Simulador publica niveles
```bash
python3 noise_sensor_simulator.py
```
[10:05:02] Nivel de ruido: 69 dB | Zona: Biblioteca Central
[10:05:07] Nivel de ruido: 73 dB | Zona: Biblioteca Central
[10:05:12] Nivel de ruido: 87 dB 🟠 PICO_DE_RUIDO
[10:05:17] Nivel de ruido: 92 dB 🔴 ZONA_CRITICA

---

## 🖼️ Ejemplos Visuales


---

### 🔹 Arquitectura del Sistema
![Arquitectura del Sistema](./images/architecture.png)

### 🔹 Dashboard en Grafana
![Dashboard en Grafana](./images/grafana_dashboard.png)

### 🔹 Flujo MQTT Funcionando
![Flujo MQTT Funcionando](./images/mqtt_flow.png)

---

## 🧩 Personalización

En el archivo `noise_sensor_simulator.py` puedes ajustar los parámetros según tus necesidades:

```python
# Probabilidad de ruido alto
if random.random() < 0.15:  # 15% probabilidad

# Umbral de alerta continua
if contador_ruido > 6:  # 6 lecturas seguidas (>75 dB)

```
### Código Fuente — Sistema IoT de Control de Ruido
1️⃣ noise_sensor_simulator.py
```python
import paho.mqtt.client as mqtt
import time
import random
import json

BROKER = "1.124.1.0"  # Cambia por la IP del broker MQTT
PORT = 1883
TOPIC = "biblioteca/ruido"

client = mqtt.Client("SimuladorRuido")
client.connect(BROKER, PORT, 60)

contador_ruido = 0

while True:
    # Simular nivel de ruido (dB)
    if random.random() < 0.15:
        ruido = random.randint(85, 95)
    else:
        ruido = random.randint(40, 70)

    alerta = None
    contador_ruido = contador_ruido + 1 if ruido > 75 else 0

    if ruido > 90:
        alerta = "ZONA_CRITICA"
    elif ruido > 85:
        alerta = "PICO_DE_RUIDO"
    elif contador_ruido > 6:
        alerta = "RUIDO_EXCESIVO_CONTINUO"

    payload = {
        "nivel_ruido": ruido,
        "alerta": alerta,
        "zona": "Biblioteca Norte",
        "timestamp": time.strftime("%Y-%m-%d %H:%M:%S")
    }

    client.publish(TOPIC, json.dumps(payload))
    print(f"[MQTT] Enviado: {payload}")
    time.sleep(5)
```
```python
noise_bridge.py
import paho.mqtt.client as mqtt
import json

# Configuración del broker AWS IoT y Local
BROKER_AWS = "a1b2c3d4e5-ats.iot.us-east-1.amazonaws.com"
BROKER_LOCAL = "1.124.1.0"

TOPICS = ["biblioteca/ruido", "alertas/ruido"]

# Cliente AWS
client_aws = mqtt.Client("BridgeAWS")
client_aws.tls_set(ca_certs="certs/root-CA.crt",
                   certfile="certs/device-certificate.crt",
                   keyfile="certs/private-key.key")

# Cliente Local
client_local = mqtt.Client("BridgeLocal")

# Conexión
client_aws.connect(BROKER_AWS, 8883, 60)
client_local.connect(BROKER_LOCAL, 1883, 60)

def on_message_local(client, userdata, msg):
    print(f"[Bridge] Mensaje local recibido -> reenviando a AWS: {msg.topic}")
    client_aws.publish(msg.topic, msg.payload)

def on_message_aws(client, userdata, msg):
    print(f"[Bridge] Mensaje AWS recibido -> reenviando a Local: {msg.topic}")
    client_local.publish(msg.topic, msg.payload)

client_local.on_message = on_message_local
client_aws.on_message = on_message_aws

for topic in TOPICS:
    client_local.subscribe(topic)
    client_aws.subscribe(topic)

print("🔗 Bridge AWS ↔ Local activo.")
client_local.loop_start()
client_aws.loop_forever()
```

3️⃣ mqtt_to_influx.py
```python
from influxdb_client import InfluxDBClient, Point, WritePrecision
from influxdb_client.client.write_api import SYNCHRONOUS
import paho.mqtt.client as mqtt
import json

# Configuración de InfluxDB
token = "tu_token_aqui"
org = "BibliotecaUniv"
bucket = "ruido"
influx_url = "http://localhost:8086"

client_influx = InfluxDBClient(url=influx_url, token=token, org=org)
write_api = client_influx.write_api(write_options=SYNCHRONOUS)

# Configuración MQTT
BROKER = "1.124.1.0"
TOPIC = "biblioteca/ruido"

def on_message(client, userdata, msg):
    try:
        data = json.loads(msg.payload.decode())
        point = (
            Point("nivel_ruido")
            .tag("zona", data.get("zona", "Desconocida"))
            .field("valor", data["nivel_ruido"])
            .field("alerta", str(data.get("alerta", "Ninguna")))
            .time(data["timestamp"], WritePrecision.NS)
        )
        write_api.write(bucket=bucket, org=org, record=point)
        print(f"[InfluxDB] Guardado: {data}")
    except Exception as e:
        print(f"Error procesando mensaje: {e}")

client_mqtt = mqtt.Client("InfluxConnector")
client_mqtt.on_message = on_message
client_mqtt.connect(BROKER, 1883, 60)
client_mqtt.subscribe(TOPIC)

print("📡 Conector MQTT → InfluxDB ejecutándose...")
client_mqtt.loop_forever()
```
🧠 Flujo de Ejecución
# 1️⃣ Simulador publica niveles de ruido
```python
python3 noise_sensor_simulator.py
```

# 2️⃣ Bridge reenvía mensajes entre AWS ↔ EC2
```python
python3 noise_bridge.py
```

# 3️⃣ InfluxDB almacena los datos recibidos
```python
python3 mqtt_to_influx.py
```
