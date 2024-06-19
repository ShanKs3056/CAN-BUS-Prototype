To use the MCP2515 CAN bus module with an Arduino to send and receive CAN messages and display data on a 20x4 Liquid Crystal Display (LCD). Below is an explanation of the main components and functionalities of the code:

1. **Libraries Used**:
   - `SPI.h`: This library allows the Arduino to communicate with SPI devices.
   - `mcp2515.h`: This library provides functions to interface with the MCP2515 CAN controller.
   - `LiquidCrystal.h`: This library allows interfacing with LCD displays.

2. **LCD Configuration**:
   - The LCD is configured with the specified pins (`rs`, `en`, `d4`, `d5`, `d6`, `d7`) and initialized to a 20x4 character display.

3. **CAN Configuration**:
   - An instance of the `MCP2515` class is created with the SPI CS pin set to 10.
   - In the `setup()` function, the CAN bus is initialized to a bitrate of 500KBPS with an 8MHz clock, and the mode is set to normal.

4. **Serial Communication**:
   - Serial communication is initialized at 9600 baud rate for debugging or logging purposes.

5. **LCD Initialization**:
   - The LCD is initialized, cleared, and a welcome message "CANBUS TUTORIAL" is displayed for 3 seconds.

6. **LED Indicator**:
   - The built-in LED pin is set as an output to indicate a specific condition (when `y > 28`).

7. **Main Loop**:
   - The loop continuously reads CAN messages and processes the data if a valid message is received.
   - It reads three bytes (`x`, `y`, `z`) from the CAN message data and displays the values of `x` and `y` on the LCD.
   - If `y` (representing temperature) is greater than 28, the built-in LED is turned on.
   - It sends a CAN message with ID `0x036` and 8 bytes of data. The data is currently set to zeros.

Here’s a brief overview of what each part of the code does:

### Libraries and Definitions
```cpp
#include <SPI.h>              // Library for using SPI Communication 
#include <mcp2515.h>          // Library for using CAN Communication
#include <LiquidCrystal.h> 

const int rs = 3, en = 4, d4 = 5, d5 = 6, d6 = 7, d7 = 8;  
LiquidCrystal lcd(rs, en, d4, d5, d6, d7);  // Define LCD display pins
```

### CAN Message Structure and MCP2515 Initialization
```cpp
struct can_frame canMsg;
MCP2515 mcp2515(10);                 // SPI CS Pin 10
```

### Setup Function
```cpp
void setup()
{
  Serial.begin(9600);                // Begins Serial Communication at 9600 baudrate
  SPI.begin();                       // Begins SPI communication
  lcd.begin(20, 4);
  lcd.clear();         
  lcd.setCursor(0, 0);
  lcd.print("CANBUS TUTORIAL");
  delay(3000);
  lcd.clear();
  pinMode(LED_BUILTIN, OUTPUT);
  mcp2515.reset();
  mcp2515.setBitrate(CAN_500KBPS, MCP_8MHZ); // Sets CAN at speed 500KBPS and Clock 8MHz
  mcp2515.setNormalMode();                  // Sets CAN at normal mode
}
```

### Main Loop Function
```cpp
void loop()
{
  if (mcp2515.readMessage(&canMsg) == MCP2515::ERROR_OK) // To receive data (Poll Read)
  {
    int x = canMsg.data[0];
    int y = canMsg.data[1];
    int z = canMsg.data[2];
    if(y > 28){
      digitalWrite(LED_BUILTIN, HIGH);
    }
 
    lcd.setCursor(0, 2);         // Display Temp & Humidity value received at 20x4 LCD
    lcd.print("Humidity: ");
    lcd.print(x);
    lcd.setCursor(0, 1);
    lcd.print("Temperature: ");
    lcd.print(y);
    delay(1000);
    lcd.clear();
  }
 
  canMsg.can_id  = 0x036;           // CAN id as 0x036
  canMsg.can_dlc = 8;               // CAN data length as 8
  canMsg.data[0] = 0x00;             
  canMsg.data[1] = 0x00;            // Update temperature value in [1]
  canMsg.data[2] = 0x00;            // Rest all with 0
  canMsg.data[3] = 0x00;
  canMsg.data[4] = 0x00;
  canMsg.data[5] = 0x00;
  canMsg.data[6] = 0x00;
  canMsg.data[7] = 0x00;
 
  mcp2515.sendMessage(&canMsg);     // Sends the CAN message
  delay(1000);
}
```

### Summary
This code sets up an Arduino to communicate with a CAN bus using an MCP2515 CAN controller. It reads messages from the CAN bus, displays relevant data on an LCD, and sends messages onto the CAN bus. Additionally, it uses an LED to indicate if a certain condition (temperature > 28) is met.
