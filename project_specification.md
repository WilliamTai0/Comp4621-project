COMP4621 (Spring 2026) Programming Project: Multiparty Chatroom MASTER SPEC

1. Project Objective

Goal: Develop a server-based multiparty chatroom system.

Learning Outcomes: Socket programming, concurrent servers using poll(), asynchronous I/O using pthread, and reliable data transfer (RDT 3.0) over UDP.

2. Technical Environment & Constraints

Language: C (Standard C libraries).

OS: Linux (Ubuntu/WSL).

Concurrency (Server): MUST use poll(). Using select() or epoll() will result in 0 points.

Asynchrony (Client): MUST use pthread (one thread for sending, one for receiving).

Compilation: Must link with -lpthread (e.g., gcc -o client client.c -lpthread).

Compatibility: Must pass grader_linux.sh tests.

3. Communication Protocols

TCP (Control & Broadcast)

Used for Client-Server connection, Registration, Login, DIR command, and Public Broadcast.

Used for Offline Message delivery from Server to Client.

UDP + RDT 3.0 (Private P2P)

Used for Private Messaging (#Nickname: MSG) between online clients.

Must implement RDT 3.0 (Stop-and-Wait ARQ) to handle packet loss and corruption.

4. Component Requirements

Server (server_skeleton.c)

User Management: Maintain a list of user_info_t (Nickname, IP, Port, FD, State).

Registration/Login: Identify new vs. existing users based on Nickname.

DIR Response: Send formatted list of all users and their status.

Broadcast Logic: Relay messages to all online clients via TCP.

Message Box: Queue messages for offline users; deliver immediately when they log in.

Client (client_skeleton.c)

Two Threads:

recv_server_msg_thread: Handles incoming TCP data from Server.

recv_peer_msg_thread: Handles incoming UDP packets from Peers.

P2P Logic:

Check target status via DIR.

If recipient is Offline: Send to Server immediately via send_offline_to_server().

If recipient is Online: Try P2P via send_to_peer().

If P2P times out (RDT failure): Fallback to send_offline_to_server().

5. RDT 3.0 Implementation Details

Header Structure:

type (16-bit): 0 for DATA, 1 for ACK.

seq (16-bit): 0 or 1.

checksum (16-bit): 16-bit Internet Checksum.

len (16-bit): Payload length.

Timer: 500ms (use RDT_TIMEOUT_USEC).

Retries: Max 10 attempts.

Logic: Sender waits for correct ACK; Receiver checks checksum and sends corresponding ACK.

6. Bonus Features

Bonus 1 (5pts) - ncurses GUI:

Implement a 3-pane interface:

Left: Online friends list.

Bottom: Command input bar.

Center/Top: Message log area.

Bonus 2 (5pts) - Graceful Server Shutdown:

Capture SIGINT (Ctrl+C).

Send a "Server is shutting down" notification to all clients.

Properly close() all sockets and free() allocated memory.

7. Implementation Rules for Grader

Immediate Flush: Use fflush(stdout) after every printf.

Formatting: Output strings must match the provided demo/PDF examples exactly.

Exception Handling: Programs must not crash on invalid input or unexpected disconnection.