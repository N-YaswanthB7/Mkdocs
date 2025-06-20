**🏷️ Title: Card Details Retrieval for Curiosity from EEPROM**  
**👤 Author: Narasapuram Yaswanth**  
**🗓️ Date: 19-06-2025**

---

## 🔍 1. Overview
- This feature enables retrieval of the Card details from EEPROM for Curiosity card. The retrieved information will be used by the Galaxy application.

---

## 📦 2. Scope
This feature focuses on the retrieval of card data stored in the EEPROM of a Curiosity Card. The retrieved information is essential for Galaxy applications to link and access relevant metadata associated with the card.

---

## 🔁 3. Inputs and Outputs

**📥 Inputs:**

- Input to **get_card_details_from_eeprom()** function is :```&card_details_from_eeprom_return``` which is a structure variable
    1. struct card_details_from_eeprom
        {
            unsigned short 	card_id;
            char         	card_name[55];
	        char         	part_code[65];
	        char         	internal_hardware_version[8];
	        char         	internal_software_version[8];
	        char         	external_hardware_version[8];
	        char         	external_software_version[8];
	        unsigned int 	card_uid;
	        char         	mac_compatible_48_bit_uid[20];
	        unsigned char 	m24aa256uid_device_address;
        };

**📤 Outputs:**

- The function get_card_details_from_eeprom() returns a status code:

    - `SUCCESS` (typically defined as 0) if all EEPROM reads and data formatting operations succeed.

    - `FAILURE` (typically defined as -1) if any error occurs during EEPROM probing, reading, or data formatting.

---

## 🛠️ 4. Design Approach

The `start_up.c` file retrieves card-related information from EEPROM using the `get_card_details_from_eeprom()` function, which reads multiple fields from an EEPROM device connected via I2C_BUS_0, and stores the information in the `vm.args` file.

**Details:**

- It first probes the EEPROM device address using a shell command (`i2cdetect`) executed via `system()`. The exit code stored in `eeprom_probe_status`. The detected address is then stored in `command_error_code` using `WEXITSTATUS(eeprom_probe_status)`.
- Several blocks of memory are read from the EEPROM using `read_multiple_bytes_from_m24aa256uid()`:
    - Card ID (2 bytes)
    - Card Name (54 bytes)
    - Part Code (64 bytes)
    - Card Version (8 bytes)
- Versions are formatted into string representations of `major.minor` versions for internal/external hardware/software.
- The function reads a 4-byte UID and a 6-byte 48-bit unique identification number (EUI-48) for MAC compatibility.
- EUI-48 is validated to be non-zero to confirm a valid EEPROM.
- The MAC address is formatted into a standard colon-separated hex string.
- Any failure during I2C read or formatting causes the function to return `FAILURE`.

---

## 🔄 5. Step-wise Flow

**System Initialization Flow:**  
1. Triggered from system initialization.  
2. Executes `start_up.c`.  
3. Insdie `start_up.c`, it calls `get_card_details_from_eeprom()`.

**Inside `get_card_details_from_eeprom()` Function:**  
4. Execute shell command to probe EEPROM I2C device address.  
5. Validate device address.  
6. Then call to `read_multiple_bytes_from_m24aa256uid()` to:
    - Read card ID bytes from EEPROM.  
    - Read card name bytes from EEPROM.  
    - Read part code bytes from EEPROM.  
    - Read card version bytes from EEPROM.  
7. Format version numbers into strings.
8. Then call to `read_uid_from_m24aa256uid()` to: 
    - Read card UID bytes from EEPROM.  
9. Then call to `read_eui48_from_m24aa256uid()` to: 
    - Read 48-bit EUI-48 bytes from EEPROM.  
10. Validate EUI-48 is non-zero.  
11. Format MAC-compatible 48-bit UID string.  
12. Return `SUCCESS` if all operations succeed; otherwise `FAILURE`.

---

## 🚨 6. Error Handling

- If the EEPROM probe command fails or returns an invalid address, it prints an error message to the console and return `FAILURE`.
- If any of the EEPROM reads fail, the function returns `FAILURE`.
- If formatting the version strings or MAC string fails (e.g., `sprintf` returns an error), it prints an error message to the console and return `FAILURE`.
- If the EUI-48 read value is all zeros, indicating invalid data, prints an error message to the console and return `FAILURE`.

These errors ensure the system does not operate on invalid or incomplete card data.

---

## ⚙️ 7. Configuration or Constants

**EEPROM memory address constants:**

- `card_id_start_memory_address = 0x0040`
- `card_name_start_memory_address = 0x0042`
- `card_version_start_memory_address = 0x0078`
- `part_code_start_memory_address = 0x0080`

**Number of bytes to read for each field:**

- Card ID: 2 bytes
- Card Name: 54 bytes
- Card Version: 8 bytes
- Part Code: 64 bytes
- card_uid: 4 bytes
- EEPROM 48-bit EUI (EUI-48): 6 bytes

**Device address probing range:**

- `0x50` to `0x57`

**Use of macros:**

- `#define SUCCESS 0` and `#define FAILURE -1` for status returns
- `#define M24AA256UID_UID_MEMORY_ADDRESS 0x7FFC`
- `#define M24AA256UID_EUI48_MEMORY_ADDRESS 0x7F7A`
- `IS_SPRINTF_RETURNED_ERROR()` for validating `sprintf` results

