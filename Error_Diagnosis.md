Error Analysis 1: [IndexError]
**Error Description**
This means the code is trying to look for an item at a specific "address" in a list that doesn't exist. It's like trying to go to the 4th floor of a 3-story building.

**Root Cause**
The loop is set to range(len(items) + 1). Since Python starts counting at 0, adding that +1 makes the loop try to access one extra index past the end of the list.

**Solution**
Remove the + 1 from the range function so it stops at the correct length:
 for i in range(len(items)):
    print(f"Item {i+1}: {items[i]['name']}")


**Learning Points**
Always remember that Python lists are 0-indexed.
Use range(len(list)) without additions to stay safe.


Error Analysis 2: [MemoryError] 
**Error Description**
The program crashed because it asked the computer for a massive amount of RAM (4.8 GB) that wasn't available. It "ran out of breath" trying to handle too much data at once.

**Root Cause**
The code uses a very "heavy" data type (float64) and tries to keep every single processed image in a list at the same time, instead of clearing them out as it goes.

**Solution**
Use a lighter data type like float32 and process images one by one, saving them to a file instead of keeping them all in a list
image_data = np.zeros((5000, 5000, 64), dtype=np.float32)

**Learning Points****
Scale matters: Consider how much RAM your data takes up.
Batching: Process and save items one by one to keep memory usage low.
