# 🏛️ Control de Ruido en Bibliotecas Universitarias

---

## 📋 Descripción del Proyecto

El **Sistema IoT para el Control de Ruido en Bibliotecas Universitarias** tiene como objetivo **monitorear y controlar los niveles de ruido** dentro de espacios académicos.  
Utiliza sensores (reales o simulados) para medir decibelios en tiempo real, enviar los datos mediante **MQTT** hacia un servidor en **AWS EC2**, almacenarlos en **InfluxDB**, y visualizarlos mediante **Grafana**.  

Además, el sistema genera **alertas automáticas** cuando los niveles de ruido superan umbrales definidos, registrando los eventos críticos para su análisis posterior.

---

## 🏗️ Arquitectura del Sistema

[Sensor de Ruido / Simulador]
↓ (MQTT)
[Broker Mosquitto EC2]
↓
[InfluxDB v2]
↓
[Grafana Dashboard]

yaml
Copiar código

**Componentes Clave:**
- 📡 Comunicación IoT basada en **MQTT**
- 💾 Almacenamiento de series temporales con **InfluxDB**
- 📊 Visualización y alertas en **Grafana**

---

## ⚙️ Instalación y Configuración

### 1️⃣ Preparación del Entorno (Raspberry Pi o PC Local)

```bash
sudo apt update
sudo apt install mosquitto-clients python3-pip -y
pip3 install paho-mqtt
Crear carpeta para certificados (si usas AWS IoT):
bash
Copiar código
mkdir ~/certs
# Copiar tus archivos de seguridad:
# root-CA.crt, device-certificate.crt, private-key.key
 2️⃣ Configuración AWS IoT Core (Opcional)
Crear una Thing en AWS IoT Core

Generar y activar los certificados

Adjuntar una política con permisos de conexión y publicación

#3️⃣ Configuración en Servidor EC2 (AWS)
Instalar servicios base
bash
Copiar código
sudo apt update && sudo apt upgrade -y
sudo apt install mosquitto mosquitto-clients -y
🔹 Instalar InfluxDB v2
bash
Copiar código
wget -q https://repos.influxdata.com/influxdata-archive.key
echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdata-archive.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
sudo apt update
sudo apt install influxdb2 -y
🔹 Instalar Grafana
bash
Copiar código
sudo apt install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
sudo apt update
sudo apt install grafana -y
🔹 Iniciar servicios
bash
Copiar código
sudo systemctl enable mosquitto influxdb grafana-server
sudo systemctl start mosquitto influxdb grafana-server
🔹 Configurar InfluxDB
bash
Copiar código
sudo influx setup
# Usuario: admin
# Password: noise123
# Org: BibliotecaUniv
# Bucket: ruido
🚀 Uso del Sistema
Inicio Completo:

Terminal	Proceso	Comando
1	Bridge MQTT (AWS ↔ EC2)	python3 noise_bridge.py
2	Conector InfluxDB	python3 mqtt_to_influx.py
3	Simulador de ruido	python3 noise_sensor_simulator.py

📜 Scripts Principales
1️⃣ noise_sensor_simulator.py
Características:

Simula niveles de ruido por hora del día.

Umbrales de alerta configurables.

Envío MQTT cada 5 segundos.

Detección de picos y ruido continuo.

Alertas Implementadas:

Tipo de Alerta	Condición	Descripción
🔴 RUIDO_EXCESIVO_CONTINUO	>75 dB durante 30s	Alerta de ruido prolongado
🟠 PICO_DE_RUIDO	>85 dB puntual	Pico breve de ruido
⚠️ ZONA_CRITICA	>90 dB prolongado	Nivel crítico de ruido

Ejemplo de simulación:

python
Copiar código
if random.random() < 0.1:
    ruido = random.randint(85, 95)
else:
    ruido = random.randint(40, 70)
2️⃣ noise_bridge.py
Funcionalidad:

Conexión bidireccional entre AWS IoT y Broker Local

Reconexión automática en caso de fallo

Soporte para múltiples topics:

biblioteca/ruido

alertas/ruido

3️⃣ mqtt_to_influx.py
Características:

Procesa mensajes MQTT y los guarda en InfluxDB.

Registra campos dinámicos:

nivel_ruido

alerta

ubicacion

Manejo robusto de errores.

📊 Configuración en Grafana
Datasource:

makefile
Copiar código
Name: InfluxDB-Ruido
URL: http://localhost:8086
Organization: BibliotecaUniv
Bucket: ruido
Token: [tu token de InfluxDB]
Dashboard sugerido:

🔹 Nivel de Ruido (línea en tiempo real)

🔹 Alertas (tabla de eventos críticos)

🔹 Promedio por hora

🔹 Alertas activas por zona

⚠️ Umbrales y Lógica de Detección
Alerta	Condición	Acción
RUIDO_EXCESIVO_CONTINUO	>75 dB por más de 30s	Registrar alerta
PICO_DE_RUIDO	>85 dB puntual	Notificar instantáneamente
ZONA_CRITICA	>90 dB prolongado	Enviar alerta urgente

🧠 Ejemplo de Flujo Completo ✅
bash
Copiar código
# 1. Simulador publica niveles
python3 noise_sensor_simulator.py

# 2. Bridge reenvía mensajes
python3 noise_bridge.py

# 3. InfluxDB guarda datos
python3 mqtt_to_influx.py

# 4. Consultar datos recientes
influx query 'from(bucket: "ruido") |> range(start: -10m)'

# 5. Visualizar en Grafana
http://[IP-EC2]:3000
📈 Métricas Capturadas
nivel_ruido (decibelios)

alerta (tipo de evento)

zona (ubicación del sensor)

timestamp (hora exacta)

🖼️ Ejemplos Visuales (para README del Repositorio)
Subir las imágenes a tu repo:

bash
Copiar código
/images/grafana_dashboard.png  
/images/mqtt_flow.png  
/images/architecture.png
Ejemplo de cómo se mostrarán:

🔹 Arquitectura del Sistema

🔹 Dashboard en Grafana

🔹 Flujo MQTT funcionando

🧩 Personalización
En noise_sensor_simulator.py puedes ajustar los parámetros según tus necesidades:

python
Copiar código
# Probabilidad de ruido alto
if random.random() < 0.15:  # 15% probabilidad

# Umbral de alerta continua
if contador_ruido > 6:  # 6 lecturas seguidas (>75 dB)
