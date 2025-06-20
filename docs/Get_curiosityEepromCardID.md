## **🏷️ Title: Card ID Retrieval for Curiosity Card** 

**🆔 Story ID: CUR-132**  
**👤 Author: Narasapuram Yaswanth**   
**🗓️ Date: 17-06-2025**

---

## 🔍 1. Overview
- This feature enables retrieval of the unique Card ID from the Curiosity card. This ID is used by the Galaxy application to fetch corresponding card metadata.

---

## 📦 2. Scope

This feature focuses exclusively on the retrieval of the Card ID for a Curiosity Card. While the Card ID is expected to be stored in the EEPROM, in the current Erlang implementation, it is fetched from an environment variable. The Card ID acts as a critical identifier that enables Galaxy applications to link and retrieve relevant card metadata.

**✅ In Scope**

- Reading the 2-byte Card ID from EEPROM using the I2C protocol (In Brief).
- Defining and exposing a new Erlang property **`curiosityEepromCardID`** that reads the Card ID from environment variables.
- Handling valid inputs structured as **`[?INDEX_TUPLE, ?INTERFACE_TUPLE]`** and returning appropriate output format.
- Implementing input validation and error responses for incorrect or malformed requests.

---

## 🔁 3. Inputs and Outputs

**📥 Inputs:**

- Input to **get_curiosityEepromCardID** property is list of two tuples: **`[ ?INDEX_TUPLE, ?INTERFACE_TUPLE ]`**
    1. ?INDEX_TUPLE ->
        - **`{index,[ {node,_Node_number}, {rack, _Rack}, {subrack, _Subrack}, {slot, _Slot} ]}`**

    2. ?INTERFACE_TUPLE ->
        - **`{ interface, List_of_interfaces }`**

**📤 Outputs:**

- Card ID is 2-byte hexadecimal.
- On success the output looks like: **`[ { success }, [ { Property_name , Card_id_from_eeprom } ] ]`**

> Property_name -> **curiosityEepromCardID** and Card_id_from_eeprom -> **0x5C**

---

## 🛠️ 4. Design Approach

To enable the Erlang node to access card-related information during system initialization, the card ID is first retrieved from EEPROM at the C layer and made available to the Galaxy application through environment variables present in the **`vim.args`** file.

**Details:**

- Added function: **`get_curiosityEepromCardID()`** in **`curiosity_mif.erl`** module.
- This function calls **`?COMMON_FUNCTION_LIB:read_environment_variable_value(card_id_from_eeprom)`** to fetch the Card ID from the environment variables using the **`card_id_from_eeprom`** atom.
- The returned value is stored in the **`Card_id_from_eeprom`** variable and is used for further validation or processing logic as needed by the application.

> **Note:** If you want a clear explanation of how the card-related information is stored in the EEPROM, go to..[Retrieving_EEPROM_Card_details](Getting_Card_Deatils_From_EEPROM.md)

---

## 🔄 5. Sequence Flow for get_curiosityEepromCardId/1
   
1. **Input:**  
  The function is called with a single argument which is a tuple of the form  
   > **`[ ?INDEX_TUPLE, ?INTERFACE_TUPLE ]`**

2. **Property Assignment:**  
  The variable **`Property_name`** is assigned the atom **`curiosityEepromCardId`**.

3. **Pattern Matching on `?INTERFACE_TUPLE`:**  
    - The function matches the value of **`?INTERFACE_TUPLE`**:
    - If it matches **`{ interface, [0] }`**:
        - Calls **`?COMMON_FUNCTION_LIB:read_environment_variable_value(card_id_from_eeprom)`** to get **`Card_id_from_eeprom`**.
    - Otherwise, if it matches **`{ interface, _any }`** (anything other than **`[0]`**):
        - Logs an "Invalid Argument" error and returns an error tuple.

4. **Check if `Card_id_from_eeprom` is an integer:**
    - If **`Card_id_from_eeprom`** **is an integer**:
        - Return **`[ { success }, [ { curiosityEepromCardId, Card_id_from_eeprom } ] ]`**
    - If **not an integer**:
        - Prints an error indicating the environment variable is not an integer and returns,
        > **`[ { error, card_id_from_environment_variable_is_not_an_integer_value }, [ ] ]`**

5. **If the initial argument does not match the expected tuple structure:**  
    - prints an "Invalid Argument" error and returns,
    > **`[ { error, invalid_argument }, [ ] ]`**

---

## 🚨 6. Error Handling

- Failure Scenarios in Erlang:
    - If invalid arguments are passed to **`get_card_details_from_eeprom()`**, an error message is printed and the function returns:
    > **[ { error, invalid_argument }, [ ] ]**
    - If **`?INTERFACE_TUPLE`** is not **`{interface, [0]}`**, an error message is printed and the function returns:
    > **[ { error, invalid_argument }, [ ] ]**
    - If the return value from **`?COMMON_FUNCTION_LIB:read_environment_variable_value(card_id_from_eeprom)`** is not an integer, an error message is printed and the function returns:
    > **[ { error, card_id_from_environment_variable_is_not_an_integer_value }, [ ] ]**

---

## ⚙️ 7. Configuration or Constants

1. `-define CURIOSITY_EEPROM_CARD_ID 92`
2. `-define CURIOSITY_I2C_BUS_0 0`
3. `-define CURIOSITY_I2C_BUS_1 1`
4. `-define COMMON_FUNCTION_LIB galaxy_common_function`

---

## 📂 8. Impacted Files

- Modified:
    - start_uo.c
- Added:
    - This property is added in curiosity_mif.erl

---

## 🧪 9. Unit Test Summary

#### Acceptance Test Cases:

- `CUR-1` Retrieve EEPROM Card ID with valid interface number.
- `CUR-2` Retrieve EEPROM Card ID with valid interface number.

---

## 🔗 10. References

- Jira Story CUR-132​
- Commit: a1b2c3d - card_id: add support for ID retrieval​
- I2C Protocol Ref Sheet​ 

