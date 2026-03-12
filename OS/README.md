# 💻 Operating System Concepts

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [Basics](Basics.md) | OS Fundamentals | ✅ Done |

## 🎯 Quick Reference

### Process vs Thread
| Feature | Process | Thread |
|---------|---------|--------|
| Memory | Separate | Shared |
| Creation | Heavy | Light |
| Communication | IPC needed | Direct |
| Crash Effect | Isolated | Affects all threads |

### Process States
```
New → Ready → Running → Terminated
              ↓    ↑
           Waiting
```

### CPU Scheduling Algorithms
| Algorithm | Description |
|-----------|-------------|
| FCFS | First Come First Serve |
| SJF | Shortest Job First |
| Round Robin | Time quantum based |
| Priority | Based on priority value |

### Memory Management
- **Paging** - Fixed size blocks
- **Segmentation** - Variable size blocks
- **Virtual Memory** - Disk as extended RAM
- **Page Replacement** - FIFO, LRU, Optimal

### Deadlock Conditions (All 4 required)
1. **Mutual Exclusion** - Resource held exclusively
2. **Hold and Wait** - Hold one, wait for another
3. **No Preemption** - Can't forcibly take resource
4. **Circular Wait** - Circular chain of waiting

### File System
```
/
├── bin/    # Essential binaries
├── etc/    # Configuration files
├── home/   # User directories
├── var/    # Variable data
└── tmp/    # Temporary files
```

## 📚 Resources
- [Operating System Concepts (Dinosaur Book)](https://www.os-book.com/)
- [MIT OpenCourseWare - OS](https://ocw.mit.edu/)

---
*Last Updated: March 2026*
