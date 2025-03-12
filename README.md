import RPi.GPIO as GPIO
import time
import smtplib
import cv2
from email.mime.image import MIMEImage
from email.mime.multipart import MIMEMultipart

# GPIO Pin configuration
PIR_SENSOR_PIN = 4
BUZZER_PIN = 17
CAMERA_PORT = 0

# Email Configuration
SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587
EMAIL_SENDER = "your_email@gmail.com"
EMAIL_PASSWORD = "your_password"
EMAIL_RECEIVER = "receiver_email@gmail.com"

# Initialize GPIO
GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.setup(PIR_SENSOR_PIN, GPIO.IN)
GPIO.setup(BUZZER_PIN, GPIO.OUT)

def capture_image():
    camera = cv2.VideoCapture(CAMERA_PORT)
    ret, frame = camera.read()
    if ret:
        image_path = "intruder.jpg"
        cv2.imwrite(image_path, frame)
        camera.release()
        return image_path
    return None

def send_email(image_path):
    msg = MIMEMultipart()
    msg['From'] = EMAIL_SENDER
    msg['To'] = EMAIL_RECEIVER
    msg['Subject'] = "Home Security Alert - Motion Detected!"
    
    with open(image_path, 'rb') as img_file:
        img_data = MIMEImage(img_file.read(), name="intruder.jpg")
        msg.attach(img_data)
    
    try:
        server = smtplib.SMTP(SMTP_SERVER, SMTP_PORT)
        server.starttls()
        server.login(EMAIL_SENDER, EMAIL_PASSWORD)
        server.sendmail(EMAIL_SENDER, EMAIL_RECEIVER, msg.as_string())
        server.quit()
        print("Email sent successfully!")
    except Exception as e:
        print("Failed to send email:", e)

def motion_detected(channel):
    print("Motion detected! Activating alarm and capturing image...")
    GPIO.output(BUZZER_PIN, GPIO.HIGH)
    image_path = capture_image()
    if image_path:
        send_email(image_path)
    time.sleep(5)
    GPIO.output(BUZZER_PIN, GPIO.LOW)

# Set up event detection
GPIO.add_event_detect(PIR_SENSOR_PIN, GPIO.RISING, callback=motion_detected)

print("Home Security System is Active.")
try:
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    print("Shutting down...")
    GPIO.cleanup()

