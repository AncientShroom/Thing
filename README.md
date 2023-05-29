import RPi.GPIO as GPIO
import time
import Adafruit_CharLCD as LCD

# Set up GPIO pins for the matrix keypad
MATRIX = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
    ['*', 0, '#']
]

ROW = [5, 6, 13, 19]
COL = [12, 16, 20]

# Set up GPIO pins for the motors
MOTOR_PINS = [21, 22, 23]  # Change these to the appropriate pins for your motors

# Set up motor rotation parameters
STEPS_90_DEGREE = 1440

# Set up GPIO pins for the load cell
LOAD_CELL_PIN = 27  # Change this to the appropriate pin for your load cell

# Set up GPIO pins for the LCD display
LCD_RS = 26
LCD_EN = 24
LCD_D4 = 23
LCD_D5 = 17
LCD_D6 = 18
LCD_D7 = 22
LCD_COLS = 16
LCD_ROWS = 2

# Initialize GPIO
GPIO.setmode(GPIO.BCM)

# Set up matrix keypad
for j in range(3):
    GPIO.setup(COL[j], GPIO.OUT)
    GPIO.output(COL[j], 1)

for i in range(4):
    GPIO.setup(ROW[i], GPIO.IN, pull_up_down=GPIO.PUD_UP)

# Set up motors
for pin in MOTOR_PINS:
    GPIO.setup(pin, GPIO.OUT)

# Set up load cell
GPIO.setup(LOAD_CELL_PIN, GPIO.IN)

# Set up LCD display
lcd = LCD.Adafruit_CharLCD(LCD_RS, LCD_EN, LCD_D4, LCD_D5, LCD_D6, LCD_D7, LCD_COLS, LCD_ROWS)

def get_key():
    # Function to scan the matrix keypad and return the pressed key
    key = None

    for j in range(3):
        GPIO.output(COL[j], 0)

        for i in range(4):
            if GPIO.input(ROW[i]) == 0:
                key = MATRIX[i][j]
                while GPIO.input(ROW[i]) == 0:
                    pass

        GPIO.output(COL[j], 1)

    return key

def rotate_motor(motor_num, steps):
    # Function to rotate a motor a specified number of steps
    motor_pin = MOTOR_PINS[motor_num]
    for _ in range(steps):
        GPIO.output(motor_pin, GPIO.HIGH)
        time.sleep(0.001)
        GPIO.output(motor_pin, GPIO.LOW)
        time.sleep(0.001)

def display_message(line1, line2):
    # Function to display a message on the LCD display
    lcd.clear()
    lcd.message(line1 + '\n' + line2)

# Main loop
try:
    while True:
        key = get_key()

        if key is not None:
            print("Key pressed:", key)

            if key == 1:
                rotate_motor(0, STEPS_90_DEGREE)
            elif key == 2:
                rotate_motor(1, STEPS_90_DEGREE)
            elif key == 3:
                rotate_motor(2, STEPS_90_DEGREE)

        if GPIO.input(LOAD_CELL_PIN) == GPIO.HIGH:
            display_message("Profit: 02:00", "")
        else:
            display_message("Dispensing food", "")

        time.sleep(0.1)

except KeyboardInterrupt:
