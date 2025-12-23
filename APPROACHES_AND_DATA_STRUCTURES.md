# Approaches and Data Structures Used in Python Project

## Overview
This project contains three main Python applications for productivity management:
1. **To-Do Backend** - Object-Oriented task management system
2. **Timetable Backend** - Functional class scheduling system
3. **todolist.py** - Simple timetable manager (legacy/alternative version)

---

## 1. To-Do Backend (`To-Do Backend.py`)

### Programming Approach
**Object-Oriented Programming (OOP)**
- Uses classes to encapsulate data and behavior
- Follows separation of concerns principle
- Implements data persistence with JSON

### Data Structures

#### Classes
1. **Task Class**
   - **Purpose**: Represents a single task/to-do item
   - **Attributes**:
     - `title` (str): Task name/description
     - `deadline` (str): Due date in "YYYY-MM-DD" format
     - `priority` (str): High/Medium/Low priority level
     - `category` (str): Task category (Study, Work, Personal, etc.)
     - `done` (bool): Completion status
     - `created_at` (str): Creation timestamp
   - **Methods**:
     - `to_dict()`: Serializes task to dictionary for JSON storage

2. **TaskManager Class**
   - **Purpose**: Manages collection of tasks and operations
   - **Attributes**:
     - `tasks` (list): List of Task objects
     - `streak` (int): Consecutive days with completed tasks
     - `last_done_date` (str): Last date a task was completed
     - `filename` (str): JSON file path for persistence
   - **Methods**:
     - `add_task()`: Creates and adds new task
     - `delete_task()`: Removes task by index
     - `edit_task()`: Updates task attributes using **kwargs
     - `mark_task()`: Toggles task completion status
     - `filter_tasks()`: Returns filtered subset using **kwargs
     - `sort_tasks()`: Sorts tasks by specified attribute
     - `save()`: Persists data to JSON file
     - `load()`: Loads data from JSON file

#### Core Data Structures
- **List**: Primary container for tasks (`self.tasks = []`)
  - Allows ordered storage
  - Supports indexing for direct access
  - Easy iteration and modification
  
- **Dictionary**: Used for JSON serialization and updates
  - `to_dict()` converts objects to dictionaries
  - `**updates` pattern for flexible attribute updates
  
- **JSON File**: Persistent storage structure
  ```json
  {
    "tasks": [
      {
        "title": "...",
        "deadline": "...",
        "priority": "...",
        "category": "...",
        "done": false,
        "created_at": "..."
      }
    ],
    "streak": 0,
    "last_done_date": null
  }
  ```

### Key Algorithms

1. **Filtering Algorithm**
   ```python
   results = self.tasks
   for key, value in filters.items():
       results = [t for t in results if getattr(t, key) == value]
   ```
   - Uses list comprehension
   - Dynamically filters by any attribute
   - Time Complexity: O(n × f) where f is number of filters

2. **Sorting Algorithm**
   ```python
   self.tasks.sort(key=lambda t: getattr(t, key) or "")
   ```
   - Uses Python's built-in Timsort (O(n log n))
   - Dynamic attribute-based sorting
   - Handles None values with empty string fallback

3. **Streak Tracking Algorithm**
   - Compares dates to detect consecutive days
   - Resets if gap > 1 day
   - Time Complexity: O(1) per update

---

## 2. Timetable Backend (`Timetable Backend.py`)

### Programming Approach
**Functional/Procedural Programming**
- Uses functions for operations
- Module-level state (global `timetable` list)
- Stateless utility functions

### Data Structures

#### Primary Data Structure
**List of Dictionaries** (`timetable = []`)
- Each dictionary represents a class/event:
  ```python
  {
    "name": "Mathematics",
    "day": "Monday",
    "start": "14:30",
    "end": "16:00"
  }
  ```
- **Advantages**:
  - Simple and lightweight
  - Easy JSON serialization
  - Flexible schema

#### Supporting Data Structures

1. **Day Mapping Structures**
   ```python
   DAYS = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]
   DAY_TO_INDEX = {d: i for i, d in enumerate(DAYS)}
   ```
   - **List**: Ordered days for indexing
   - **Dictionary**: O(1) day-to-index lookup for sorting

2. **Time Representation**
   - Times stored as strings ("HH:MM")
   - Converted to minutes (integers) for comparisons
   - **parse_time()**: "14:30" → 870 minutes (14×60 + 30)

### Key Algorithms

1. **Time Parsing Algorithm**
   ```python
   def parse_time(t):
       h, m = map(int, t.split(":"))
       return h * 60 + m
   ```
   - Converts "HH:MM" to total minutes from midnight
   - Enables numeric comparison
   - Time Complexity: O(1)

2. **Conflict Detection Algorithm**
   ```python
   def conflicts(day, start, end, ignore_index=None):
       s, e = parse_time(start), parse_time(end)
       for idx, cls in enumerate(timetable):
           if cls["day"] == day:
               cs, ce = parse_time(cls["start"]), parse_time(cls["end"])
               if not (e <= cs or ce <= s):  # Overlap condition
                   return True
       return False
   ```
   - **Logic**: Two intervals overlap if NOT (one ends before other starts)
   - **Formula**: Overlap = NOT (end1 ≤ start2 OR end2 ≤ start1)
   - Time Complexity: O(n) where n is number of classes
   - **Optimization**: Only checks same day

3. **Sorting Algorithm**
   ```python
   def sort_timetable():
       timetable.sort(key=lambda x: (DAY_TO_INDEX[x["day"]], parse_time(x["start"])))
   ```
   - **Multi-level sorting**: First by day, then by start time
   - **Tuple comparison**: Python compares tuples element-by-element
   - Time Complexity: O(n log n)

4. **Search Algorithm**
   ```python
   def find_class_index(name, day, start):
       for i, cls in enumerate(timetable):
           if cls["name"] == name and cls["day"] == day and cls["start"] == start:
               return i
       return None
   ```
   - Linear search with multi-field matching
   - Time Complexity: O(n)
   - Returns index or None

5. **Weekly Summary Algorithm**
   ```python
   def weekly_summary():
       totals = {}
       for cls in timetable:
           duration = parse_time(cls["end"]) - parse_time(cls["start"])
           totals[cls["name"]] = totals.get(cls["name"], 0) + duration
   ```
   - Uses dictionary as accumulator
   - Aggregates durations by class name
   - Time Complexity: O(n)
   - Space Complexity: O(k) where k is unique class names

---

## 3. todolist.py (Simple Version)

### Approach
- Identical to Timetable Backend
- Functional programming approach
- Same conflict detection and sorting algorithms
- Code duplication suggests it's an alternative or legacy version

---

## Common Patterns Across All Files

### 1. Data Persistence
**JSON Serialization**
- All applications use JSON for file I/O
- Human-readable format
- Easy debugging and manual editing
- Built-in Python `json` module

### 2. Input Validation
**Time Validation** (Timetable applications)
```python
def validate_time(t):
    try:
        if ":" not in t:
            return False
        h, m = map(int, t.split(":"))
        return 0 <= h <= 23 and 0 <= m <= 59
    except:
        return False
```
- Defensive programming approach
- Try-except for error handling
- Range checking for valid hours/minutes

### 3. Menu-Driven Interface
All applications use:
- Infinite `while True` loop
- Numbered menu options
- Input-based function dispatch
- User-friendly console interaction

### 4. Date/Time Handling
- `datetime` module for current date/time
- String format for storage ("YYYY-MM-DD", "HH:MM")
- Conversion to comparable formats (timestamps, minutes)

---

## Design Patterns Observed

### 1. **Repository Pattern** (To-Do Backend)
- TaskManager acts as repository
- Abstracts data access (save/load)
- Separates business logic from storage

### 2. **Command Pattern** (Menu Systems)
- User input maps to specific operations
- Functions encapsulate commands
- Easy to extend with new commands

### 3. **Data Transfer Object (DTO)** (To-Do Backend)
- `to_dict()` method converts objects for transfer/storage
- Separates internal representation from external format

---

## Time and Space Complexity Summary

### To-Do Backend
| Operation | Time Complexity | Space Complexity |
|-----------|----------------|------------------|
| Add Task | O(1) | O(1) |
| Delete Task | O(n) | O(1) |
| Filter Tasks | O(n × f) | O(m) where m = results |
| Sort Tasks | O(n log n) | O(1) |
| Save/Load | O(n) | O(n) |

### Timetable Backend
| Operation | Time Complexity | Space Complexity |
|-----------|----------------|------------------|
| Add Class | O(n) + O(n log n) | O(1) |
| Conflict Check | O(n) | O(1) |
| Sort | O(n log n) | O(1) |
| Find Class | O(n) | O(1) |
| Weekly Summary | O(n) | O(k) |

---

## Strengths and Weaknesses

### Strengths
1. **Simple and Clear**: Easy to understand code structure
2. **JSON Persistence**: Human-readable, easy to debug
3. **Conflict Detection**: Prevents scheduling overlaps
4. **Flexible Filtering/Sorting**: Dynamic attribute-based operations
5. **Error Handling**: Input validation and try-except blocks

### Weaknesses
1. **No Database**: JSON files not suitable for large datasets
2. **Linear Search**: O(n) searches could be optimized with indexing
3. **Code Duplication**: `todolist.py` and `Timetable Backend.py` are nearly identical
4. **Global State**: Module-level variables in functional code
5. **No Concurrency Control**: File operations not thread-safe

---

## Recommended Improvements

1. **Database Integration**: Use SQLite for better querying and scalability
2. **Indexing**: Add hash maps for O(1) lookups
3. **Code Reuse**: Consolidate duplicate code
4. **Testing**: Add unit tests
5. **Error Messages**: More descriptive error handling
6. **Type Hints**: Add Python type annotations
7. **Configuration**: Move constants to config file
8. **Validation**: Add more robust date/time validation

---

## Conclusion

This project demonstrates two different programming paradigms:
- **OOP** (To-Do Backend): Better for complex state management and extensibility
- **Functional** (Timetable Backend): Simpler for straightforward operations

Both approaches use appropriate data structures (lists, dictionaries) and algorithms (sorting, filtering, conflict detection) for their respective use cases. The code is beginner-friendly and focuses on practical functionality over optimization.
