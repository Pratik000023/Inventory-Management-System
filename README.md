# Inventory-Management-System
import tkinter as tk
from tkinter import messagebox, simpledialog, filedialog
from tkinter import ttk
from reportlab.lib.pagesizes import A4
from reportlab.pdfgen import canvas

class InventoryItem:
    def __init__(self, item_id, name, quantity, price_rupees):
        self.item_id = item_id
        self.name = name
        self.quantity = quantity
        self.price_rupees = price_rupees

    def to_tuple(self):
        return (self.item_id, self.name, self.quantity, f"₹{self.price_rupees:.2f}")

class InventoryApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Inventory Management System")
        self.items = {}
        self.create_widgets()

    def create_widgets(self):
        frame = tk.Frame(self.root)
        frame.pack(padx=10, pady=10)

        tk.Button(frame, text="Add Item", width=15, command=self.add_item).grid(row=0, column=0)
        tk.Button(frame, text="Remove Item", width=15, command=self.remove_item).grid(row=0, column=1)
        tk.Button(frame, text="Update Quantity", width=15, command=self.update_quantity).grid(row=0, column=2)
        tk.Button(frame, text="Search Item", width=15, command=self.search_item).grid(row=0, column=3)
        tk.Button(frame, text="View All", width=15, command=self.view_all_items).grid(row=0, column=4)
        tk.Button(frame, text="Download as PDF", width=15, command=self.download_pdf).grid(row=0, column=5)

        self.tree = ttk.Treeview(self.root, columns=("ID", "Name", "Quantity", "Price"), show="headings")
        self.tree.heading("ID", text="ID")
        self.tree.heading("Name", text="Name")
        self.tree.heading("Quantity", text="Quantity")
        self.tree.heading("Price", text="Price")
        self.tree.pack(padx=10, pady=10, fill=tk.BOTH, expand=True)

    def add_item(self):
        item_id = simpledialog.askstring("Input", "Enter Item ID:")
        if not item_id or item_id in self.items:
            messagebox.showerror("Error", "Invalid or duplicate ID.")
            return

        name = simpledialog.askstring("Input", "Enter Item Name:")
        try:
            quantity = int(simpledialog.askstring("Input", "Enter Quantity:"))
            price_rupees = float(simpledialog.askstring("Input", "Enter Price (in rupees):"))
        except Exception:
            messagebox.showerror("Error", "Invalid input. Please enter numeric values for quantity and price.")
            return

        self.items[item_id] = InventoryItem(item_id, name, quantity, price_rupees)
        messagebox.showinfo("Success", "Item added!")
        self.view_all_items()

    def remove_item(self):
        item_id = simpledialog.askstring("Input", "Enter Item ID to remove:")
        if item_id in self.items:
            del self.items[item_id]
            messagebox.showinfo("Success", "Item removed.")
            self.view_all_items()
        else:
            messagebox.showerror("Error", "Item not found.")

    def update_quantity(self):
        item_id = simpledialog.askstring("Input", "Enter Item ID to update:")
        if item_id in self.items:
            try:
                quantity = int(simpledialog.askstring("Input", "Enter new quantity:"))
                self.items[item_id].quantity = quantity
                messagebox.showinfo("Success", "Quantity updated.")
                self.view_all_items()
            except Exception:
                messagebox.showerror("Error", "Invalid quantity.")
        else:
            messagebox.showerror("Error", "Item not found.")

    def search_item(self):
        name_query = simpledialog.askstring("Search", "Enter name to search:")
        results = [item for item in self.items.values() if name_query.lower() in item.name.lower()]
        self.display_items(results)

    def view_all_items(self):
        self.display_items(self.items.values())

    def display_items(self, item_list):
        for row in self.tree.get_children():
            self.tree.delete(row)
        for item in item_list:
            self.tree.insert("", "end", values=item.to_tuple())

    def download_pdf(self):
        if not self.items:
            messagebox.showwarning("Warning", "No items to export.")
            return

        file_path = filedialog.asksaveasfilename(defaultextension=".pdf", filetypes=[("PDF files", "*.pdf")])
        if not file_path:
            return

        c = canvas.Canvas(file_path, pagesize=A4)
        width, height = A4

        c.setFont("Helvetica-Bold", 16)
        c.drawString(200, height - 50, "Inventory Report")

        c.setFont("Helvetica-Bold", 12)
        c.drawString(50, height - 100, "ID")
        c.drawString(120, height - 100, "Name")
        c.drawString(300, height - 100, "Quantity")
        c.drawString(400, height - 100, "Price (₹)")

        c.setFont("Helvetica", 12)
        y = height - 130
        for item in self.items.values():
            c.drawString(50, y, str(item.item_id))
            c.drawString(120, y, item.name)
            c.drawString(300, y, str(item.quantity))
            c.drawString(400, y, f"₹{item.price_rupees:.2f}")
            y -= 20
            if y < 50:
                c.showPage()
                y = height - 50

        c.save()
        messagebox.showinfo("Success", "Inventory PDF downloaded successfully!")

if __name__ == "__main__":
    root = tk.Tk()
    app = InventoryApp(root)
    root.mainloop()
