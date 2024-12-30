import sqlite3
import tkinter as tk
from tkinter import ttk, messagebox, simpledialog

# Utility function to open a DB connection
def get_db_connection():
    """Returns a connection to the SQLite database."""
    return sqlite3.connect('products.db')

# Initialize the database and create the table
def initialize_db():
    """Create the database and products table if they don't exist."""
    with get_db_connection() as conn:
        cursor = conn.cursor()
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                price REAL,
                image_url TEXT
            )
        ''')
        conn.commit()

# Get the next available product ID
def get_next_id():
    """Find and return the smallest unused ID in the products table."""
    with get_db_connection() as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT id FROM products ORDER BY id')
        used_ids = [row[0] for row in cursor.fetchall()]

    next_id = 1
    for used_id in used_ids:
        if used_id != next_id:
            break
        next_id += 1

    return next_id

# Add a product to the database
def add_product(name, price=None, image_url=None):
    """Add a new product to the database."""
    try:
        next_id = get_next_id()
        if price is not None and not isinstance(price, (int, float)):
            raise ValueError("Price must be a number.")
        
        with get_db_connection() as conn:
            cursor = conn.cursor()
            cursor.execute('''
                INSERT INTO products (id, name, price, image_url)
                VALUES (?, ?, ?, ?)
            ''', (next_id, name, price, image_url))
            conn.commit()

        print(f"Product '{name}' added successfully with ID {next_id}.")
        refresh_products()
    except Exception as e:
        print(f"Error adding product: {e}")
        messagebox.showerror("Error", f"Error adding product: {e}")

# Remove a product from the database
def remove_product(product_id):
    """Remove a product by its ID."""
    with get_db_connection() as conn:
        cursor = conn.cursor()
        cursor.execute('DELETE FROM products WHERE id = ?', (product_id,))
        conn.commit()

    print(f"Product with ID {product_id} removed.")
    refresh_products()

# List all products in the database
def list_products():
    """Retrieve all products and display them in the Listbox."""
    with get_db_connection() as conn:
        cursor = conn.cursor()
        cursor.execute('SELECT * FROM products')
        products = cursor.fetchall()

    listbox.delete(0, tk.END)

    if products:
        for product in products:
            price_str = f"${float(product[2]):.2f}" if product[2] else "N/A"
            listbox.insert(tk.END, f"ID: {product[0]}, Name: {product[1]}, Price: {price_str}, Image URL: {product[3]}")
    else:
        listbox.insert(tk.END, "No products found.")

# Function to refresh the displayed products list
def refresh_products():
    """Refresh the list of products shown in the Listbox."""
    list_products()

# Set up the GUI for the application
def setup_gui():
    global listbox, search_entry, root, clear_button

    root = tk.Tk()
    root.title("Product Manager")
    
    # Window size and centering
    window_width = 800
    window_height = 600
    screen_width = root.winfo_screenwidth()
    screen_height = root.winfo_screenheight()

    position_top = int(screen_height / 2 - window_height / 2)
    position_right = int(screen_width / 2 - window_width / 2)

    root.geometry(f'{window_width}x{window_height}+{position_right}+{position_top}')
    root.config(bg="#f5f5f5")
    
    # Center the frame and UI components
    frame = ttk.Frame(root, padding="15")
    frame.place(relx=0.5, rely=0.5, anchor="center")  # Center the frame using place geometry manager

    # Title label
    ttk.Label(frame, text="Product Manager", font=("Helvetica", 18, "bold")).grid(row=0, column=0, columnspan=3, pady=15)

    # Search bar
    ttk.Label(frame, text="Search Products:", font=("Helvetica", 12)).grid(row=1, column=0, sticky="w", padx=10, pady=5)
    search_entry = ttk.Entry(frame, width=50)
    search_entry.grid(row=1, column=1, padx=10, pady=5)
    
    # "X" clear button to cancel search
    clear_button = ttk.Button(frame, text="X", width=2, command=cancel_search)
    clear_button.grid(row=1, column=2, padx=10, pady=5)
    clear_button.grid_forget()  # Hide initially

    # Control Buttons (Add, Remove, Refresh)
    ttk.Button(frame, text="Add Product", width=20, command=add_product_dialog).grid(row=2, column=0, padx=10, pady=10)
    ttk.Button(frame, text="Remove Product", width=20, command=remove_product_dialog).grid(row=2, column=1, padx=10, pady=10)
    ttk.Button(frame, text="Refresh", width=20, command=refresh_products).grid(row=2, column=2, padx=10, pady=10)

    # Listbox for product display
    listbox = tk.Listbox(frame, width=80, height=15, font=("Helvetica", 12), selectmode=tk.SINGLE, bg="#ffffff", fg="#000000", bd=2)
    listbox.grid(row=3, column=0, columnspan=4, padx=10, pady=15)

    # Bind the Enter key to the search function
    search_entry.bind("<Return>", lambda event: on_search(search_entry.get()))

    # Populate the list initially
    refresh_products()

    root.mainloop()

# Add Product dialog for user input
def add_product_dialog():
    name = simpledialog.askstring("Input", "Enter product name:")
    if not name:
        messagebox.showerror("Error", "Name is required!")
        return

    price = simpledialog.askstring("Input", "Enter product price:")
    try:
        price = float(price) if price else None
    except ValueError:
        messagebox.showerror("Error", "Price must be a valid number!")
        return

    image_url = simpledialog.askstring("Input", "Enter product image URL (optional):")
    add_product(name, price, image_url)

# Remove Product dialog for user input
def remove_product_dialog():
    product_id = simpledialog.askinteger("Input", "Enter product ID to remove:")
    if product_id:
        remove_product(product_id)
    else:
        messagebox.showerror("Error", "Product ID is required!")

# Search products by name or ID
def on_search(search_term):
    with get_db_connection() as conn:
        cursor = conn.cursor()
        cursor.execute('''
            SELECT * FROM products WHERE name LIKE ? OR id LIKE ?
        ''', (f"%{search_term}%", f"%{search_term}%"))
        products = cursor.fetchall()

    listbox.delete(0, tk.END)

    if products:
        for product in products:
            price_str = f"${float(product[2]):.2f}" if product[2] else "N/A"
            listbox.insert(tk.END, f"ID: {product[0]}, Name: {product[1]}, Price: {price_str}, Image URL: {product[3]}")
    else:
        listbox.insert(tk.END, "No products found.")

    # Show the clear button during search
    if search_term:
        clear_button.grid(row=1, column=2, padx=10, pady=5)
    else:
        clear_button.grid_forget()  # Hide if the search bar is empty

# Clear search and refresh the product list
def cancel_search():
    search_entry.delete(0, tk.END)  # Clear the search input
    refresh_products()  # Reload all products
    clear_button.grid_forget()  # Hide the "X" button

if __name__ == "__main__":
    initialize_db()
    setup_gui()

