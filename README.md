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

📂 **Sube las imágenes a tu repositorio en la carpeta:**

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
