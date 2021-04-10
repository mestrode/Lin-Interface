# LIN-Interface-Library
Send and Sends and Request data by compiling a LIN Frames and transmission via Serial (as a Bus Master)

The HardwareSerial UART of an ESP32 is used. (But in the past I used a software serial and therefore I derived this class in a prior version from the class SoftwareSerial)

# Transceiver
I've used a TJA1020 Transceiver on HW side in my project. The chip contains a statemachine, which needs to be controlled before you will be able to write or receive data. To keep thinks easy, I created a derived class (from this one) which consider the statemachine every time using the bus: https://github.com/mestrode/Lin-Transceiver-Library

# example
Take a look into this repo to see, how this works: https://github.com/mestrode/IBS-Sensor-Library

This code calls some methods of BatSensor which utilizes the Lin-Interface

    // LIN Bus Interface provided viy TJA1020
    #include "TJA1020.hpp"
    // IBS Batterie Sensor
    #include "IBS_Sensor.hpp"

    #define LIN_SERIAL_SPEED LIN_BAUDRATE_IBS_SENSOR /* Required by IBS Sensor */
    #define lin_NSLP_Pin 32
    
    // utilize the TJA1020 by using UART2 for writing and reading Frames
    // but keep in mind: the Lin_TJA1020 is only a extension of this library.
    Lin_TJA1020 LinBus(2, LIN_SERIAL_SPEED, lin_NSLP_Pin); // UART_nr, Baudrate, /SLP
    
    // Hella IBS 200x "Sensor 2"
    IBS_Sensor BatSensor(2);

    void setup()
    {
        // tell the BatSensor object which LinBus to be used
        BatSensor.LinBus = &LinBus;
    }

    void showSensorData() {
        // read data from sensor (method request data by using several
        // Lin-Frames)
        BatSensor.readFrames();

        // may you using a Bus-Transceiver like the TJA1020 which should
        // go to sleep after transmission (depends on your HW)
        LinBus.setMode(LinBus.Sleep);

        // use received data
        Serial.printf("Calibration done: %d &\n",
                                   BatSensor.CalibrationDone);
        Serial.printf("Voltage: %.3f Volt\n", BatSensor.Ubat);
        Serial.printf("Current: %.3f Ampere\n", BatSensor.Ibat);
        Serial.printf("State of Charge: %.1f %\n", BatSensor.SOC);
        Serial.printf("State of Health: %.1f &\n", BatSensor.SOH);
        Serial.printf("Available Capacity: %.1f &\n", BatSensor.Cap_Available);
    }

The LinBus is provided to the BatSensor and is used internaly.
The aktual data handling looks like this:

   bool IBS_Sensor::readFrameCapacity()
    {

        bool chkSumValid = LinBus->readFrame(IBS_FrameID[_SensorNo][IBS_FRM_CAP]);
        if (chkSumValid)
        {
            // decode some bytes (incl. rescaling)
            Cap_Max = (float((LinBus->LinMessage[1] << 8) + LinBus->LinMessage[0])) / 10;
            Cap_Available = (float((LinBus->LinMessage[3] << 8) + LinBus->LinMessage[2])) / 10;
            // receive a single byte
            Cap_Configured = LinBus->LinMessage[4];
            // decode flags within a byte
            CalibByte = LinBus->LinMessage[5];
            CalibrationDone = bitRead(LinBus->LinMessage[5], 0);
        }
        return chkSumValid;
    }

# configuration Frames
See description of Frame 0x3C and 0x3D in the doc folder of this project.
Don't know if this is valid in general, but at least in the Project IBS-Sensor-Library it worked.

# see also
Lin Specification provided by Microchip
https://microchipdeveloper.com/local--files/lin:specification/LIN-Spec_2.2_Rev_A.PDF
IBS-Sensor-Library
https://github.com/mestrode/IBS-Sensor-Library
Lin-Transceiver-Library (TJA1020)
https://github.com/mestrode/Lin-Transceiver-Library