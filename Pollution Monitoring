import time
import os
import Adafruit_DHT
import RPi.GPIO as GPIO
import sqlite3
 
# Setup for DHT11 sensor
DHT_SENSOR = Adafruit_DHT.DHT11
DHT_PIN = 17  # GPIO17 for DHT11
 
# Setup for MQ2 Sensor (Digital Output)
MQ2_PIN = 27  # GPIO27 for MQ2 Digital Output
 
# Set up GPIO mode
GPIO.setmode(GPIO.BCM)
GPIO.setup(MQ2_PIN, GPIO.IN)  # Set MQ2 digital output pin as input
 
# SQLite database setup
DATABASE = 'database_pollution.db'
 
# Function to initialize the SQLite database
def init_db():
    # Connect to the SQLite database (it will be created if it doesn't exist)
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    # Create the table if it doesn't exist already
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS pollution_data (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            temperature REAL,
            humidity REAL,
            final_score REAL,
            timestamp DATETIME DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    conn.commit()
    conn.close()
 
# Function to insert data into the SQLite database
def insert_data(temperature, humidity, final_score):
    conn = sqlite3.connect(DATABASE)
    cursor = conn.cursor()
    cursor.execute('''
        INSERT INTO pollution_data (temperature, humidity, final_score)
        VALUES (?, ?, ?)
    ''', (temperature, humidity, final_score))
    conn.commit()
    conn.close()
 
# Function to read DHT11 sensor data and calculate score
def read_dht11():
    humidity, temperature = Adafruit_DHT.read_retry(DHT_SENSOR, DHT_PIN)
    if humidity is not None and temperature is not None:
        print(f"Temp: {temperature}C  Humidity: {humidity}%")
        # Calculate the temperature score (ideal range 20-25°C)
        if 20 <= temperature <= 25:
            temp_score = 100
        else:
            temp_score = max(0, 100 - abs(temperature - 22) * 5)
        # Calculate the humidity score (ideal range 40-60%)
        if 40 <= humidity <= 60:
            humidity_score = 100
        else:
            humidity_score = max(0, 100 - abs(humidity - 50) * 2)
        overall_score = (temp_score + humidity_score) / 2
        print(f"Temperature Score: {temp_score}  Humidity Score: {humidity_score}")
        return temperature, humidity, overall_score
    else:
        print("Failed to get reading from DHT11 sensor.")
        return None, None, 0
 
# Function to read MQ2 sensor data and calculate air quality score
def read_mq2():
    # Read the digital output of the MQ2 sensor
    if GPIO.input(MQ2_PIN):
        print("Air Quality is good.")
        air_quality_score = 100  # Good air quality
    else:
        print("Warning: High pollution detected!")
        air_quality_score = 0  # Poor air quality
    return air_quality_score
 
# Main loop to monitor sensors, calculate scores, and log data to the database
def main():
    print(f"Current working directory: {os.getcwd()}")
    init_db()  # Initialize the database and table
    try:
        while True:
            print("Reading DHT11 Sensor...")
            temperature, humidity, temp_humidity_score = read_dht11()
            if temperature is None or humidity is None:
                time.sleep(2)
                continue  # Skip this loop iteration if sensor failed to read
            print("Reading MQ2 Sensor...")
            air_quality_score = read_mq2()
            # Calculate final overall score (average of the DHT11 and MQ2 scores)
            final_score = (temp_humidity_score + air_quality_score) / 2
            print(f"Overall Score: {final_score} / 100")
            # Log data to SQLite database
            insert_data(temperature, humidity, final_score)
            # Give feedback based on overall score
            if final_score == 100:
                print("Best condition possible!")
            elif final_score >= 80:
                print("Great condition!")
            elif final_score >= 50:
                print("Moderate condition.")
            elif final_score >= 30:
                print("Unpleasant, but tolerable.")
            else:
                print("Unbearable conditions!")
            time.sleep(2)  # Delay before next reading
 
    except KeyboardInterrupt:
        print("Program interrupted.")
        GPIO.cleanup()  # Clean up GPIO pins on exit
 
if __name__ == "__main__":
    main()
