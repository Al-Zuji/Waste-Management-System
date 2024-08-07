# Waste-Management-System
## Functionality Overview
The Waste Management System project uses a Raspberry Pi (or similar device) to manage and monitor waste bins in different areas. The system reads the weight of the waste bins via a serial port, displays this information on a graphical user interface (GUI), logs the data into Excel files, and can send reports via Telegram.

<details>
<summary>Key Components</summary>
### Key Components
1- Serial Communication:
* The system reads data from a weight sensor connected via a serial port (/dev/ttyUSB0).
* The data includes the current weight of a bin.
  
2- Graphical User Interface (GUI):

* Built using tkinter, the GUI displays the current date, time, and weight information.
* Users can select the area and enter the bin number.
* The GUI provides buttons to save the data and send reports.
  
3- Data Logging:

* The weight data is logged into Excel files using the openpyxl library.
* Each area has its own Excel file where the data is saved.
  
4- Telegram Integration:

* The system can send a daily report via Telegram using the telebot library.
* The report includes the total net weight of the waste for each area.
</details>
<details>

<summary>Detailed Functionality</summary>

1- Initialization:

The application window is set to fullscreen mode and configured with a white background.
Serial communication and the Telegram bot are initialized.

2- Variable Setup:

* Arrays and variables for storing area names, total weights, current weight, etc., are initialized.
* StringVar and IntVar are used for dynamic values in the GUI.

3- User Interface Setup:

* Labels, entry fields, radio buttons, and buttons are created and placed on the GUI.
* The GUI shows the current time, the gross weight of the bin, and the net waste weight.

4- Clock and Data Updates:

* The clock updates every second.
* The weight sensor data is read from the serial port and updated on the GUI every second.

5- Data Submission:

* When the user submits the data, it is saved into an Excel file corresponding to the selected area.
* The total net weight for the area is updated and displayed.

6- Report Sending:

* The system generates a daily report with the total net weight for each area and sends it via Telegram.
* After sending the report, the total weights are reset for the next day.

</details>

<details>

<summary>Workflow</summary>

1- Startup:

* The application starts and the GUI is displayed in fullscreen mode.
* The clock and sensor data updates begin.

2- User Interaction:

* The user selects an area and enters the bin number.
* The user can see the current weight of the bin and the calculated net waste weight.
* The user can save the data or send a report via Telegram.

3- Data Logging:

* When the data is saved, it is logged into the corresponding Excel file.
* The total net weight for the selected area is updated.

4- Reporting:

* At the end of the day, the user can send a report via Telegram, which includes the total net weight for each area.
* The Excel files are reset for the next day's data logging.

This system allows efficient management and monitoring of waste bins across multiple areas, with automated data logging and reporting capabilities.

</details>

## Stage 1: Importing Libraries
```
import serial
import telebot
from tkinter import *
from time import strftime
from tkinter.ttk import *
from openpyxl import Workbook, load_workbook
```
the necessary libraries are imported:

* 'serial' for serial communication.
* 'telebot' for interacting with the Telegram API.
* 'tkinter' and tkinter.ttk for creating the graphical user interface (GUI).
* 'strftime' from the time module for formatting date and time.
* 'openpyxl' for working with Excel files.

## Stage 2: Initializing the Class
```
class WasteManagementApp:
    def __init__(self, root):
        self.root = root
        self.root.title('Waste Management System')
        self.root.attributes('-fullscreen', True)
        self.root.configure(bg='white')

```
* A class named WasteManagementApp is created, which initializes the GUI window with a title, fullscreen mode, and white background.

## Stage 3: Setting Up Serial and Telegram
```
        self.ser = serial.Serial(port='/dev/ttyUSB0', baudrate=9600, parity=serial.PARITY_NONE,
                                 stopbits=serial.STOPBITS_ONE, bytesize=serial.EIGHTBITS, timeout=1)
        self.tb = telebot.TeleBot('YOUR_TOKEN_HERE')
        self.channel_id = 'YOUR_ID_HERE'
```
* Initializes the serial connection to read data from a device connected to /dev/ttyUSB0.
* Initializes the Telegram bot with the provided token and channel ID for sending reports.

## Stage 4: Setting Up Variables
```
        self.setup_variables()
        self.setup_ui()
        self.start_clock()
        self.update_sensor_data()
        self.update_tong_bin()
        self.update_data()
```
* Calls methods to set up variables, the user interface, the clock, sensor data updates, and other periodic updates.

## Stage 5: Variable Setup
```
    def setup_variables(self):
        self.arr_str = ["Num", "Date", "Time", "T/No", "G.w(kg)", "T.w(kg)", "N.w(kg)"]
        self.nama_kawasan = ["AREA1", "AREA2", "AREA3", "AREA4", "AREA5", "AREA6", "AREA7", "AREA8", "AREA9"]
        self.total_weights = [0.0] * len(self.nama_kawasan)
        self.current_weight = 0.0
        self.bin_count = 1
        self.tong_num = StringVar()
        self.selected_area = StringVar()
        self.row = 2
```
* Initializes lists and variables to store information about areas, weights, bin numbers, and more.

## Stage 6: Setting Up the User Interface
```
    def setup_ui(self):
        self.lbl = Label(self.root, font=('calibri', 100, 'bold'), background='white', foreground='black')
        ...
        self.place_widgets()
```
* Creates various labels, entry widgets, radio buttons, and buttons for the GUI.
* Calls place_widgets to arrange these widgets on the screen.

## Stage 7: Placing Widgets
```
    def place_widgets(self):
        self.lbl.place(x=50, y=50)
        ...
```
* Positions the widgets on the screen using the place method.

## Stage 8: Clock and Sensor Data Updates 
```
    def start_clock(self):
        self.update_time()
        self.root.after(1000, self.start_clock)

    def update_time(self):
        string = strftime("%I:%M:%S %p %d:%m:%y")
        self.lbl.config(text=string)
```
* Starts the clock and updates the time every second.

## Stage 9: Updating Sensor Data
```
    def update_sensor_data(self):
        if self.ser.inWaiting() > 0:
            x = self.ser.readline().decode("utf-8").split()
            self.current_weight = float(x[1])
            self.lbl1.config(text=f"{self.current_weight} KG")
            net_weight = self.current_weight - 13.4
            self.lbl3.config(text=f"Waste Weight: {net_weight:.2f} KG")
        self.root.after(1000, self.update_sensor_data)
```
* Reads data from the serial port, updates the current weight, and displays it on the GUI every second.

## Stage 10: Submitting Data
```
    def submit_data(self):
        area = self.selected_area.get()
        if not area:
            return

        wb_path = f"/home/sailcott/Weight/{area}.xlsx"
        wb = load_workbook(wb_path)
        sheet = wb.active

        date = strftime("%d/%m/%Y")
        time = strftime("%I:%M:%S %p")
        total_weight = self.current_weight - 13.4

        data = [self.row - 1, date, time, self.tong_num.get(), self.current_weight, 13.4, total_weight]
        for col, value in enumerate(data, 1):
            sheet.cell(self.row, col).value = value

        self.total_weights[self.nama_kawasan.index(area)] += total_weight
        self.total_lbl.config(text=f"{self.total_weights[self.nama_kawasan.index(area)]:.2f}")

        self.tong_num.set("")
        self.row += 1
        wb.save(wb_path)
```
* Submits data to an Excel file and updates the GUI.

## Stage 11: Sending Report
```
    def send_report(self):
        area = self.selected_area.get()
        if not area:
            return

        wb_path = f"/home/sailcott/Weight/{area}.xlsx"
        wb = load_workbook(wb_path)
        sheet = wb.active
        sheet.cell(self.row, 1).value = "Total Net Weight"
        sheet.cell(self.row, 7).value = self.total_weights[self.nama_kawasan.index(area)]
        wb.save(wb_path)

        self.total_weights[self.nama_kawasan.index(area)] = 0
        self.total_lbl.config(text="0.00")
        self.tb.send_message(self.channel_id, text=f"Hello guys!!\nNow is {strftime('%d/%m/%Y %I:%M %p')}\nand this is today's report")
        self.tb.send_document(self.channel_id, document=open(wb_path, 'rb'))
        self.row = 2
        urm = wb['Sheet']
        for row in urm.iter_rows():
            for cell in row:
                cell.value = None
        wb.save(wb_path)
        self.init_excel_file(wb_path)
```
* Sends a report via Telegram and resets the total weights.

## Stage 12: Initializing Excel File
```
    def init_excel_file(self, wb_path):
        wb = load_workbook(wb_path)
        sheet = wb.active
        for col, title in enumerate(self.arr_str, 1):
            sheet.cell(1, col).value = title
        wb.save(wb_path)
```
* Initializes an Excel file with column headers.

## Stage 13: Running the Application
```
if __name__ == "__main__":
    root = Tk()
    app = WasteManagementApp(root)
    root.bind("<Escape>", exit)
    root.mainloop()
```
* Starts the application.

# Installing Required Libraries
To install the required libraries, you can use pip. Open your terminal or command prompt and run the following commands:
```
pip install pyserial
pip install pyTelegramBotAPI
pip install tk
pip install openpyxl
```
This will install the serial, telebot, tkinter, and openpyxl libraries needed for your project.
