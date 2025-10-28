
![Real-time-Multifunctional-clock report-4](https://github.com/user-attachments/assets/0a03f995-822f-4600-b440-68ca33a06c80)

This project is a sophisticated embedded system centered around an 8051 microcontroller, designed to be a comprehensive timekeeping and utility device. It leverages the DS12887 Real-Time Clock (RTC) integrated circuit for highly accurate, battery-backed timekeeping and extends its functionality to include a stopwatch, multiple countdown timers, a tally counter, and ambient temperature monitoring with a basic weather status indicator. The system provides a user-friendly interface via a 16x2 LCD and a two-button navigation system, demonstrating a complete integration of hardware and low-level firmware design.

**1\. Core System Architecture & Hardware Integration:**

*   **Microcontroller Unit (MCU):** An 8051-family microcontroller acts as the central brain. It executes the control logic, manages communication between peripherals, handles user input, and drives the LCD output.
    
*   **Real-Time Clock (RTC):** The DS12887 RTC chip is critical for autonomous timekeeping. It contains its own oscillator and battery, allowing it to maintain time and calendar data even when the main system power is off. Communication with the 8051 is achieved via a memory-mapped parallel interface, with a dedicated Chip Select (CS) line (P3.3) enabling the RTC's internal register bank.
    
*   **Display Unit:** A standard HD44780-compatible 16x2 LCD is used for all user output. It interfaces with the 8051 in 8-bit parallel mode:
    
    *   **Data Bus:** Port 1 (P1).
        
    *   Control Lines: RS (P3.0), R/W (P3.1), and E (P3.2).
        
*   **User Input:** A minimalist two-button interface is implemented for navigation and control:
    
    *   **Button A (P3.5):** Used for mode cycling, menu exiting, and reset functions.
        
    *   **Button B (P3.4):** Used for selection, incrementing counts, and pausing/resuming timers.
        
*   **Audio Output:** A piezo buzzer connected to P3.1 provides audible feedback. Its primary function is an hourly chime, activated when the RTC time reaches exactly hh:00:00.
    
*   **Environmental Sensor:** An analog temperature sensor is connected to Port 2 (P2). The 8051 reads the raw analog voltage (acting as a simple ADC input) and converts it into a temperature status.
    
![Real-time-Multifunctional-clock report-3](https://github.com/user-attachments/assets/e1e5ddd3-b682-49ac-860e-ece71477c6a6)



**2\. Firmware Operation & Key Algorithms:**

*   **Initialization Sequence:**
    
    1.  Upon power-up, a critical 200ms delay allows the DS12887's internal oscillator to stabilize.
        
    2.  **RTC Configuration:** The 8051 writes to the RTC's internal registers to set its operational mode.
        
        *   **Register A (0x0A):** Bit 6 is set to 1 to start the oscillator.
            
        *   **Register B (0x0B):** Configured for 24-hour format (BIT2=0), Binary-Coded Decimal (BCD) data mode (BIT2=0?), and to enable the update of time/date registers (BIT7=0).
            
    3.  **Initial Time/Date Set:** The firmware hard-codes an initial time (16:58:55) and date (October 19, 2004) into the RTC's respective registers.
        
    4.  **LCD Initialization:** The LCD is configured for 2-line display, cursor off, and auto-incrementing cursor movement.
        
*   **Main Control Loop (OV1):**This infinite loop is the core of the system's real-time operation, continuously performing the following tasks:
    
    *   **Time Fetching & Formatting:** Reads hours, minutes, and seconds from the RTC. A global flag (FORMAT\_SELECT at 27H.0) determines the display format.
        
        *   **24-Hour Mode:** Displays hours from 00-23 directly.
            
        *   **12-Hour Mode:** Implements a conversion algorithm. It masks the MSB (which holds the AM/PM flag in the DS12887), handles the special cases of 00 (converts to 12 AM) and 12 (stays as 12 PM), and appends the correct "AM" or "PM" indicator.
            
    *   **Date & Day Display:** Reads the day, month, and year from the RTC. The day-of-the-week value (0-6) is used as an index into a lookup table (DAY\_STRINGS) stored in ROM to fetch and display the corresponding string (e.g., "Mon").
        
    *   **Temperature Processing:** Reads the analog value from Port 2. A subroutine (GET\_TEMP\_MSG) classifies the reading into a weather status using predefined thresholds:
        
        *   < 15 → "COLD"
            
        *   15 - 25 → "WARM"
            
        *   \> 25 → "HOT"The raw ADC value is also converted to a 3-digit ASCII string and displayed.
            
    *   **Hourly Chime Logic:** Checks if both the minutes and seconds registers are zero. If true, it activates the buzzer on P3.1 for approximately one second.
        
    *   **Input Polling:** Constantly scans the state of Button A. A press event triggers a debounce delay and then enters the menu system.
        
*   **Menu System & Additional Features:**The menu is a state machine navigated with Buttons A (cycle) and B (select).
    
    *   **Stopwatch:** Counts upwards with a resolution of 0.1 seconds, achieved using a DELAY\_200MS subroutine in a loop. It uses registers R0 (tens of seconds) and R1 (seconds) to count up to 99.9 seconds. Button B pauses/resumes; Button A exits.
        
    *   **Countdown Timer:** Offers three preset durations (30, 60, 90 seconds). It counts down with 1-second resolution using the DELAY\_1S subroutine, which is precisely calibrated using Timer 1 in mode 2. The buzzer is activated upon reaching zero.
        
    *   **Tally Counter:** A simple 0-99 counter. Button B increments the count; Button A resets and exits.
        
    *   **Clock Format Toggle:** Allows the user to switch the main display between 12-hour and 24-hour formats, dynamically changing the FORMAT\_SELECT flag.
        
![Real-time-Multifunctional-clock report-1](https://github.com/user-attachments/assets/2d4690ac-899a-4468-bd9b-36e61cdd1536)
![Real-time-Multifunctional-clock report-2](https://github.com/user-attachments/assets/eb9cd9c9-7242-4ea2-acc9-116668eb2b91)



**3\. Technical Challenges and Solutions:**

*   **Button Debouncing:** Implemented in software by introducing a delay (e.g., DELAY\_50MS) after detecting a button press and then re-checking the button state before registering the input.
    
*   **RTC Communication Timing:** The DS12887 requires specific timing for read/write cycles. The code carefully toggles the CS line and inserts NOP instructions and custom DELAY calls to meet these requirements.
    
*   **12/24 Hour Conversion:** The algorithm correctly handles edge cases like 12 PM and 12 AM, which are non-intuitive in a 24-hour to 12-hour conversion.
    
*   **Resource Management:** The 8051's limited internal RAM is managed by using bit-addressable memory for flags and carefully allocating register banks for different subroutines.
    

### **Steps to Run and Reproduce the Project**

**A. Hardware Requirements & Setup:**

1.  **Components List:**
    
    *   8051 Microcontroller (e.g., AT89C51/52)
        
    *   DS12887 Real-Time Clock (RTC) Module
        
    *   16x2 LCD Display (HD44780 compatible)
        
    *   Analog Temperature Sensor (e.g., LM35 or a potentiometer for simulation)
        
    *   Piezo Buzzer (Active)
        
    *   Two Push Buttons
        
    *   Resistors (for pull-ups/pull-downs on buttons)
        
    *   Crystal Oscillator for 8051 (typically 11.0592 MHz or 12 MHz)
        
    *   Capacitors & Power Supply (5V DC)
        
2.  **Circuit Assembly:**
    
    *   **Microcontroller:** Connect the crystal and reset circuit.
        
    *   **LCD:**
        
        *   Data Pins (D0-D7) → Port 1 (P1.0-P1.7)
            
        *   RS → P3.0
            
        *   R/W → P3.1 (can be grounded if only writing is required)
            
        *   E → P3.2
            
    *   **RTC (DS12887):**
        
        *   Data/Address Bus → Connect to the same Port 1 as the LCD (if multiplexing is handled in code) or a separate port. The report suggests Port 1 is used for both, which would require careful control via the CS lines.
            
        *   Chip Select (CS) → P3.3
            
    *   **Input/Output:**
        
        *   Button A → P3.5
            
        *   Button B → P3.4
            
        *   Buzzer → P3.1 (use a transistor driver if needed)
            
        *   Temperature Sensor Output → Port 2 (P2)
            

**B. Software Execution:**

1.  **Development Environment:**
    
    *   Use an 8051-compatible assembler (like ASEM-51 or the assembler in the Keil µVision IDE) to compile the provided assembly code (.asm).
        
    *   A simulator like Proteus is highly recommended for initial testing and debugging without physical hardware.
        
2.  **Code Compilation and Loading:**
    
    *   Create a new project in your IDE.
        
    *   Add the provided assembly code file.
        
    *   Assemble/Build the project to generate a Hexadecimal (.hex) file.
        
    *   Load this .hex file into the program memory of the 8051 microcontroller in your simulator or physical programmer.
        
3.  **Operation:**
    
    *   Power on the system. The LCD should initialize and display the starting time (16:58:55), date (19/10/04), day, and temperature status.
        
    *   **Main Screen:** Observe the time updating in real-time. The format will be either 12-hour or 24-hour based on the initial FORMAT\_SELECT flag.
        
    *   **Accessing Menu:** Press **Button A (P3.5)**. The display will show the first menu option (e.g., "CLOCK FORMAT").
        
    *   **Navigation:** Press **Button A** repeatedly to cycle through the menu options: Clock Format, Stopwatch, Countdown, Tally.
        
    *   **Selection:** Press **Button B (P3.4)** to enter the highlighted function.
        
    *   **Testing Features:**
        
        *   In **Stopwatch** and **Countdown**, use Button B to Pause/Resume and Button A to Exit/Reset.
            
        *   In **Tally**, use Button B to count up and Button A to reset.
            
        *   In **Clock Format**, select between 12H and 24H formats.
            
    *   **Hourly Chime:** Simulate or wait for the time to reach exactly hh:00:00. The buzzer should sound for one second.
        
    *   **Temperature Display:** Vary the input voltage on Port 2 (e.g., using a potentiometer in simulation) to see the status change between "COLD," "WARM," and "HOT."
  


## Project Demo

Watch the demo video: [YouTube Shorts - Real-time-Multifunctional-clock-with-8051-microcontrolle Demo](https://youtube.com/shorts/ftqqADBZWs0)
