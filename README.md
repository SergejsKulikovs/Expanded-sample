# Expanded-sample
#include "mbed.h"
#include "LSM6DSLSensor.h"

// Serial interface setup
UnbufferedSerial pc(USBTX, USBRX, 115200); // Serial connection to PC for debugging

// Sensor initialization (using hypothetical I2C pins, adjust as necessary)
I2C i2c(D14, D15);
LSM6DSLSensor acc(&i2c, LSM6DSL_ACC_GYRO_I2C_ADDRESS_HIGH);

// Buffer for incoming serial data
char buffer[1] = {0};

// Flag to indicate new data received
volatile bool newData = false;

// Interrupt Service Routine for handling received data
void serial_isr() {
    pc.read(buffer, 1);  // Read one byte from the serial
    newData = true;      // Set flag to true
}

void printAccelerometerData() {
    int16_t accData[3];  // Array to store accelerometer data
    acc.get_x_axes(accData);  // Get accelerometer data
    printf("Accelerometer Data: X=%d, Y=%d, Z=%d\r\n", accData[0], accData[1], accData[2]);
}

int main() {
    pc.set_format(8, BufferedSerial::None, 1); // Set serial format
    pc.attach(&serial_isr, UnbufferedSerial::RxIrq); // Attach ISR to handle incoming data
    acc.enable_x();  // Enable accelerometer

    while (true) {
        if (newData) {  // Check if new data is available
            if (buffer[0] == 'a') {  // If 'a' is received, print accelerometer data
                printAccelerometerData();
            }
            newData = false;  // Reset flag after handling
        }
        ThisThread::sleep_for(10ms);  // Sleep to reduce CPU usage
    }
}
