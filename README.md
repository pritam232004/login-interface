# login-interface

import tkinter as tk
from tkinter import messagebox
import sqlite3
import hashlib

# Create or connect to a database
def connect_db():
    conn = sqlite3.connect('user_data.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS users
                 (username TEXT PRIMARY KEY, password TEXT)''')
    conn.commit()
    conn.close()

# Function to hash passwords
def hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()

# Register function
def register():
    def submit_registration():
        username = username_entry.get()
        password = password_entry.get()
        if username == "" or password == "":
            messagebox.showwarning("Input Error", "Please fill all fields.")
        else:
            conn = sqlite3.connect('user_data.db')
            c = conn.cursor()
            try:
                # Insert the new user into the database
                c.execute("INSERT INTO users (username, password) VALUES (?, ?)", 
                          (username, hash_password(password)))
                conn.commit()
                messagebox.showinfo("Success", "User registered successfully!")
                conn.close()
                register_window.destroy()
            except sqlite3.IntegrityError:
                messagebox.showerror("Error", "Username already exists!")
    
    # Create registration window
    register_window = tk.Toplevel(root)
    register_window.title("Register")

    tk.Label(register_window, text="Username").pack()
    username_entry = tk.Entry(register_window)
    username_entry.pack()

    tk.Label(register_window, text="Password").pack()
    password_entry = tk.Entry(register_window, show="*")
    password_entry.pack()

    tk.Button(register_window, text="Register", command=submit_registration).pack()

# Login function
def login():
    def submit_login():
        username = login_username_entry.get()
        password = login_password_entry.get()
        if username == "" or password == "":
            messagebox.showwarning("Input Error", "Please fill all fields.")
        else:
            conn = sqlite3.connect('user_data.db')
            c = conn.cursor()
            c.execute("SELECT * FROM users WHERE username = ? AND password = ?", 
                      (username, hash_password(password)))
            user = c.fetchone()
            conn.close()

            if user:
                messagebox.showinfo("Success", "Login successful!")
                login_window.destroy()
            else:
                messagebox.showerror("Error", "Invalid username or password!")

    # Create login window
    login_window = tk.Toplevel(root)
    login_window.title("Login")

    tk.Label(login_window, text="Username").pack()
    login_username_entry = tk.Entry(login_window)
    login_username_entry.pack()

    tk.Label(login_window, text="Password").pack()
    login_password_entry = tk.Entry(login_window, show="*")
    login_password_entry.pack()

    tk.Button(login_window, text="Login", command=submit_login).pack()

# Main window
root = tk.Tk()
root.title("Register and Login System")

# Main menu
tk.Button(root, text="Register", command=register).pack(pady=50)
tk.Button(root, text="Login", command=login).pack(pady=50)

# Start the Tkinter event loop
connect_db()
root.mainloop()
