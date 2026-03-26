#  Development Journal: Task Manager Analysis

## Part 1: Task Creation & Status Updates
- **Main Components:**
    - `models.py`: Defines the `Task` structure and `Enums`.
    - `task_manager.py`: Handles the business logic and user commands.
    - `storage.py`: Manages JSON serialization and file.
- **Execution Flow:** 
    1. `create_task` receives raw data.
    2. Data is converted (e.g., strings to `datetime`).
    3. A `Task` object is instantiated with a unique `uuid`.
    4. `TaskStorage.add_task` pushes the object to a dictionary and triggers a file `save()`.
- **Data Persistence:** Uses a `tasks.json` file. Data is retrieved by decoding JSON dictionaries back into class instances using a custom `TaskDecoder`.

---

## Part 2: Task Prioritization System
- **Initial Understanding:** I thought priorities were just simple integers (1, 2, 3).
- **Deepened Understanding:** Priorities are always controlled via the `TaskPriority(Enum)`. 
- **Key Insights:** 
    - The system defaults to `MEDIUM (2)`.
    - Using an `Enum` prevents "magic numbers" in the code, making it readable (e.g., `TaskPriority.HIGH` instead of just `4`).
    - **Constraint:** If a user inputs a value outside 1-4, the `TaskPriority(value)` call will raise a `ValueError`, which currently isn't caught in `create_task`—a potential crash point.

---

## Part 3: Mapping Data Flow (Task Completion)
- **Process Map:**
    - **Trigger:** `manager.update_task_status(task_id, "done")`
    - **State Change:** `task.mark_as_done()` sets `self.status` to `DONE` and `self.completed_at` to `datetime.now()`.
    - **Persistence:** `storage.save()` converts the Python object to JSON via `TaskEncoder`.
- **Potential Points of Failure:**
    - **File Corruption:** If `tasks.json` is malformed, the `load()` method's generic `except Exception` will catch it but might leave the user with an empty list and no explanation.
    - **Overwrite Risk:** The `save()` method rewrites the *entire* file for every single change, which is inefficient for large task lists.

---

## Part 4: Reflection & Presentation Notes
- **High-Level Architecture:** A **three-tier approach** (Data, Logic, Storage). This separation of concerns means I could swap the JSON storage for a SQL database without changing the `Task` model.
- **Interesting Pattern:** **Custom Serialization.** The use of `JSONEncoder` and `JSONDecoder` to handle non-standard JSON types like `datetime` and `uuid`.
- **Most Challenging Part:** Tracking how the `object_hook` in the decoder re-instantiates objects. It’s a clever way to ensure that data coming from a text file behaves like a real Python object immediately.
`get_statistics ()` method iterates over all tasks three separate times once for status counts, once for priority counts, and once for overdue — when a single loop could handle all three.

---

## Part 5: Algorithm Analysis (Task Priority Scoring)

###  How the Algorithm Works
A **Weighted Scoring System** is used by this algorithm to identify the jobs that are most important. It computes a numerical value for each activity rather than only sorting by date.

- **The Logic (The "Weights"):**
    - **Base Priority:** Urgent (60 pts) vs. Low (10 pts).
    - **Urgency Boost:** Overdue tasks get a massive **+35 point** boost to push them to the top.
    - **Status Penalty:** Completed tasks lose **50 points**, effectively burying them at the bottom.
    - **Contextual Boosts:** Tasks tagged as "blocker" or "critical" get an extra **+8 points**.

###  Data Flow & Sorting Strategy
1. **Calculate:** The `calculate_task_score` function runs through the logic above for a single task.
2. **Decorate:** `sort_tasks_by_importance` creates a list of pairs: `(score, task)`.
3. **Sort:** It sorts the list by the score (highest first).
4. **Undecorate:** It strips the scores away and returns only the sorted list of tasks.

###  Key Insights & Learning Points
- **Single Responsibility Principle:** The code separates the *math* (calculating the score) from the *organization* (sorting the list). This makes it much easier to tweak the scoring logic later without breaking the sorting logic.
- **Efficiency:** By calculating the score once per task before sorting (rather than during the sort), the algorithm stays fast even if the list of tasks grows large.
- **Subjective Logic:** The algorithm "decides" that being **Overdue (+35)** is more important than being **Urgent (+10)**. This is a design choice that reflects the developer's priority on deadlines over categories.

---

## Part 6: Algorithm Analysis (Task Text Parser)

---

###  How the Algorithm Works
This parser is a **Natural Language Processing (NLP)** helper. It transforms a single string into a fully populated `Task` object by hunting for specific "trigger" characters.

- **The Trigger Symbols:**
    - `!` = **Priority:** Recognises both numbers (`!1`) and words (`!urgent`).
    - `@` = **Tags:** Extracts labels like `@work` or `@shopping`.
    - `#` = **Dates:** This is the most complex part. It looks for words like `#tomorrow`, `#friday`, or even specific dates like `#2023-12-25`.

###  The Date Logic (The "Smart" Part)
Unlike a simple search, this algorithm uses a helper function called `get_next_weekday`. 
- **If you type `#mon`:** The code calculates how many days are between "today" and the next Monday and generates a real `datetime` object.
- **Cleanup:** As it finds these pieces, it uses `re.sub` (Regex Substitute) to delete them from the text. This ensures the final task title is clean (e.g., "Buy milk" instead of "Buy milk @shopping !2 #tomorrow").

###  Insights and Learning Points
- **Regex Boundaries:** The code uses `\b` (word boundaries) in its regex. This is a pro move—it prevents the code from accidentally picking up a `!` or `@` that is stuck in the middle of a word.
- **Graceful Failure:** The `try...except` block around the date parsing (`%Y-%m-%d`) is a safety net. If a user types a date format the code doesn't understand, the program doesn't crash; it just skips that specific date.
- **The "Title" Paradox:** Since the title is just "whatever is left over" after the symbols are removed, the order in which you remove things matters to avoid leaving double spaces behind.



---

## Part 7: Algorithm Analysis (Task List Merging)

###  How the Algorithm Works
This is a **Bidirectional Synchronization** algorithm. It ensures that data remains consistent across multiple devices by resolving "conflicts."

- **The Strategy (Last Write Wins):** 
    - The algorithm compares the `updated_at` timestamp of two tasks with the same ID.
    - Whichever task was edited most recently is kept as the "truth".
- **New Task Detection:** 
    - If an ID exists in the "Remote" list but not the "Local" one, it's identified as a new task and added automatically.

###  Data Flow Map
1. **Fetch:** Load `Local List` (from phone) and `Remote List` (from cloud).
2. **Identify:** Match tasks by their unique `ID` (UUID).
3. **Resolve:** 
    - If Timestamps match: Do nothing.
    - If Timestamps differ: Choose the newest one.
4. **Finalize:** Save the new merged list back to both locations.

###  Insights and Learning Points
- **The "Source of Truth":** In a two-way sync, there is no single "master" list; the "truth" is determined by the **timestamp**.
- **Conflict Handling:** Without timestamps, the system wouldn't know which version is correct and might accidentally delete your new notes.
- **Complexity:** This is the hardest part of the app because it requires both systems to have perfectly synchronized clocks. If your phone's clock is wrong, the sync will fail!

---

## Part 8: Project Structure & Technology Stack

###  Directory & Tech Analysis
- **Tech Stack:** Python 3, `argparse` (CLI), `json` (Storage), `re` (Regex), and `datetime`.
- **Architecture:** **MVC-lite (Model-View-Controller).**
    - **Models:** `models.py` (The data shape).
    - **Logic/Controller:** `manager.py` (The decision maker).
    - **Data/Storage:** `storage.py` (The file handler).
- **Entry Point:** The application is designed to be run via a CLI (Command Line Interface), likely starting from a `main.py` (if provided) or by calling `TaskManager`.

---

## Part 9: Feature Plan - Task Export to CSV

### Hypothesis
To implement "Export to CSV," the logic belongs in **`storage.py`** or a new **`exporters.py`**. 

- **Why?** `storage.py` already knows how to iterate through all tasks and handle file writing.
- **Affected Components:**
    - `TaskStorage`: Needs a new `export_to_csv(filename)` method.
    - `TaskManager`: Needs a way to trigger the export.

###  Implementation Plan
1. Use Python’s built-in `csv` module.
2. Create a header row: `id, title, priority, status, due_date`.
3. Loop through `self.get_all_tasks()` and write each task's attributes as a row.

---

## Part 10: Practical Application (The "Abandoned" Rule)

###  The Rule
*"Tasks overdue for >7 days are marked 'Abandoned' UNLESS they are 'High' or 'Urgent' priority."*

###  Implementation Strategy
- **Files to Modify:** 
    1. `models.py`: Add `ABANDONED` to the `TaskStatus` Enum.
    2. `manager.py`: Create a method `cleanup_old_tasks()`.
    
- **Logic Pseudo-code:**
for task in all_tasks:
    if task.is_overdue() and task.priority < TaskPriority.HIGH:
        days_overdue = (datetime.now() - task.due_date).days
        if days_overdue > 7:
            task.status = TaskStatus.ABANDONED
