# miniOS — Team Contribution Document

**Project:** miniOS — Operating System Simulator  
**Language:** Java (Swing GUI)  
**Team:** Kedar · Sahil · Nandika · Shrinivas · Atharv

---

## Project Overview

miniOS is a two-part Java application that simulates core Operating System concepts:

- **RealOS** — A realistic desktop OS simulator where users open apps (File Manager, Calculator, Text Editor, Browser, etc.) and every action silently triggers real OS mechanisms visible in a live Backend Monitor with 11 tabs.
- **OS_GUI** — A standalone OS concepts simulator with interactive panels for each algorithm (Scheduling, Disk, Paging, Memory, Deadlock, Sync, System Calls, Interrupts).

The project is structured so that each OS concept lives in its own dedicated Java file, making ownership clear and modular.

---

## File Structure

```
miniOS/
├── RealOS.java              — Main OS frame, Kernel, Desktop, Taskbar, BackendMonitor
├── AppWindows.java          — All app windows (File Manager, Calculator, Text Editor, etc.)
├── OS_GUI.java              — OS concepts simulator GUI
│
├── OSProcess.java           — [Kedar]     Process data model
├── ProcessScheduler.java    — [Kedar]     CPU Scheduling algorithms
│
├── MemoryManager.java       — [Sahil]     Memory allocation
├── PageManager.java         — [Sahil]     Page replacement
│
├── DeadlockDetector.java    — [Nandika]   Banker's Algorithm
├── SyncManager.java         — [Nandika]   Mutex / Semaphore / Reader-Writer
│
├── DiskManager.java         — [Shrinivas] Disk scheduling
├── DiskRequest.java         — [Shrinivas] Disk I/O request model
├── InterruptController.java — [Shrinivas] Interrupt handler
│
├── MultiUserManager.java    — [Atharv]    Multi-user management
├── SystemCallHandler.java   — [Atharv]    System call handler
└── OSFileSystem.java        — [Atharv]    File system
```

---

---

# KEDAR — CPU Scheduling & Process Management

## Files
- `OSProcess.java`
- `ProcessScheduler.java`

## OS Concepts Covered
- Process lifecycle (NEW → READY → RUNNING → WAITING → TERMINATED)
- FCFS (First Come First Serve)
- SJF (Shortest Job First)
- Priority Scheduling
- Round Robin with configurable Time Quantum
- Smart auto-selection of algorithm based on process count

## Detailed Explanation

### OSProcess.java
Defines the `OSProcess` class — the fundamental unit of execution in the OS.

| Field | Type | Purpose |
|---|---|---|
| `pid` | int | Unique Process ID (starts at 1000) |
| `name` | String | App name (e.g., "Calculator") |
| `type` | ProcessType | SYSTEM or USER |
| `state` | ProcessState | Current lifecycle state |
| `priority` | int | Random 1–10, used by Priority scheduling |
| `memorySize` | int | Random 10–100 MB, used by MemoryManager |
| `cpuTime` | int | Accumulated CPU time in ms |
| `burstTimeRemaining` | int | Remaining burst; MAX_VALUE for interactive apps |
| `isInteractive` | boolean | True = runs until user closes window |

Also defines two enums:
- `ProcessState`: NEW, READY, RUNNING, WAITING, TERMINATED
- `ProcessType`: SYSTEM, USER

### ProcessScheduler.java
Runs as a background thread. Maintains a `ConcurrentLinkedQueue` as the ready queue.

**Smart Mode (default):**  
Automatically selects the best algorithm based on how many processes are running:

| Process Count | Algorithm Selected | Reason |
|---|---|---|
| 1 | FCFS | Simple, no overhead needed |
| 2 | Priority or FCFS | Uses Priority if priorities differ |
| 3–4 | SJF | Minimizes average waiting time |
| 5+ | Round Robin | Fair time-sharing, prevents starvation |
| 7+ | Round Robin (TQ=50ms) | Shorter quantum for heavier load |

**Manual Mode:**  
User can override via the Scheduler Control window — pick any algorithm and pause/resume the scheduler.

**Algorithm implementations:**
- **FCFS** — polls from front of queue in arrival order
- **SJF** — scans entire queue, picks process with lowest `burstTimeRemaining`
- **Priority** — scans entire queue, picks process with lowest priority number (lower = higher priority)
- **Round Robin** — polls from front, sleeps for `timeQuantum` ms, re-queues if not done

**How it connects to the app:**  
Every time a user double-clicks an app icon, `kernel.createProcess()` is called → `scheduler.addProcess()` is triggered → algorithm re-evaluates → visible in the ⚙ Processes tab and Scheduler stat label in the Backend Monitor.

---

---

# SAHIL — Memory Management & Paging

## Files
- `MemoryManager.java`
- `PageManager.java`

## OS Concepts Covered
- Best Fit memory allocation
- Memory deallocation and compaction warning
- LRU (Least Recently Used) page replacement
- Page fault tracking

## Detailed Explanation

### MemoryManager.java
Manages a simulated 1024 MB RAM pool. Uses a `ConcurrentHashMap<PID, MB>` to track how much memory each process holds.

**Allocation (Best Fit):**  
When a process is created, `allocate()` is called. It checks if `usedMemory + required <= 1024`. If yes, memory is granted and logged. If no, an out-of-memory error is logged.

**Deallocation:**  
When a process is terminated, `deallocate()` removes its entry and frees the memory back to the pool.

**Background thread:**  
Runs every 2 seconds. If memory usage exceeds 80% of total, it logs a compaction warning — simulating OS memory pressure response.

**Monitor integration:**  
The 🧠 Memory tab in the Backend Monitor shows a table with PID, process name, MB allocated, start address (hex), end address (hex), and algorithm. The memory progress bar in the stats row updates live.

### PageManager.java
Simulates virtual memory paging using LRU replacement with 10 available frames.

**LRU Algorithm:**  
Maintains a `List<Integer>` of page numbers in order of use. When a page is accessed:
- If not in frames → page fault, load it. If frames full, remove index 0 (least recently used).
- If already in frames → remove from current position, add to end (mark as most recently used).

**Background thread:**  
Every 500ms, randomly picks a running process and accesses a random page number (0–19), simulating realistic page access patterns.

**Monitor integration:**  
The 📄 Paging tab shows each page number, its frame slot, valid/referenced/modified flags, and algorithm. The Paging stat label shows total page faults and current frame count.

---

---

# NANDIKA — Deadlock Detection & Synchronization

## Files
- `DeadlockDetector.java`
- `SyncManager.java`

## OS Concepts Covered
- Banker's Algorithm (deadlock avoidance)
- Allocation Matrix and Need Matrix
- Safe sequence computation
- Mutex (mutual exclusion locks)
- Semaphores (counting semaphores)
- Reader-Writer lock

## Detailed Explanation

### DeadlockDetector.java
Implements Dijkstra's Banker's Algorithm with 3 resource types:

| Resource | Available |
|---|---|
| CPU slots | 5 |
| File Handles | 8 |
| Network Sockets | 4 |

**On process creation (`registerProcess`):**  
Randomly assigns allocation and max need vectors. Deducts allocated resources from available pool. Logs the allocation to the monitor.

**On process termination (`releaseProcess`):**  
Returns allocated resources to the available pool, then immediately calls `runBankersCheck()`.

**Banker's Safety Check (`runBankersCheck`):**  
1. Computes Need = Max − Allocation for each process
2. Simulates resource allocation using a `work` vector
3. Finds a process whose Need ≤ Work, grants it, adds its allocation back to Work
4. Repeats until all processes finish (SAFE) or no progress (DEADLOCK)
5. Logs the safe sequence or deadlock detection with timestamp

**Monitor integration:**  
The 🔒 Deadlock tab shows two live tables — Allocation Matrix and Need Matrix — plus a scrollable history of every Banker's check result. The stat label turns red on deadlock detection.

### SyncManager.java
Manages three types of synchronization primitives:

**Mutexes (3 named locks):**
- `FileSystem` — acquired by File Manager, Text Editor, Image Viewer
- `SharedMemory` — acquired by Calculator
- `NetworkSocket` — acquired by Web Browser, Music Player

When a process tries to acquire a held mutex, it is logged as BLOCKED. Released on process termination.

**Semaphores (counting, max=3):**
- `PrinterQueue` — decremented on every Calculator button press (simulates shared resource access)
- `DBConnections` — decremented when Browser loads a page (simulates connection pool)

**Reader-Writer Lock:**
- Music Player `play` → `readerEnter()` — multiple readers allowed simultaneously
- Music Player `pause` → `readerExit()`
- Text Editor `save` → `writerEnter()` then `writerExit()` — exclusive write access, blocked if readers active

**Monitor integration:**  
The 🔄 Sync tab shows three stat labels (mutex count, semaphore values, reader/writer state) and a full event log table with timestamp, type, resource, process, and action.

---

---

# SHRINIVAS — Disk Scheduling & Interrupt Handling

## Files
- `DiskManager.java`
- `DiskRequest.java`
- `InterruptController.java`

## OS Concepts Covered
- SSTF (Shortest Seek Time First) disk scheduling
- Disk I/O request queue
- Hardware and software interrupt handling
- IRQ (Interrupt Request) logging

## Detailed Explanation

### DiskRequest.java
Simple data model for a disk I/O request:

| Field | Purpose |
|---|---|
| `operation` | Type of operation: READ, WRITE, OPEN, etc. |
| `sector` | Target disk sector (0–999) |
| `processName` | Which process made the request |

Two constructors: one for system-generated requests (processName defaults to "System"), one for process-specific requests.

### DiskManager.java
Runs as a background thread processing a `ConcurrentLinkedQueue<DiskRequest>`.

**SSTF Simulation:**  
Maintains a `headPosition` (current disk head location). For each request dequeued:
1. Calculates `seekTime = |headPosition - sector|`
2. Sleeps for `min(seekTime, 200)` ms to simulate physical seek time
3. Moves head to the new sector
4. Logs the completed operation with seek time

**How requests are generated:**  
Every file operation (open, read, write) in `SystemCallHandler` automatically adds a `DiskRequest` to the queue. App actions that trigger disk I/O:
- File Manager: open, mkdir, create, unlink
- Text Editor: open, read, write
- Image Viewer: open, read
- Web Browser: socket, connect, read

**Monitor integration:**  
The 💿 Disk I/O tab shows each pending request with operation, sector, process name, status, and calculated seek time. The Disk stat label shows current head position and queue size.

### InterruptController.java
Handles both user-triggered and background hardware interrupts.

**Interrupt types handled:**
| Type | Triggered by |
|---|---|
| Keyboard | Every Calculator button press |
| Disk I/O | File Manager open, Text Editor open/save |
| Network | Web Browser page load |
| Timer | Music Player play button |
| Timer/Keyboard/Mouse/Page Fault/Network | Background random IRQs every 3 seconds |

**Each interrupt:**
1. Increments `interruptCount`
2. Records timestamp, type, source, handled=Yes, sequence number
3. Logs to the system log

**Background thread:**  
Every 3 seconds, with 30% probability, fires a random interrupt from a pool of 6 types and 4 sources — simulating realistic hardware activity.

**Monitor integration:**  
The ⚡ Interrupts tab shows the last 50 interrupts in reverse order with IRQ number, time, type, source, and handled status. The Interrupts stat label shows total count.

---

---

# ATHARV — Multi-User Management, System Calls & File System

## Files
- `MultiUserManager.java`
- `SystemCallHandler.java`
- `OSFileSystem.java`

## OS Concepts Covered
- Multi-user login/logout with role-based access control
- Permission enforcement (ADMIN / USER / GUEST)
- User action audit logging
- System call interception and logging
- File system root directory management

## Detailed Explanation

### MultiUserManager.java
Manages user sessions and enforces role-based permissions.

**Three roles:**

| Role | Permissions |
|---|---|
| ADMIN | Full access — all operations allowed |
| USER | Standard access — no delete operations |
| GUEST | Read-only — write, delete, exec all denied |

**UserSession object tracks:**
- Username and role
- Login timestamp
- Process count (how many apps opened)
- List of all actions taken

**On system boot:**  
A `system` user with ADMIN role is automatically logged in.

**Permission check (`checkPermission`):**  
Called before sensitive operations. GUEST role is denied any action containing "write", "delete", or "exec". Logs ALLOWED or DENIED with role to the audit log.

**Action recording (`recordAction`):**  
Called on every app open, close, file operation, and browse action. Records to the session's action list and the global audit log with timestamp.

**Monitor integration:**  
The 👥 Multi-User tab shows the last 50 audit entries in reverse order: timestamp, username, action, and result (ALLOWED/DENIED/RECORDED).

### SystemCallHandler.java
Intercepts every OS system call made by any application and maintains a complete log.

**System calls handled:**

| Call | Triggered by |
|---|---|
| `exec()` | Any app opening (process creation) |
| `exit()` | Any app closing (process termination) |
| `open()` | File Manager, Text Editor, Image Viewer, Music Player |
| `read()` | Text Editor open file, Image Viewer load, Music Player play, Browser |
| `write()` | Text Editor save |
| `mkdir()` | File Manager new folder |
| `create()` | File Manager new file |
| `unlink()` | File Manager delete |
| `socket()` | Web Browser page load |
| `connect()` | Web Browser page load |

**For file-related calls (open/read/write):**  
Automatically generates a `DiskRequest` and adds it to the DiskManager queue — correctly simulating that file operations require disk I/O.

**Log format:**  
Each entry stores: timestamp (HH:mm:ss), process name, call name with `()`, result ("OK"). Capped at 200 entries.

**Monitor integration:**  
The 📞 Sys Calls tab shows the last 50 calls in reverse order. Every call is also written to the 📋 System Log.

### OSFileSystem.java
Manages the physical root directory for the RealOS file system.

Creates a folder called `RealOS_FileSystem/` in the working directory on first run. All File Manager operations (create file, create folder, delete) happen inside this directory. The 📁 File System tab in the Backend Monitor scans this directory recursively and displays every file and folder with name, type, size, path, and last modified date — showing real files the user creates during the session.

---

---

## How Everything Connects

When a user opens an app, here is the exact chain of events across all five members' code:

```
User double-clicks "Text Editor"
        │
        ▼
[Kedar]   OSKernel.createProcess("Text Editor", USER, ...)
          → new OSProcess(pid=1001, name="Text Editor", ...)
          → ProcessScheduler.addProcess(process)        ← process enters ready queue
          → MemoryManager.allocate(process)             ← [Sahil] 40MB allocated
          → DeadlockDetector.registerProcess(process)   ← [Nandika] alloc/max vectors assigned
          → SyncManager.acquireMutex("FileSystem", ...) ← [Nandika] mutex acquired
          → MultiUserManager.recordAction("open:...")   ← [Atharv] action logged
          → SystemCallHandler.handle("exec", process)   ← [Atharv] exec() syscall logged
        │
        ▼
[Shrinivas] InterruptController.trigger("Disk I/O", "Text Editor")
            → IRQ logged, interrupt count incremented
        │
        ▼
User clicks File → Save
        │
        ▼
[Atharv]  SystemCallHandler.handle("write", process)
          → DiskRequest("WRITE", sector=742, "Text Editor") added to queue
[Shrinivas] DiskManager processes request, seek time calculated, head moves
[Nandika]  SyncManager.writerEnter("Text Editor") → exclusive write lock
           SyncManager.writerExit("Text Editor")  → lock released
        │
        ▼
User closes Text Editor
        │
        ▼
[Kedar]   OSKernel.terminateProcess(process)
          → MemoryManager.deallocate(process)            ← [Sahil] 40MB freed
          → DeadlockDetector.releaseProcess(process)     ← [Nandika] resources returned
          → DeadlockDetector.runBankersCheck()           ← [Nandika] safety check runs
          → SyncManager.releaseMutex("FileSystem", ...)  ← [Nandika] mutex freed
          → MultiUserManager.recordAction("close:...")   ← [Atharv] action logged
          → SystemCallHandler.handle("exit", process)    ← [Atharv] exit() syscall logged
```

All of this is visible live in the Backend Monitor which updates every 500ms.

---

## Monitor Tab Ownership

| Tab | Owner | Data Source |
|---|---|---|
| ⚙ Processes | Kedar | OSProcess list, ProcessScheduler state |
| 🧠 Memory | Sahil | MemoryManager allocation map |
| 💿 Disk I/O | Shrinivas | DiskManager request queue |
| 📄 Paging | Sahil | PageManager frame list |
| 📁 File System | Atharv | OSFileSystem directory scan |
| ⚡ Interrupts | Shrinivas | InterruptController log |
| 📞 Sys Calls | Atharv | SystemCallHandler log |
| 🔒 Deadlock | Nandika | DeadlockDetector allocation/need/history |
| 🔄 Sync | Nandika | SyncManager mutex/semaphore/RW state |
| 👥 Multi-User | Atharv | MultiUserManager audit log |
| 📋 System Log | All | Aggregated kernel log from all components |
