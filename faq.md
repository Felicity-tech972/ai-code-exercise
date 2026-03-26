# Frequently Asked Questions (FAQ)

## 1. Getting Started
**Q: How do I run the Task Manager?**
A: Open your terminal in VS Code and type `python cli.py`. Ensure you have Python 3.11 or higher installed.

**Q: Where are my tasks saved?**
A: All tasks are automatically saved to a file named `tasks.json` in your project folder.

## 2. Common Features & Functionality
**Q: What do the priority numbers (1-4) mean?**
A: The system uses a ranking scale: **1** (Low), **2** (Medium), **3** (High), and **4** (Urgent).

**Q: How do I see only my completed tasks?**
A: Use the list command with a status filter: `python cli.py list --status done`.

## 3. Troubleshooting
**Q: Why am I getting a "Warning: Attribute not found" error?**
A: This happens if you try to update a field that doesn't exist (e.g., a typo like `stutus` instead of `status`). Check your spelling and try again.

**Q: I get a "ValueError" when setting priority. What's wrong?**
A: You must enter a number between 1 and 4. The system currently only accepts these specific integer values for priorities.

## 4. Advanced Usage
**Q: Can I sync my tasks between two different computers?**
A: Yes! Our bidirectional sync algorithm uses timestamps to merge local and remote files. Just ensure both computers have accurate system clocks.

**Q: How does the "Importance Score" work?**
A: The system calculates a score based on your priority and due date. Overdue tasks get a +35 point boost to stay at the top of your list.
