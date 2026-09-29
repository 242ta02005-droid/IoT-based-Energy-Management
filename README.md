# IoT-based-Energy-Management
import time
import random
import json
import paho.mqtt.client as mqtt

# -----------------------------
# MQTT Configuration
# -----------------------------
MQTT_BROKER = "broker.hivemq.com"
MQTT_PORT = 1883
MQTT_TOPIC = "home/energy"

client = mqtt.Client()

try:
    client.connect(MQTT_BROKER, MQTT_PORT, 60)
    print("Connected to MQTT broker")
except Exception as e:
    print("MQTT connection error:", e)


# -----------------------------
# Simulated Sensors
# -----------------------------
def read_sensors():

    solar_power = random.uniform(0, 1500)
    home_load = random.uniform(200, 2000)

    battery_voltage = random.uniform(11.5, 13.8)
    battery_current = random.uniform(0, 10)

    battery_power = battery_voltage * battery_current

    return {
        "solar_power": round(solar_power, 2),
        "home_load": round(home_load, 2),
        "battery_voltage": round(battery_voltage, 2),
        "battery_current": round(battery_current, 2),
        "battery_power": round(battery_power, 2)
    }


# -----------------------------
# Energy Management
# -----------------------------
def energy_status(data):

    solar = data["solar_power"]
    load = data["home_load"]

    if solar > load:
        status = "SOLAR SURPLUS"
    elif solar < load:
        status = "ENERGY DEFICIT"
    else:
        status = "BALANCED"

    data["status"] = status

    return data


# -----------------------------
# Main Loop
# -----------------------------
while True:

    data = read_sensors()

    data = energy_status(data)

    message = json.dumps(data)

    print("\nEnergy Data:")
    print(message)

    try:
        client.publish(MQTT_TOPIC, message)
        print("Data sent to IoT cloud")
    except Exception as e:
        print("Publish error:", e)

    time.sleep(5)
