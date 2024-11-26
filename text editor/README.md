# Documentation for Simple Text Editor in C

---

## Overview

This program is a foundational implementation for a simple text editor written in C. It operates by configuring the terminal into "raw mode" to capture user input directly without line buffering or processing by the terminal. The text editor reads and displays character input, including special control characters, and exits when the `q` key is pressed.

---

## Features

- **Raw Mode Terminal Handling**: Captures raw input from the user, bypassing standard terminal processing (e.g., line buffering, canonical mode).
- **Character Inspection**: Displays ASCII values of characters, including control characters, in real-time.
- **Graceful Exit**: Restores the terminal's original settings when the program exits.

---

## Code Components

### 1. **Includes**
```c
#include <ctype.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <termios.h>
#include <unistd.h>
```
The program includes standard libraries for:
- **ctype.h**: Character type functions like `iscntrl()`.
- **errno.h**: Error handling.
- **stdio.h**: Input/output operations.
- **stdlib.h**: General utility functions like `exit()`.
- **termios.h**: Terminal I/O settings.
- **unistd.h**: POSIX API, including system calls like `read()`.

---

### 2. **Data Structures**
#### `struct termios orig_termios`
Stores the original terminal settings, allowing them to be restored upon program exit.

---

### 3. **Functions**

#### `void die(const char *s)`
**Purpose**: Handles critical errors by printing an error message and exiting the program.

**Parameters**:
- `const char *s`: Custom error message to display.

**Behavior**:
- Calls `perror()` to append a detailed error message.
- Exits the program with status `1`.

---

#### `void disableRawMode()`
**Purpose**: Restores the terminal to its original settings.

**Behavior**:
- Sets the terminal attributes to the previously saved `orig_termios` settings using `tcsetattr()`.
- Exits with an error if the operation fails.

---

#### `void enableRawMode()`
**Purpose**: Configures the terminal to "raw mode," disabling standard processing features like canonical mode and echoing.

**Behavior**:
1. Retrieves the current terminal settings via `tcgetattr()`.
2. Registers `disableRawMode()` to be called automatically upon program exit using `atexit()`.
3. Modifies terminal attributes:
   - Disables input flags: `BRKINT`, `ICRNL`, `INPCK`, `ISTRIP`, `IXON`.
   - Disables output processing: `OPOST`.
   - Configures 8-bit character size: `CS8`.
   - Disables local flags: `ECHO`, `ICANON`, `IEXTEN`, `ISIG`.
   - Sets control characters for non-blocking input (`VMIN = 0`, `VTIME = 1`).
4. Applies the modified attributes using `tcsetattr()`.

**Error Handling**:
- Exits with an error if terminal attribute modification fails.

---

### 4. **Main Function**
**Purpose**: Entry point for the program.

**Workflow**:
1. **Enable Raw Mode**: Activates raw mode for the terminal using `enableRawMode()`.
2. **Input Loop**:
   - Continuously reads one character from standard input (`STDIN_FILENO`) using `read()`.
   - Displays the ASCII value of the character. If the character is non-printable (control character), it prints the value only. Otherwise, it prints the value and the character itself.
   - Exits the loop if the user inputs the `q` character.
3. **Graceful Exit**:
   - Restores terminal settings via `disableRawMode()` (automatically registered).

---

## Example Output
### Input: `a`
```plaintext
97 ('a')
```

### Input: `CTRL+C` (control character)
```plaintext
3
```

### Input: `q`
```plaintext
113 ('q')
```
*Exits program.*

---

## Error Handling
- **`read()` Errors**: Displays an error message and exits if reading from standard input fails.
- **Terminal Attribute Errors**: Ensures terminal is restored and exits with appropriate error messaging if `tcgetattr()` or `tcsetattr()` fails.

---

## Usage
1. Compile the program:
   ```bash
   gcc -o text_editor text_editor.c
   ```
2. Run the program:
   ```bash
   ./text_editor
   ```
3. Type characters to see their ASCII values. Press `q` to exit.

---

## Future Enhancements
- Add support for basic text editing commands (e.g., insert, delete).
- Implement file reading and writing capabilities.
- Include cursor movement and text display in a scrollable terminal window.

---

## References
- **`termios` Documentation**: [Linux `termios` man page](https://man7.org/linux/man-pages/man3/termios.3.html)
- **POSIX API**: [POSIX.1-2017 Standard](https://pubs.opengroup.org/onlinepubs/9699919799/)
