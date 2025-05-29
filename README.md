# ESP32 Motion Detection with MPU6050, ST7789 Display & Edge Impulse

This project demonstrates real-time motion detection using an MPU6050 accelerometer/gyroscope sensor on an ESP32 microcontroller. The sensor data is processed by a machine learning model trained with Edge Impulse, and the recognized motion label along with its confidence score is displayed on an ST7789 TFT screen.

The code is designed to be compiled and uploaded using the Arduino IDE.

## Acknowledgments

-   **Creator**: Anany Sharma at the University of Florida working under NSF grant. 2405373
-   This material is based upon work supported by the National Science Foundation under Grant No. 2405373.
-   Any opinions, findings, and conclusions or recommendations expressed in this material are those of the authors and do not necessarily reflect the views of the National Science Foundation.

---

## Features

* Reads 3-axis accelerometer data from MPU6050.
* Samples data at a configurable frequency (default: 60 Hz).
* Uses an Edge Impulse C++ library for on-device inference.
* Displays the most probable motion label and confidence percentage on an ST7789 TFT display.
* Includes a simple debouncing mechanism for stable label display.
* Outputs detailed inference results and sensor settings to the Serial Monitor for debugging.

---

## Hardware Requirements

* **ESP32 Development Board**: Any ESP32 board should work (e.g., ESP32-Dev Module).
* **MPU6050 Accelerometer/Gyroscope Module**: Connected via I2C.
* **ST7789 TFT LCD Display**:
    * The provided code uses a display initialized with `170x320` resolution. Adjust `display.init()` if your display has a different resolution (e.g., `240x240`, `240x320`).
    * Requires SPI connection (CS, DC, RST pins).
* **Jumper Wires**
* **Breadboard** (optional, for easier connections)

---

## Software & Libraries

1.  **Arduino IDE**: Download and install from the [Arduino website](https://www.arduino.cc/en/software).
2.  **ESP32 Board Support**: Install the ESP32 board package in Arduino IDE.
    * Go to `File > Preferences`.
    * Add `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json` to "Additional Boards Manager URLs".
    * Go to `Tools > Board > Boards Manager...`, search for "esp32" and install it.
3.  **Required Arduino Libraries**: Install these via `Sketch > Include Library > Manage Libraries...`
    * `Adafruit MPU6050` (by Adafruit)
    * `Adafruit Unified Sensor` (by Adafruit, dependency for MPU6050)
    * `Wire` (usually comes with ESP32 core)
    * `Adafruit GFX Library` (by Adafruit)
    * `Adafruit ST7735 and ST7789 Library` (by Adafruit)
4.  **Edge Impulse Exported Arduino Library**:
    * This project requires a custom Edge Impulse library specific to your trained model.
    * The placeholder in the code is `Lab_7_motion_detection_v1_inferencing.h`. **You will need to replace this with the library generated from your Edge Impulse project.**

---

## Edge Impulse Integration

This code is designed to work with a model trained on Edge Impulse. The general workflow is:

1.  **Collect Data**: Use your ESP32 and MPU6050 to collect motion data (e.g., "idle", "walking", "waving") and upload it to your Edge Impulse project.
    * You might need a separate data collection sketch or use Edge Impulse's data forwarder.
2.  **Design Impulse**: In Edge Impulse Studio, create an impulse. This typically involves:
    * A time-series input block.
    * A processing block (e.g., Spectral Analysis for motion data).
    * A learning block (e.g., Neural Network Classifier).
3.  **Train Model**: Train your model using the collected data.
4.  **Deployment**:
    * Go to the "Deployment" tab in your Edge Impulse project.
    * Select "Arduino library" and click "Build".
    * Download the generated ZIP file.
5.  **Add Library to Arduino IDE**:
    * In Arduino IDE, go to `Sketch > Include Library > Add .ZIP Library...` and select the downloaded ZIP file.
    * This will make the necessary `<YourProjectName_inferencing.h>` file available.

---

## Setup & Wiring

1.  **MPU6050 (I2C)**:
    * `VCC` -> `3.3V` on ESP32
    * `GND` -> `GND` on ESP32
    * `SCL` -> `GPIO22` (ESP32 default SCL)
    * `SDA` -> `GPIO21` (ESP32 default SDA)
    * *(You can use other I2C pins if needed, but you'll have to modify `Wire.begin()`)*

2.  **ST7789 TFT Display (SPI)**:
    * `VCC` / `VIN` -> `3.3V` or `5V` (check your display's requirements)
    * `GND` -> `GND`
    * `SCL` / `SCK` / `CLK` -> `GPIO18` (ESP32 default HSPI_CLK)
    * `SDA` / `MOSI` / `DIN` -> `GPIO23` (ESP32 default HSPI_MOSI)
    * `RES` / `RST` -> `GPIO26` (configurable, as defined by `TFT_RST`)
    * `DC` / `A0` -> `GPIO25` (configurable, as defined by `TFT_DC`)
    * `CS` -> `GPIO33` (configurable, as defined by `TFT_CS`)
    * `BLK` / `LED` (Backlight) -> `3.3V` (or a PWM pin for brightness control, not implemented in this code)

    *Important*: The pins `TFT_CS`, `TFT_DC`, and `TFT_RST` are defined in the code. You can change these GPIOs if necessary.

    ```c++
    #define TFT_CS   33  // Chip Select control pin
    #define TFT_DC   25  // Data/Command select pin
    #define TFT_RST  26  // Reset pin
    ```

---

## How to Use

1.  **Install Libraries**: Ensure all required software and libraries (see above) are installed.
2.  **Edge Impulse Model**:
    * Train your motion detection model in Edge Impulse.
    * Deploy it as an Arduino library and add it to your Arduino IDE.
    * Modify the `#include` statement at the top of the `.ino` file to match your Edge Impulse project's header file name:
        ```c++
        // Replace with your actual Edge Impulse library header
        #include <YourProjectName_inferencing.h>
        ```
3.  **Wiring**: Connect the MPU6050 and ST7789 display to your ESP32 as described in the "Setup & Wiring" section.
4.  **Configure Arduino IDE**:
    * Select your ESP32 board from `Tools > Board`.
    * Select the correct COM port from `Tools > Port`.
5.  **Upload Code**: Click the "Upload" button in the Arduino IDE.
6.  **Monitor**:
    * Open the Serial Monitor (`Tools > Serial Monitor`) and set the baud rate to `115200`. You should see sensor initialization messages, and then inference results.
    * The ST7789 display should show the detected motion label and its confidence.

---

## Code Explanation

* **Includes**: Necessary libraries for MPU6050, ST7789, Wire (I2C), and your Edge Impulse model.
* **Sampling Parameters**:
    * `FREQUENCY_HZ`: How many times per second to sample accelerometer data.
    * `INTERVAL_MS`: Calculated delay between samples.
* **Sensor & Buffer Variables**:
    * `Adafruit_MPU6050 mpu`: Instance for the MPU6050 sensor.
    * `Adafruit_ST7789 display`: Instance for the ST7789 display with specified CS, DC, RST pins.
    * `features[]`: A float array to store one window of accelerometer data (X, Y, Z). Its size (`EI_CLASSIFIER_DSP_INPUT_FRAME_SIZE`) is determined by your Edge Impulse model's window size and the number of axes (3 for X, Y, Z).
    * `feature_ix`: Index to keep track of the current position in the `features` buffer.
    * `labelStartTime`, `currentLabel`, `lastLabel`, `thresholdTime`: Variables for managing label display stability (debouncing).
* **`setup()` Function**:
    * Initializes Serial communication (`Serial.begin(115200)`).
    * Initializes the ST7789 display (`display.init(170, 320); display.setRotation(3);`).
    * Initializes the MPU6050 sensor (`mpu.begin()`).
    * Sets the MPU6050 accelerometer range (e.g., `MPU6050_RANGE_8_G`), gyroscope range, and filter bandwidth. These settings should ideally match the settings used during data collection for your Edge Impulse model.
    * Prints model information (input size, label count) to the Serial Monitor.
* **`loop()` Function**:
    * **Sampling**: Checks if `INTERVAL_MS` has passed since the last sample.
    * Reads accelerometer data (`a.acceleration.x`, `a.acceleration.y`, `a.acceleration.z`) from the MPU6050.
    * Stores the three-axis data into the `features` buffer.
    * **Inference**: Once the `features` buffer is full (`feature_ix == EI_CLASSIFIER_DSP_INPUT_FRAME_SIZE`):
        * Creates an Edge Impulse `signal_t` structure from the `features` buffer.
        * Calls `run_classifier()` to perform inference.
        * Prints detailed classification results (label and confidence for each class) to the Serial Monitor using `ei_printf`.
        * Determines the label with the `highestProbability`.
        * **Label Debouncing**: Updates `currentLabel` only if `newLabel` persists for longer than `thresholdTime` (500ms by default). This prevents flickering on the display if predictions change rapidly.
        * **Display Update**:
            * Clears the ST7789 display.
            * Prints the uppercase version of the determined `newLabel` (or `currentLabel` after debouncing logic refinement).
            * Prints the `highestProbability` as a percentage.
        * Resets `feature_ix` to 0 to start collecting the next window of data.
* **`ei_printf()` Function**: A helper function provided by Edge Impulse to print formatted strings to the Serial Monitor, similar to `printf`.

---

## Customization

* **Sensor Settings**: In `setup()`, you can change `mpu.setAccelerometerRange()`, `mpu.setGyroRange()`, and `mpu.setFilterBandwidth()` if your Edge Impulse model was trained with different settings or if you need different sensitivity/noise filtering.
* **Display Pins**: Modify `TFT_CS`, `TFT_DC`, `TFT_RST` macros if you use different GPIOs for your display.
* **Display Resolution & Rotation**: Adjust `display.init()` and `display.setRotation()` in `setup()` based on your ST7789 display.
* **Sampling Frequency**: Change `FREQUENCY_HZ` if your model expects a different sampling rate. Ensure `EI_CLASSIFIER_RAW_SAMPLE_COUNT` in your Edge Impulse model corresponds to this.
* **Label Debounce Time**: Modify `thresholdTime` (in milliseconds) to make the displayed label update faster or slower.
* **`WiFi.h` and `GyverOLED.h`**: These libraries are included but not actively used in the primary code block. They might be remnants of previous experiments or planned features. You can remove them if not needed.

---

## Troubleshooting

* **No MPU6050 Found**:
    * Check wiring (VCC, GND, SDA, SCL).
    * Ensure the MPU6050 is powered correctly (3.3V).
    * Run an I2C scanner sketch to see if the ESP32 detects the MPU6050 (default address is 0x68).
* **Display Not Working**:
    * Check all SPI wiring (CS, DC, RST, SCL/CLK, SDA/MOSI) and power.
    * Ensure `TFT_CS`, `TFT_DC`, `TFT_RST` pins in the code match your wiring.
    * Verify the `display.init()` parameters match your screen resolution.
* **Incorrect Motion Detection**:
    * Ensure the MPU6050 settings (range, filter) in `setup()` match those used for data collection in Edge Impulse.
    * Your Edge Impulse model might need more data or retraining.
    * The `EI_CLASSIFIER_DSP_INPUT_FRAME_SIZE` must match what your Edge Impulse model expects. This is automatically handled if you use the exported library correctly.
* **Serial Monitor Gibberish**: Ensure the baud rate is set to `115200`.

---

## License

This project code is provided as-is. You are free to use, modify, and distribute it. If you use the acknowledgments, please keep them intact. Consider using a common open-source license like MIT or Apache 2.0 if you plan to share your derivative work more broadly.