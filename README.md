# 🛒 Shopping List App

A simple **frontend-only Shopping List** built with vanilla HTML, CSS, and JavaScript. Items are saved to the browser's **localStorage** — so they persist even after you close the tab.

No backend. No database. No frameworks. Just pure browser tech.

---

## 🗂️ Project Structure

```
shopping-list-main/
├── index.html      ← Page structure (form, list, filter, clear button)
├── script.js       ← All app logic (add, edit, delete, filter, localStorage)
├── style.css       ← Styling and responsive layout
└── images/
    └── note.png    ← Header icon
```

---

## 🏗️ How the App Works — Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        BROWSER (User)                           │
│                                                                 │
│   Types item → Submits form   Clicks item → Edit mode          │
│   Clicks ✕  → Deletes item    Types in filter → Search         │
│   Clicks "Clear All" → Wipes everything                        │
└──────────────────────────┬──────────────────────────────────────┘
                           │  Events (submit, click, input)
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                       script.js                                 │
│                                                                 │
│  ┌────────────────┐  ┌──────────────────┐  ┌───────────────┐   │
│  │ onAddItem      │  │ onClickItem      │  │ filterItems   │   │
│  │ Submit()       │  │                  │  │               │   │
│  │                │  │ → removeItem()   │  │ Hides/shows   │   │
│  │ Add or Update  │  │ → setItemToEdit()│  │ <li> by text  │   │
│  │ an item        │  │                  │  │               │   │
│  └───────┬────────┘  └────────┬─────────┘  └───────────────┘   │
│          │                   │                                  │
│          ▼                   ▼                                  │
│  ┌─────────────────────────────────────┐                        │
│  │         DOM Manipulation            │                        │
│  │  addItemToDOM() / item.remove()     │                        │
│  └─────────────────────────────────────┘                        │
│          │                   │                                  │
│          ▼                   ▼                                  │
│  ┌─────────────────────────────────────┐                        │
│  │         localStorage                │                        │
│  │  addItemToStorage()                 │                        │
│  │  removeItemFromStorage()            │                        │
│  │  getItemsFromStorage()              │                        │
│  └─────────────────────────────────────┘                        │
└─────────────────────────────────────────────────────────────────┘
                           │
                           ▼
              Items persist across page reloads ✅
```

---

## 🧩 Code Walkthrough — Only the Functional Parts

---

### 1. 🚀 Initializing the App
**File:** `script.js`

```js
function init() {
  itemForm.addEventListener('submit', onAddItemSubmit);
  itemList.addEventListener('click', onClickItem);
  clearBtn.addEventListener('click', clearItems);
  itemFilter.addEventListener('input', filterItems);
  document.addEventListener('DOMContentLoaded', displayItems);

  checkUI();
}

init();
```

**What it does:**
- This is the entry point — called immediately when the script loads.
- Attaches all event listeners to the right elements in one place.
- `DOMContentLoaded` → calls `displayItems()` once the page finishes loading, so saved items from localStorage are shown instantly.
- `checkUI()` → hides the filter and clear button if the list is empty on first load.

---

### 2. 📦 Loading Items from localStorage on Page Load
**File:** `script.js`

```js
function displayItems() {
  const itemsFromStorage = getItemsFromStorage();
  itemsFromStorage.forEach((item) => addItemToDOM(item));
  checkUI();
}

function getItemsFromStorage() {
  let itemsFromStorage;
  if (localStorage.getItem('items') === null) {
    itemsFromStorage = [];
  } else {
    itemsFromStorage = JSON.parse(localStorage.getItem('items'));
  }
  return itemsFromStorage;
}
```

**What it does:**
- `getItemsFromStorage()` — reads the `items` key from localStorage. If nothing is saved yet, returns an empty array. If items exist, parses the JSON string back into an array.
- `displayItems()` — loops through the saved items array and calls `addItemToDOM()` for each one, re-drawing all items on the page after every reload.

---

### 3. ➕ Adding a New Item
**File:** `script.js`

```js
function onAddItemSubmit(e) {
  e.preventDefault();

  const newItem = itemInput.value;

  if (newItem === '') {
    alert('Please add an item !');
    return;
  }

  // If in edit mode, remove the old item first
  if (isEditMode) {
    const itemToEdit = itemList.querySelector('.edit-mode');
    removeItemFromStorage(itemToEdit.textContent);
    itemToEdit.classList.remove('edit-mode');
    itemToEdit.remove();
    isEditMode = false;
  }

  addItemToDOM(newItem);
  addItemToStorage(newItem);
  checkUI();
  itemInput.value = '';
}
```

**What it does:**
- `e.preventDefault()` — stops the form from doing a page reload on submit.
- Validates the input — shows an alert if the field is empty.
- If the user was editing an existing item (`isEditMode = true`), the old item is removed from both the DOM and localStorage before the updated one is added.
- Calls `addItemToDOM()` to show it on screen, and `addItemToStorage()` to save it to localStorage.

---

### 4. 🏗️ Adding an Item to the Page (DOM)
**File:** `script.js`

```js
function addItemToDOM(item) {
  const li = document.createElement('li');
  li.appendChild(document.createTextNode(item));

  const button = createButton('remove-item btn-link text-red');
  li.appendChild(button);

  itemList.appendChild(li);
}

function createButton(classes) {
  const button = document.createElement('button');
  button.className = classes;
  const icon = createIcon('fa-solid fa-xmark');
  button.appendChild(icon);
  return button;
}
```

**What it does:**
- Creates a `<li>` element with the item text and appends a red ✕ button to it.
- `createButton()` builds the delete button with a Font Awesome ✕ icon inside.
- The `<li>` is then added to the `<ul id="item-list">` on the page.
- This same function is used both when adding a new item AND when loading items from localStorage on page load.

---

### 5. 💾 Saving an Item to localStorage
**File:** `script.js`

```js
function addItemToStorage(item) {
  let itemsFromStorage = getItemsFromStorage();

  itemsFromStorage.push(item);

  localStorage.setItem('items', JSON.stringify(itemsFromStorage));
}
```

**What it does:**
- Reads the current items array from localStorage.
- Pushes the new item into the array.
- Converts the array to a JSON string and saves it back under the `'items'` key.
- localStorage only stores strings, so `JSON.stringify()` is needed to save an array.

---

### 6. 🖊️ Editing an Existing Item
**File:** `script.js`

```js
function setItemToEdit(item) {
  isEditMode = true;

  itemList.querySelectorAll('li').forEach((i) => i.classList.remove('edit-mode'));
  item.classList.add('edit-mode');

  formBtn.innerHTML = '<i class="fa-solid fa-pen"></i>  Update Item';
  formBtn.style.backgroundColor = '#228B22';

  itemInput.value = item.textContent;
}
```

**What it does:**
- Sets `isEditMode = true` — a flag that tells `onAddItemSubmit()` to replace an item instead of creating a new one.
- Removes the `edit-mode` class from all items, then adds it only to the clicked item (visually greys it out).
- Changes the "Add Item" button to say "Update Item" and turns it green so the user knows they're in edit mode.
- Fills the input field with the clicked item's text so the user can modify it.

---

### 7. ❌ Deleting a Single Item
**File:** `script.js`

```js
function onClickItem(e) {
  if (e.target.parentElement.classList.contains('remove-item')) {
    removeItem(e.target.parentElement.parentElement);
  } else {
    setItemToEdit(e.target);
  }
}

function removeItem(item) {
  if (confirm('Are you sure !')) {
    item.remove();
    removeItemFromStorage(item.textContent);
    checkUI();
  }
}

function removeItemFromStorage(item) {
  let itemsFromStorage = getItemsFromStorage();
  itemsFromStorage = itemsFromStorage.filter((i) => i !== item);
  localStorage.setItem('items', JSON.stringify(itemsFromStorage));
}
```

**What it does:**
- `onClickItem()` — a single listener on the whole `<ul>`. It checks if the click was on the ✕ icon (delete) or on the item text (edit). This pattern is called **event delegation**.
- `removeItem()` — shows a confirmation dialog, then removes the `<li>` from the DOM if confirmed.
- `removeItemFromStorage()` — filters out the deleted item from the localStorage array and saves the updated array back.

---

### 8. 🧹 Clearing All Items
**File:** `script.js`

```js
function clearItems() {
  while (itemList.firstChild) {
    itemList.removeChild(itemList.firstChild);
  }

  localStorage.removeItem('items');
  checkUI();
}
```

**What it does:**
- The `while` loop keeps removing the first child of the list until there are none left — this empties the `<ul>` completely.
- `localStorage.removeItem('items')` — wipes the entire `items` key from localStorage so items don't come back on reload.
- `checkUI()` then hides the filter box and "Clear All" button since the list is now empty.

---

### 9. 🔍 Filtering Items in Real Time
**File:** `script.js`

```js
function filterItems(e) {
  const items = itemList.querySelectorAll('li');
  const text = e.target.value.toLowerCase();

  items.forEach((item) => {
    const itemName = item.firstChild.textContent.toLowerCase();

    if (itemName.indexOf(text) != -1) {
      item.style.display = 'flex';
    } else {
      item.style.display = 'none';
    }
  });
}
```

**What it does:**
- Fires on every keystroke in the filter input (`input` event).
- Loops through every `<li>` and checks if the item text **contains** the typed text (using `indexOf`).
- If it matches → `display: flex` (visible). If not → `display: none` (hidden).
- This is purely visual — nothing is deleted. Items reappear when you clear the filter.

---

### 10. 👁️ Showing/Hiding UI Elements Smartly
**File:** `script.js`

```js
function checkUI() {
  itemInput.value = '';
  const items = document.querySelectorAll('li');

  if (items.length === 0) {
    clearBtn.style.display = 'none';
    itemFilter.style.display = 'none';
  } else {
    clearBtn.style.display = 'block';
    itemFilter.style.display = 'block';
  }

  formBtn.innerHTML = '<i class="fa-solid fa-plus"></i> Add Item';
  formBtn.style.backgroundColor = '#333';
}
```

**What it does:**
- Called after every add, delete, or clear action.
- Hides the "Clear All" button and the filter input when the list is empty (no point showing them).
- Shows both when at least one item exists.
- Also resets the submit button back to "Add Item" (dark) after finishing an edit, cancelling edit mode visually.

---

## 🔄 Full Flow — Visual Summary

```
Page Load
   │
   ▼
displayItems()
   │── getItemsFromStorage()  ←── localStorage["items"] (JSON array)
   │── addItemToDOM(each)     ──► <ul> rendered on screen
   └── checkUI()              ──► hide/show filter & clear btn

User types + clicks "Add Item"
   │
   ▼
onAddItemSubmit()
   │── validate input
   │── addItemToDOM(item)     ──► new <li> added to <ul>
   └── addItemToStorage(item) ──► localStorage["items"] updated

User clicks item text
   │
   ▼
setItemToEdit()
   │── isEditMode = true
   │── item gets .edit-mode class (grey)
   └── button changes to "Update Item" (green)

User clicks ✕ on item
   │
   ▼
removeItem()
   │── confirm dialog
   │── item.remove()              ──► removed from DOM
   └── removeItemFromStorage()    ──► removed from localStorage

User types in filter box
   │
   ▼
filterItems()
   └── each <li> shown or hidden based on text match (no deletion)

User clicks "Clear All"
   │
   ▼
clearItems()
   │── empties <ul>
   └── localStorage.removeItem('items')
```

---

## 🖼️ UI Overview

```
┌──────────────────────────────────────┐
│  🗒️  Shopping List                   │
│                                      │
│  [ Enter Item            ] [Add Item]│
│                                      │
│  ________________________            │
│  Filter Items                        │
│                                      │
│  ┌────────────┐  ┌────────────┐      │
│  │ Milk    [✕]│  │ Eggs    [✕]│      │
│  └────────────┘  └────────────┘      │
│  ┌────────────┐  ┌────────────┐      │
│  │ Bread   [✕]│  │ Butter  [✕]│      │
│  └────────────┘  └────────────┘      │
│                                      │
│        [ Clear All ]                 │
└──────────────────────────────────────┘
```

---

## 💡 Key Concepts at a Glance

| Concept | Where | What it means |
|---|---|---|
| **localStorage** | `addItemToStorage`, `getItemsFromStorage` | Browser's built-in key-value store — data survives page reloads |
| **JSON.stringify / parse** | Storage functions | localStorage only holds strings, so arrays must be converted to/from JSON |
| **Event Delegation** | `onClickItem` | One listener on the `<ul>` handles clicks for all `<li>` items — no need to attach listeners to each item |
| **isEditMode flag** | `setItemToEdit`, `onAddItemSubmit` | A boolean that switches the form between "add new" and "update existing" mode |
| **DOM Creation** | `addItemToDOM`, `createButton` | Items are built programmatically with `createElement` instead of hardcoded HTML |
| **`indexOf` filter** | `filterItems` | Checks if the typed text appears anywhere in the item name — a simple search implementation |

---

## 🚀 How to Run

No installation needed. Just open the file in a browser.

```bash
# Option 1 — Open directly
open index.html

# Option 2 — Use VS Code Live Server extension
# Right-click index.html → "Open with Live Server"
```

> Items are saved in the browser's localStorage. They will persist across page reloads but are specific to each browser/device.
