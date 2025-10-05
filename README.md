# hack-aura-2025
hybrid traffic signal system
SOCKET PROGRAMING FOR COMMUNICATION PROTOCAL: LAP AS CLINET AND RASPBERRY PI AS SERVER
CODE FOR CLIENT:
import requests
import cv2
import base64
import json
import socket
import time

# Raspberry Pi IP & Port
PI_IP = "10.96.229.76"
PI_PORT = 5050

# Roboflow API setup
API_KEY = "4HFaAsMENMDwBhFt3uvl"
PROJECT = "adoptable-traffic-light-signal"
VERSION = 4
URL = f"https://detect.roboflow.com/{PROJECT}/{VERSION}?api_key={API_KEY}"

# Connect to Raspberry Pi
client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client_socket.connect((PI_IP, PI_PORT))
print("✅ Connected to Raspberry Pi")

cap = cv2.VideoCapture(0)  # webcam
while True:
    ret, frame = cap.read()
    if not ret:
        break

    # Encode image to base64
    _, buffer = cv2.imencode(".jpg", frame)
    b64_img = base64.b64encode(buffer).decode("utf-8")

    # Send to Roboflow
    response = requests.post(URL, data=b64_img, headers={"Content-Type": "application/x-www-form-urlencoded"})
    preds = response.json()

    # Count detections (class "toycar")
    car_count = sum(1 for p in preds.get("predictions", []) if p["class"] == "toycar")

    data = {"road_1": car_count, "road_2": 0, "road_3": 0, "road_4": 0}
    print("📤 Sending:", data)
    client_socket.sendall(json.dumps(data).encode())

    time.sleep(1)

cap.release()
client_socket.close()
 CODE FOR CLIENT :
 # raspi_signal_server.py
import socket
import RPi.GPIO as GPIO
import time

# 🟢 LED pin setup
GREEN_PIN = 17
GPIO.setmode(GPIO.BCM)
GPIO.setup(GREEN_PIN, GPIO.OUT)

# 📡 Setup socket server
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server_socket.bind(("0.0.0.0", 5050))  # Listen on all interfaces
server_socket.listen(1)

print("[INFO] Waiting for laptop connection...")
conn, addr = server_socket.accept()
print(f"[INFO] Connected to {addr}")

ASDKtry:
    while True:
        data = conn.recv(1024)
        if not data:
            break

        green_time = float(data.decode())
        print(f"[SIGNAL] Green time received: {green_time}s")

        # 🟩 Turn ON LED
        GPIO.output(GREEN_PIN, GPIO.HIGH)
        time.sleep(green_time)
        GPIO.output(GREEN_PIN, GPIO.LOW)

except Exception as e:
    print(f"[ERROR] {e}")
finally:
    conn.close()
    server_socket.close()
    GPIO.cleanup()

 
