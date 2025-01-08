# Python-project
import tkinter as tk
from tkinter import messagebox, scrolledtext
from tkinter import font
from tkinter import ttk
from PIL import Image, ImageTk
import mysql.connector

class RestaurantManagementSystem:
    def __init__(self, root):
        self.root = root
        self.root.title("Restaurant Management System")
        self.root.geometry("800x600")

        self.customer_name = tk.StringVar()
        self.customer_contact = tk.StringVar()

        self.items = {
            "Drinks": {
                "Coke": 50,
                "Lemonade": 40,
                "Water": 20,
                "Juice": 60,
                "Tea": 30
            },
            "Mocktails": {
                "Virgin Mojito": 120,
                "Shirley Temple": 110,
                "Pina Colada": 140,
                "Fruit Punch": 130,
                "Blue Lagoon": 150
            },
            "Bakery Items": {
                "Croissant": 70,
                "Brownie": 90,
                "Muffin": 60,
                "Cake Slice": 80,
                "Cookie": 50
            },
            "Biryanis": {
                "Chicken Biryani": 250,
                "Veg Biryani": 200,
                "Mutton Biryani": 300,
                "Egg Biryani": 220,
                "Paneer Biryani": 240,
                "Prawn Biryani": 320,
                "Fish Biryani": 310,
                "Hyderabadi Biryani": 350
            },
            "Veg Starters": {
                "Veg Pakora": 80,
                "Paneer Tikka": 120,
                "Spring Rolls": 100,
                "Veg Samosa": 60,
                "Chili Paneer": 140
            }
        }

        self.orders = {}
        self.gst_percentage = 7

        self.create_gui()
        self.connect_to_db()

    def connect_to_db(self):
        try:
            self.connection = mysql.connector.connect(
                host="localhost",        # Change as needed
                user="root",    # Change to your MySQL username
                password="SRI@2006.ssslv", # Change to your MySQL password
                database="restaurant_management"
            )
            if self.connection.is_connected():
                print("Successfully connected to the database")
        except mysql.connector.Error as err:
            print(f"Error: {err}")
            messagebox.showerror("Database Connection Error", f"Error: {err}")

    def create_gui(self):
        self.notebook = ttk.Notebook(self.root)
        self.notebook.pack(fill='both', expand=True)

        self.create_welcome_page()
        self.create_customer_details_page()
        self.create_menu_page()
        self.create_printed_bill_page()

    def create_welcome_page(self):
        welcome_frame = tk.Frame(self.notebook, bg="#f0f0f0")
        self.notebook.add(welcome_frame, text="Welcome")

        try:
            self.image_path = "C:/Users/hp/Desktop/download1.jpeg"  # Update the path to your image
            image = Image.open(self.image_path)
            self.welcome_image = ImageTk.PhotoImage(image)
            image_label = tk.Label(welcome_frame, image=self.welcome_image, bg="#f0f0f0")
            image_label.pack(pady=7)
        except Exception as e:
            print("Error loading image:", e)

        shop_font = font.Font(family="Arial", size=24, weight="bold")
        shop_name_label = tk.Label(welcome_frame, text="EATING HOUSE", font=shop_font, bg="#f0f0f0", fg="#333333")
        shop_name_label.pack(pady=100)
        shop_name_label.place(relx=0.5, rely=0.1, anchor='center')

        start_button = tk.Button(welcome_frame, text="Start", font=10, command=self.go_to_customer_details_page, bg="#4CAF50", fg="#ffffff")
        start_button.pack(pady=10)
        start_button.place(relx=0.5, rely=0.5, anchor='center')

    def create_customer_details_page(self):
        self.customer_details_frame = tk.Frame(self.notebook, bg="#C2185B")
        self.notebook.add(self.customer_details_frame, text="Customer Details")

        details_frame = tk.LabelFrame(self.customer_details_frame, text="Customer Details", bg="#D81B60", fg="#ffffff")
        details_frame.place(relx=0.5, rely=0.5, anchor='center', width=400, height=300)

        shop_font = font.Font(family="Arial", size=24, weight="bold")
        shop_name_label = tk.Label(details_frame, text="EATING HOUSE", font=shop_font, bg="#D81B60", fg="#ffffff")
        shop_name_label.grid(row=0, column=0, columnspan=2, pady=(10, 20))

        name_label = tk.Label(details_frame, text="Name:", bg="#D81B60", fg="#ffffff", font=("Arial", 16))
        name_label.grid(row=1, column=0, padx=5, pady=5, sticky="e")
        name_entry = tk.Entry(details_frame, textvariable=self.customer_name, bg="#ffffff", fg="#000000", font=("Arial", 16))
        name_entry.grid(row=1, column=1, padx=5, pady=5, sticky="w")

        contact_label = tk.Label(details_frame, text="Contact:", bg="#D81B60", fg="#ffffff", font=("Arial", 16))
        contact_label.grid(row=2, column=0, padx=5, pady=5, sticky="e")
        contact_entry = tk.Entry(details_frame, textvariable=self.customer_contact, bg="#ffffff", fg="#000000", font=("Arial", 16))
        contact_entry.grid(row=2, column=1, padx=5, pady=5, sticky="w")
        contact_entry.configure(validate="key")
        contact_entry.configure(validatecommand=(contact_entry.register(self.validate_contact), "%P"))

        next_button = tk.Button(details_frame, text="Next", command=self.go_to_menu_page, bg="#4CAF50", fg="#ffffff", font=("Arial", 16))
        next_button.grid(row=3, column=0, columnspan=2, pady=10)

    def create_menu_page(self):
        self.menu_frame = tk.Frame(self.notebook, bg="#f0f0f0")
        self.notebook.add(self.menu_frame, text="Menu")

        for category, items in self.items.items():
            category_frame = tk.LabelFrame(self.menu_frame, text=category, bg="#4CAF50", fg="#ffffff")
            category_frame.pack(fill="both", expand=True, padx=10, pady=10)

            item_header = tk.Label(category_frame, text="Items", bg="#4CAF50", fg="#ffffff")
            item_header.grid(row=0, column=0, padx=5, pady=5, sticky="w")
            quantity_header = tk.Label(category_frame, text="Quantity", bg="#4CAF50", fg="#ffffff")
            quantity_header.grid(row=0, column=1, padx=5, pady=5, sticky="w")

            row = 1
            for item, price in items.items():
                quantity_entry = tk.Entry(category_frame, width=5, bg="#ffffff", fg="#000000")
                quantity_entry.grid(row=row, column=1, padx=5, pady=5, sticky="w")

                self.orders[item] = {"quantity": quantity_entry, "price": price}

                quantity_entry.bind("<FocusOut>", lambda event: self.update_sample_bill())
                quantity_entry.bind("<Return>", lambda event: self.update_sample_bill())

                item_label = tk.Label(category_frame, text=f"{item} - {self.convert_to_inr(price)}", bg="#4CAF50", fg="#ffffff")
                item_label.grid(row=row, column=0, padx=5, pady=5, sticky="w")

                row += 1

        buttons_frame = tk.Frame(self.menu_frame, bg="#e0e0e0")
        buttons_frame.pack(fill="x", padx=10, pady=10)

        clear_selection_button = tk.Button(buttons_frame, text="Clear Selection", command=self.clear_selection, bg="#F44336", fg="#ffffff")
        clear_selection_button.pack(side="left", padx=5)

        self.sample_bill_text = tk.Text(self.menu_frame, height=10, bg="#ffffff", fg="#000000")
        self.sample_bill_text.pack(fill="x", padx=10, pady=10)

        self.update_sample_bill()

        print_bill_button = tk.Button(self.menu_frame, text="Print Bill", command=self.go_to_printed_bill_page, bg="#4CAF50", fg="#ffffff")
        print_bill_button.place(relx=0.95, rely=0.95, anchor='se')

    def create_printed_bill_page(self):
        self.printed_bill_frame = tk.Frame(self.notebook, bg="#d5006d")  # Magenta background
        self.notebook.add(self.printed_bill_frame, text="Printed Bill")

        self.printed_bill_text = scrolledtext.ScrolledText(self.printed_bill_frame, wrap=tk.WORD, height=30, bg="#ffffff", fg="#000000")
        self.printed_bill_text.pack(fill="both", expand=True, padx=10, pady=10)

        show_saved_orders_button = tk.Button(self.printed_bill_frame, text="Show Saved Orders", command=self.show_saved_orders, bg="#4CAF50", fg="#ffffff")
        show_saved_orders_button.pack(pady=10)

        close_button = tk.Button(self.printed_bill_frame, text="Close", command=self.close_printed_bill, bg="#F44336", fg="#ffffff")
        close_button.pack(pady=10)

        back_button = tk.Button(self.printed_bill_frame, text="Back to Welcome", command=self.go_to_welcome_page, bg="#4CAF50", fg="#ffffff")
        back_button.pack(pady=10)

    def update_sample_bill(self):
        self.sample_bill_text.delete(1.0, tk.END)
        total_amount = 0

        for item, details in self.orders.items():
            quantity = details["quantity"].get()
            if quantity.isdigit() and int(quantity) > 0:
                price = details["price"]
                item_total = price * int(quantity)
                total_amount += item_total
                self.sample_bill_text.insert(tk.END, f"{item}: {quantity} x {self.convert_to_inr(price)} = {self.convert_to_inr(item_total)}\n")

        gst = total_amount * (self.gst_percentage / 100)
        total_with_gst = total_amount + gst
        self.sample_bill_text.insert(tk.END, f"\nSubtotal: {self.convert_to_inr(total_amount)}\n")
        self.sample_bill_text.insert(tk.END, f"GST ({self.gst_percentage}%): {self.convert_to_inr(gst)}\n")
        self.sample_bill_text.insert(tk.END, f"Total: {self.convert_to_inr(total_with_gst)}\n")

    def go_to_customer_details_page(self):
        self.notebook.select(self.notebook.tabs()[1])

    def go_to_menu_page(self):
        self.notebook.select(self.notebook.tabs()[2])

    def go_to_printed_bill_page(self):
        self.print_bill()
        self.save_order_to_db()  # Save order to database
        self.notebook.select(self.notebook.tabs()[3])

    def go_to_welcome_page(self):
        self.notebook.select(self.notebook.tabs()[0])  # Go back to the welcome page

    def print_bill(self):
        self.printed_bill_text.delete(1.0, tk.END)
        self.printed_bill_text.insert(tk.END, f"Customer Name: {self.customer_name.get()}\n")
        self.printed_bill_text.insert(tk.END, f"Customer Contact: {self.customer_contact.get()}\n")
        self.printed_bill_text.insert(tk.END, "-" * 30 + "\n")

        total_amount = 0
        for item, details in self.orders.items():
            quantity = details["quantity"].get()
            if quantity.isdigit() and int(quantity) > 0:
                price = details["price"]
                item_total = price * int(quantity)
                total_amount += item_total
                self.printed_bill_text.insert(tk.END, f"{item}: {quantity} x {self.convert_to_inr(price)} = {self.convert_to_inr(item_total)}\n")

        gst = total_amount * (self.gst_percentage / 100)
        total_with_gst = total_amount + gst
        self.printed_bill_text.insert(tk.END, f"\nSubtotal: {self.convert_to_inr(total_amount)}\n")
        self.printed_bill_text.insert(tk.END, f"GST ({self.gst_percentage}%): {self.convert_to_inr(gst)}\n")
        self.printed_bill_text.insert(tk.END, f"Total: {self.convert_to_inr(total_with_gst)}\n")

    def save_order_to_db(self):
        try:
            cursor = self.connection.cursor()
            cursor.execute("""
                CREATE TABLE IF NOT EXISTS past_orders (
                    id INT AUTO_INCREMENT PRIMARY KEY,
                    customer_name VARCHAR(255),
                    customer_contact VARCHAR(255),
                    item VARCHAR(255),
                    quantity INT,
                    total DECIMAL(10, 2)
                )
            """)

            for item, details in self.orders.items():
                quantity = details["quantity"].get()
                if quantity.isdigit() and int(quantity) > 0:
                    price = details["price"]
                    item_total = price * int(quantity)

                    cursor.execute("""
                        INSERT INTO past_orders (customer_name, customer_contact, item, quantity, total)
                        VALUES (%s, %s, %s, %s, %s)
                    """, (self.customer_name.get(), self.customer_contact.get(), item, quantity, item_total))

            self.connection.commit()
            print("Order saved to database")
        except mysql.connector.Error as err:
            print(f"Error saving order: {err}")

    def show_saved_orders(self):
        saved_orders = self.fetch_past_orders()
        self.printed_bill_text.delete(1.0, tk.END)  # Clear existing text

        if saved_orders:
            self.printed_bill_text.insert(tk.END, "Saved Orders:\n")
            for order in saved_orders:
                self.printed_bill_text.insert(tk.END, f"Name: {order[1]}, Contact: {order[2]}, Item: {order[3]}, Quantity: {order[4]}, Total: {self.convert_to_inr(order[5])}\n")
        else:
            self.printed_bill_text.insert(tk.END, "No saved orders found.\n")

    def fetch_past_orders(self):
        try:
            cursor = self.connection.cursor()
            cursor.execute("SELECT * FROM past_orders")
            return cursor.fetchall()
        except mysql.connector.Error as err:
            print(f"Error fetching orders: {err}")
            return []

    def close_printed_bill(self):
        self.notebook.select(self.notebook.tabs()[2])

    def clear_selection(self):
        for item, details in self.orders.items():
            details["quantity"].delete(0, tk.END)
        self.update_sample_bill()

    def validate_contact(self, contact):
        return contact.isdigit() and len(contact) <= 10

    def convert_to_inr(self, amount):
        return f"₹ {amount:.2f}"

if __name__ == "__main__":
    root = tk.Tk()
    app = RestaurantManagementSystem(root)
    root.mainloop()
