# 🧠 Core CS Concepts

## 📂 Topics Covered

| File | Description | Status |
|------|-------------|--------|
| [Array](array.md) | Array Data Structure | ✅ Done |
| [Time & Space](Time&Space.md) | Complexity Analysis | ✅ Done |
| [Memory Management](Memory_Management.md) | Memory Concepts | ✅ Done |
| [BCNF](BCNF.md) | Database Normalization | ✅ Done |
| [SQL Concept](SQL_Concept.md) | SQL Fundamentals | ✅ Done |
| [File Permission](File%20Permission.md) | Linux Permissions | ✅ Done |
| [Access & Refresh Token](access_refresh_token.md) | Authentication | ✅ Done |
| [SDK](SDK.md) | Software Development Kit | ✅ Done |
| [Interview](Interview.md) | Interview Preparation | ✅ Done |

## 🎯 Quick Reference

### Time Complexity
| Notation | Name | Example |
|----------|------|---------|
| O(1) | Constant | Array access |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Linear search |
| O(n log n) | Linearithmic | Merge sort |
| O(n²) | Quadratic | Bubble sort |
| O(2ⁿ) | Exponential | Recursive Fibonacci |

### Data Structures
- **Array** - Contiguous memory, O(1) access
- **Linked List** - Dynamic size, O(n) access
- **Stack** - LIFO (Last In First Out)
- **Queue** - FIFO (First In First Out)
- **Hash Table** - O(1) average lookup
- **Tree** - Hierarchical structure
- **Graph** - Nodes and edges

### Authentication Flow
```
1. User Login → Server validates
2. Server → Access Token (short-lived) + Refresh Token (long-lived)
3. API Request → Access Token in header
4. Token Expired → Use Refresh Token to get new Access Token
```

## 📚 Resources
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)
- [Visualgo](https://visualgo.net/)
- [GeeksforGeeks](https://www.geeksforgeeks.org/)

---
*Last Updated: March 2026*
