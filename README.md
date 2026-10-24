# IT 126 - Module 3 Lesson 2: Local Storage and Database Operations

A Flutter application demonstrating local data persistence using SQLite (`sqflite`), file path resolution (`path`), data model mapping, and the Repository pattern.

---

## Analysis Questions (Page 2)

### 1. What happened to the task titles when you added them to the list?
The task titles were stored in the device's volatile Random Access Memory (RAM) as elements inside a standard Dart `List<String>`. The UI rebuilt to render the updated in-memory collection.

### 2. Why could the application display the tasks while it was running?
The Flutter application process remained alive, holding an active reference to the in-memory list object across widget redraws and lifecycle states.

### 3. What happened after the application process ended and the application was relaunched?
All previously entered task titles were lost. When the operating system terminates the application process, the volatile RAM allocated to that process is cleared.

### 4. Would keeping the list inside a StatefulWidget automatically make the tasks persistent? Explain your answer.
No. A `StatefulWidget` only preserves state during runtime widget builds and screen navigation while the app process is running. It does not write to non-volatile disk storage; restarting the app discards the widget state.

### 5. What would the application need to do to preserve the tasks between sessions?
The application must write data records to non-volatile local storage (such as an SQLite database via the `sqflite` package) and reload the stored records into memory whenever the application launches.

### 6. If the application eventually stored hundreds or thousands of tasks, what problems might arise from managing all the data manually in memory?
* **Excessive Memory Usage:** Retaining thousands of full objects in RAM can trigger memory pressure warnings and OS-level termination.
* **Inefficient Querying & Filtering:** Linear searches ($O(N)$) must process every list element sequentially rather than utilizing indexed database b-tree queries ($O(\log N)$).
* **Absence of Data Integrity & Transactions:** Manual state updates lack ACID transactional guarantees, increasing the risk of data corruption or inconsistent states during crashes.

### 7. How could separating the operations that save and retrieve tasks from the UI make the application easier to maintain?
Separating data access logic into a repository layer enforces the Single Responsibility Principle. The UI focuses purely on presentation and event handling, while the repository encapsulates database initialization, schema interactions, and raw SQL queries. This allows database schemas or querying patterns to change without modifying UI widgets.

---

## Lesson Reflection / Analysis (Page 13)

### 1. Why does the application use SQLite instead of relying only on a Dart list?
SQLite persists structured records onto non-volatile disk storage. This guarantees that user data remains intact after process termination, background memory recycling, or system reboots.

### 2. What is the purpose of the primary key in the tasks table?
The primary key (`id INTEGER PRIMARY KEY AUTOINCREMENT`) assigns a unique integer identifier to every row. This enables precise targeting for update and delete queries using a strict `WHERE id = ?` clause, preventing accidental alterations to matching or duplicate titles.

### 3. How does the repository simplify the UI's interaction with the database?
The repository abstracts database connection management and SQL queries behind clean, strongly typed Dart asynchronous methods (`getAllTasks()`, `insertTask()`, `updateTask()`, `deleteTask()`). The UI consumes and passes typed model instances without needing SQL syntax awareness.

### 4. What would happen if a task were removed from the displayed list but not deleted from SQLite?
The task would disappear from the screen during the current session, but its record would remain on disk. The next time the app opens or reloads from the database, the deleted item would reappear as stale data.

### 5. If the database schema later needed a new column for task priority, what parts of the application might need to change?
1. **Database Schema & Version:** Increment the database version number in `DatabaseHelper` and provide an `onUpgrade` migration strategy (e.g., executing `ALTER TABLE tasks ADD COLUMN priority TEXT`).
2. **Task Model:** Update the `Task` entity by adding the `priority` property, updating its constructor, and adjusting `toMap()` and `fromMap()`.
3. **Task Repository:** Update querying and ordering routines if tasks need to be retrieved or sorted by priority.
4. **User Interface:** Add UI form controls (such as a dropdown or choice chip) to assign priority when adding or updating tasks, and display a visual priority indicator on the task list item.