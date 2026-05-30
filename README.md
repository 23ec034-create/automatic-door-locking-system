# automatic-door-locking-system
from machine import Pin, UART, PWM
from time import sleep

uart = UART(0, baudrate=9600, tx=Pin(0), rx=Pin(1))
servoPin = PWM(Pin(16))
servoPin.freq(50)

led = Pin(15, Pin.OUT)

def set_servo_angle(angle):
    # Ensure angle is within valid range (0 to 180 degrees)
    angle = max(0, min(180, angle))
    
    # Convert angle to duty cycle (0.5ms to 2.5ms pulse width)
    min_duty = 1638  # Corresponds to 0.5ms pulse width
    max_duty = 8192  # Corresponds to 2.5ms pulse width
    duty = int(min_duty + (max_duty - min_duty) * (angle / 180))
    
    # Set the PWM duty cycle
    servoPin.duty_u16(duty)
    print(f"Servo moved to {angle}°")

print("Waiting for Bluetooth command...")

while True:
    if uart.any():  # Check if data is received from HC-05
        command = uart.read(1)  # Read one byte of data
        print(f"Received command: {command}")
        
        if command == b'1':  # Command '1' rotates servo to 180°
            set_servo_angle(180)
            led.value(1)  # Turn on LED
        elif command == b'0':  # Command '0' rotates servo back to 0°
            set_servo_angle(0)
            led.value(0)  # Turn off LED
        
        sleep(0.1)  # Small delay for stability
