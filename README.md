# Task-1
For codsoft
import tkinter as tk
from tkinter import ttk, messagebox
import json
import os
from datetime import datetime

TASKS_FILE = "smart_tasks.json"

# Load tasks from file
def load_tasks():
    if os.path.exists(TASKS_FILE):
        with open(TASKS_FILE, "r") as f:
            try:
                return json.load(f)
            except json.JSONDecodeError:
                return []
    return []

# Save tasks to file
def save_tasks(tasks):
    with open(TASKS_FILE, "w") as f:
        json.dump(tasks, f, indent=4)

class ToDoApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Smart To-Do List")
        self.tasks = load_tasks()

        self.build_ui()
        self.refresh_task_list()

    def build_ui(self):
        input_frame = tk.Frame(self.root)
        input_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(input_frame, text="Task:").pack(side="left")
        self.task_entry = tk.Entry(input_frame, width=30)
        self.task_entry.pack(side="left", padx=5)

        tk.Label(input_frame, text="Priority:").pack(side="left")
        self.priority_var = tk.StringVar()
        self.priority_menu = ttk.Combobox(input_frame, textvariable=self.priority_var, values=["High", "Medium", "Low"], width=7, state="readonly")
        self.priority_menu.set("Medium")
        self.priority_menu.pack(side="left", padx=5)

        tk.Label(input_frame, text="Due (YYYY-MM-DD):").pack(side="left")
        self.due_entry = tk.Entry(input_frame, width=12)
        self.due_entry.pack(side="left", padx=5)

        tk.Button(input_frame, text="Add Task", command=self.add_task).pack(side="left", padx=5)

        list_frame = tk.Frame(self.root)
        list_frame.pack(padx=10, fill="both", expand=True)

        self.task_listbox = tk.Listbox(list_frame, font=("Arial", 12), height=10)
        self.task_listbox.pack(side="left", fill="both", expand=True)

        scrollbar = tk.Scrollbar(list_frame, command=self.task_listbox.yview)
        scrollbar.pack(side="right", fill="y")
        self.task_listbox.config(yscrollcommand=scrollbar.set)

        button_frame = tk.Frame(self.root)
        button_frame.pack(pady=10)

        tk.Button(button_frame, text="Mark Done/Undone", command=self.toggle_done).pack(side="left", padx=5)
        tk.Button(button_frame, text="Delete Task", command=self.delete_task).pack(side="left", padx=5)

    def add_task(self):
        description = self.task_entry.get().strip()
        priority = self.priority_var.get()
        due_date = self.due_entry.get().strip()

        if not description:
            messagebox.showwarning("Input Error", "Task description is required.")
            return

        if due_date:
            try:
                datetime.strptime(due_date, "%Y-%m-%d")
            except ValueError:
                messagebox.showwarning("Date Error", "Invalid date format. Use YYYY-MM-DD.")
                return

        task = {
            "description": description,
            "priority": priority,
            "due": due_date,
            "done": False
        }

        self.tasks.append(task)
        save_tasks(self.tasks)
        self.clear_inputs()
        self.refresh_task_list()

    def clear_inputs(self):
        self.task_entry.delete(0, tk.END)
        self.due_entry.delete(0, tk.END)
        self.priority_menu.set("Medium")

    def refresh_task_list(self):
        self.task_listbox.delete(0, tk.END)
        for idx, task in enumerate(self.tasks):
            text = f"{task['description']} | {task['priority']} | {task['due']} {'✔️' if task['done'] else ''}"
            self.task_listbox.insert(tk.END, text)

            # Apply color based on priority and status
            if task["done"]:
                self.task_listbox.itemconfig(idx, fg="gray")
            elif task["priority"] == "High":
                self.task_listbox.itemconfig(idx, fg="red")
            elif task["priority"] == "Medium":
                self.task_listbox.itemconfig(idx, fg="orange")
            else:
                self.task_listbox.itemconfig(idx, fg="green")

    def toggle_done(self):
        selected = self.task_listbox.curselection()
        if not selected:
            messagebox.showinfo("Select Task", "Please select a task to mark done/undone.")
            return
        index = selected[0]
        self.tasks[index]["done"] = not self.tasks[index]["done"]
        save_tasks(self.tasks)
        self.refresh_task_list()

    def delete_task(self):
        selected = self.task_listbox.curselection()
        if not selected:
            messagebox.showinfo("Select Task", "Please select a task to delete.")
            return
        index = selected[0]
        task = self.tasks.pop(index)
        save_tasks(self.tasks)
        self.refresh_task_list()
        messagebox.showinfo("Deleted", f"Deleted task: {task['description']}")

# Run app
if __name__ == "__main__":
    root = tk.Tk()
    app = ToDoApp(root)
    root.mainloop()
