# Minitalk

A client-server communication program using UNIX signals, developed as part of the 42 curriculum.

---

## Grade

125%

---

## About

**Minitalk** is a project from the 42 School focused on inter-process communication using UNIX signals.

It consists of:

* A **server** that receives messages
* A **client** that sends messages

Communication is done using:

* `SIGUSR1`
* `SIGUSR2`

Each character is transmitted bit by bit, requiring precise synchronization between processes.

---

## How It Works

* The server starts and displays its PID
* The client sends a message using that PID
* Each character is broken into bits
* Bits are sent one by one using signals
* The server reconstructs and prints the message

---

## Project Structure

```
.
├── libft/
├── Makefile
├── client.c
├── client_bonus.c
├── server.c
├── server_bonus.c
```

---

## Usage

### Compilation

Compile mandatory version:

```bash
make
```

Compile bonus version:

```bash
make bonus
```

Or manually:

```bash
cc -Wall -Wextra -Werror server.c -o server
cc -Wall -Wextra -Werror client.c -o client
```

---

### Running

Start the server:

```bash
./server
```
It will display its PID.

Run the client:

```bash
./client <server_pid> "your message here"
```

---

### Bonus

Run bonus version:

```bash
./server_bonus
./client_bonus <server_pid> "message"
```

---

## Bit Transmission

Each character is transmitted **bit by bit** using UNIX signals.

* `SIGUSR1` → represents **0**
* `SIGUSR2` → represents **1**

---

### Client (LSB → MSB)

The client sends bits starting from the **least significant bit (LSB)** to the **most significant bit (MSB)**:

```c
while (i < 8)
{
	if ((c >> i) & 1)
		kill(server_pid, SIGUSR2);
	else
		kill(server_pid, SIGUSR1);
	i++;
}
```

Example with the character `'A'`:

```
'A' = 65
Binary: 01000001
```

Bits sent (LSB → MSB):

```
1 → 0 → 0 → 0 → 0 → 0 → 1 → 0
```

---

### Server (Reconstruction)

The server reconstructs the character using the same bit order:

```c
if (signum == SIGUSR2)
	c |= (1 << i);
i++;
```

* Each received bit is placed at position `i`
* The server rebuilds the character from **LSB → MSB**
* After receiving 8 bits, the character is complete

---

### Synchronization (ACK System)

After receiving each bit, the server sends a signal back to the client:

```c
kill(client_pid, SIGUSR1);
```

The client waits for this acknowledgment before sending the next bit:

```c
while (g_timeout)
	pause();
```

This ensures:

* No signal loss
* Proper synchronization between processes

---

### Visualization

```
Client sends:   1 0 0 0 0 0 1 0
                 ↓ ↓ ↓ ↓ ↓ ↓ ↓ ↓
Bit index:       0 1 2 3 4 5 6 7

Server builds:
c |= (1 << i)

Result: 01000001 → 'A'
```

---

### Important

Both client and server must follow the **same bit order (LSB → MSB)**.
If the order differs, the reconstructed character will be incorrect.

---

## Concepts Covered

* UNIX signals (`kill`, `sigaction`)
* Bitwise operations
* Process communication
* Synchronization
* Low-level data transmission

---

## Notes

* Uses only `SIGUSR1` and `SIGUSR2`
* Built with flags: `-Wall -Wextra -Werror`
* Includes a custom `libft`
* Implements per-bit acknowledgment for reliability

---

## Example
```bash
./client 12345 "Hello, world!"
```
Output on server:
```bash
Hello, world!
```
