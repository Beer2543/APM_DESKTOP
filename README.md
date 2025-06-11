import tkinter as tk
from tkinter import ttk, messagebox
import sqlite3
import datetime

# --- 1. การเชื่อมต่อและสร้างฐานข้อมูล ---
def connect_db():
    conn = sqlite3.connect('business_app.db')
    cursor = conn.cursor()
    
    # ตารางพนักงาน
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS employees (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            position TEXT,
            contact TEXT
        )
    ''')
    
    # ตารางยอดขาย Susco
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS sales_susco (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            sale_date DATE NOT NULL,
            total_sales REAL NOT NULL,
            growth_percent REAL
        )
    ''')

    # ตารางยอดขาย Lawson108
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS sales_lawson (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            sale_date DATE NOT NULL,
            total_sales REAL NOT NULL,
            growth_percent REAL
        )
    ''')

    # ตารางตัดกะ (Shift Summary)
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS shift_summary (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            shift_date DATE NOT NULL,
            shift_taker TEXT NOT NULL,
            fuel_sales REAL,
            engine_oil_sales REAL,
            coupon_sales REAL,
            total_sales_summary REAL,
            payment_credit1 REAL,
            payment_credit2 REAL,
            payment_credit3 REAL,
            payment_credit4 REAL,
            payment_credit5 REAL,
            total_credit_card REAL,
            true_shoppe REAL,
            points_redeem REAL,
            member_card REAL,
            credit_account REAL,
            coupon_payment REAL,
            other_payment REAL,
            cash_drop REAL,
            cash_at_close REAL,
            cash_discrepancy REAL
        )
    ''')

    # ตารางงานทำความสะอาด (Tasks)
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS tasks (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            task_name TEXT NOT NULL,
            responsible_person TEXT,
            due_date DATE,
            status TEXT,
            notes TEXT
        )
    ''')

    # ตารางบันทึกค่าไฟ (Electricity Log)
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS electricity_log (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            log_date DATE NOT NULL,
            unit_consumed REAL NOT NULL,
            cost REAL
        )
    ''')

    conn.commit()
    return conn

# --- 2. ฟังก์ชันสำหรับส่วนต่างๆ ของแอป ---

def create_employee_management_tab(notebook):
    frame = ttk.Frame(notebook)
    notebook.add(frame, text='🧑🏻‍💻 ข้อมูลพนักงาน')

    # ตัวอย่าง: การเพิ่มพนักงาน
    lbl_name = tk.Label(frame, text="ชื่อพนักงาน:")
    lbl_name.grid(row=0, column=0, padx=5, pady=5)
    entry_name = tk.Entry(frame)
    entry_name.grid(row=0, column=1, padx=5, pady=5)

    lbl_position = tk.Label(frame, text="ตำแหน่ง:")
    lbl_position.grid(row=1, column=0, padx=5, pady=5)
    entry_position = tk.Entry(frame)
    entry_position.grid(row=1, column=1, padx=5, pady=5)

    def add_employee():
        name = entry_name.get()
        position = entry_position.get()
        if name:
            conn = connect_db()
            cursor = conn.cursor()
            cursor.execute("INSERT INTO employees (name, position) VALUES (?, ?)", (name, position))
            conn.commit()
            conn.close()
            messagebox.showinfo("สำเร็จ", "เพิ่มพนักงานเรียบร้อย!")
            entry_name.delete(0, tk.END)
            entry_position.delete(0, tk.END)
            # โหลดข้อมูลพนักงานใหม่
            load_employees_to_dropdown()
        else:
            messagebox.showerror("ข้อผิดพลาด", "กรุณากรอกชื่อพนักงาน")

    btn_add = tk.Button(frame, text="เพิ่มพนักงาน", command=add_employee)
    btn_add.grid(row=2, column=0, columnspan=2, pady=10)

    # แสดงพนักงานที่มีอยู่ (ใช้ Treeview)
    employee_tree = ttk.Treeview(frame, columns=("ID", "ชื่อ", "ตำแหน่ง"), show="headings")
    employee_tree.heading("ID", text="ID")
    employee_tree.heading("ชื่อ", text="ชื่อ")
    employee_tree.heading("ตำแหน่ง", text="ตำแหน่ง")
    employee_tree.grid(row=3, column=0, columnspan=2, padx=5, pady=5)

    def refresh_employee_list():
        for item in employee_tree.get_children():
            employee_tree.delete(item)
        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("SELECT id, name, position FROM employees")
        for row in cursor.fetchall():
            employee_tree.insert("", "end", values=row)
        conn.close()

    btn_refresh = tk.Button(frame, text="รีเฟรชข้อมูลพนักงาน", command=refresh_employee_list)
    btn_refresh.grid(row=4, column=0, columnspan=2, pady=5)
    refresh_employee_list() # โหลดข้อมูลตอนเปิดแท็บ

    # Dropdown สำหรับเลือกพนักงาน (จะใช้ในส่วนอื่นๆ เช่น ใบตัดกะ)
    global employee_names # ทำให้เป็น global เพื่อให้เข้าถึงจากฟังก์ชันอื่นได้
    employee_names = []
    def load_employees_to_dropdown():
        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("SELECT name FROM employees")
        global employee_names
        employee_names = [row[0] for row in cursor.fetchall()]
        conn.close()
        # อัปเดต dropdown ที่ใช้งาน (ถ้ามี)
        
    load_employees_to_dropdown() # โหลดพนักงานครั้งแรก

def create_sales_dashboard_tab(notebook):
    frame = ttk.Frame(notebook)
    notebook.add(frame, text='📈 Dashboard ยอดขาย')

    # ส่วนนี้จะซับซ้อนขึ้น อาจต้องใช้ไลบรารีเช่น matplotlib สำหรับกราฟ
    # เบื้องต้นจะแสดงข้อมูลดิบ
    tk.Label(frame, text="Dashboard ยอดขาย Susco และ Lawson108").pack(pady=10)
    
    # ตัวอย่าง: แสดงข้อมูลยอดขาย Susco
    susco_tree = ttk.Treeview(frame, columns=("วันที่", "ยอดขาย", "%เติบโต"), show="headings")
    susco_tree.heading("วันที่", text="วันที่")
    susco_tree.heading("ยอดขาย", text="ยอดขาย Susco")
    susco_tree.heading("%เติบโต", text="%เติบโต")
    susco_tree.pack(pady=5)

    def refresh_susco_sales():
        for item in susco_tree.get_children():
            susco_tree.delete(item)
        conn = connect_db()
        cursor = conn.cursor()
        cursor.execute("SELECT sale_date, total_sales, growth_percent FROM sales_susco ORDER BY sale_date DESC LIMIT 10") # แสดง 10 รายการล่าสุด
        for row in cursor.fetchall():
            susco_tree.insert("", "end", values=row)
        conn.close()

    btn_refresh_susco = tk.Button(frame, text="รีเฟรชยอดขาย Susco", command=refresh_susco_sales)
    btn_refresh_susco.pack(pady=5)
    refresh_susco_sales()

    # ส่วนของ Lawson108 ก็จะคล้ายกัน

def create_shift_summary_tab(notebook):
    frame = ttk.Frame(notebook)
    notebook.add(frame, text='📄 ใบตัดกะ')

    # Dropdown สำหรับเลือกคนรับผิดชอบกะ
    tk.Label(frame, text="คนรับผิดชอบกะ:").grid(row=0, column=0, padx=5, pady=5)
    shift_taker_var = tk.StringVar(frame)
    shift_taker_dropdown = ttk.Combobox(frame, textvariable=shift_taker_var, values=employee_names, state="readonly")
    shift_taker_dropdown.grid(row=0, column=1, padx=5, pady=5)
    shift_taker_dropdown.set(employee_names[0] if employee_names else "") # ตั้งค่าเริ่มต้น

    # ฟิลด์สำหรับกรอกข้อมูลต่างๆ
    tk.Label(frame, text="ยอดขายน้ำมัน:").grid(row=1, column=0, padx=5, pady=5)
    entry_fuel_sales = tk.Entry(frame)
    entry_fuel_sales.grid(row=1, column=1, padx=5, pady=5)

    tk.Label(frame, text="ยอดขายรวม:").grid(row=2, column=0, padx=5, pady=5)
    entry_total_sales_summary = tk.Entry(frame)
    entry_total_sales_summary.grid(row=2, column=1, padx=5, pady=5)

    # (เพิ่มฟิลด์อื่นๆ สำหรับการชำระเงิน, เงินดรอป, เงินปิดกะ ตามที่คุณระบุ)

    def calculate_discrepancy():
        try:
            total_sales = float(entry_total_sales_summary.get())
            # สมมติว่ามีช่องสำหรับกรอกยอดชำระเงินทั้งหมดแล้ว
            # ตัวอย่าง: total_payments = float(entry_credit1.get()) + ... + float(entry_cash_drop.get())
            # สำหรับตอนนี้ ให้สมมติว่ามีการกรอกข้อมูลครบถ้วน
            
            # **นี่คือตัวอย่างการคำนวณแบบง่ายๆ คุณต้องเพิ่มช่องกรอกข้อมูลทั้งหมดและรวมยอดก่อน**
            # เพื่อให้เห็นภาพ
            total_payments_example = 0
            if entry_fuel_sales.get():
                total_payments_example += float(entry_fuel_sales.get())
            
            discrepancy = total_sales - total_payments_example # นี่เป็นตัวอย่างที่ไม่สมบูรณ์
            messagebox.showinfo("สรุปเงินขาดหาย", f"เงินขาด/เกิน: {discrepancy:.2f}")

        except ValueError:
            messagebox.showerror("ข้อผิดพลาด", "กรุณากรอกข้อมูลตัวเลขให้ถูกต้อง")
        except Exception as e:
            messagebox.showerror("ข้อผิดพลาด", f"เกิดข้อผิดพลาด: {e}")

    btn_calculate = tk.Button(frame, text="คำนวณเงินขาดหาย", command=calculate_discrepancy)
    btn_calculate.grid(row=3, column=0, columnspan=2, pady=10)

    def save_shift_summary():
        # ต้องดึงข้อมูลจากทุกช่องกรอก
        try:
            shift_taker = shift_taker_var.get()
            shift_date = datetime.date.today().isoformat()
            fuel_sales = float(entry_fuel_sales.get()) if entry_fuel_sales.get() else 0.0
            total_sales = float(entry_total_sales_summary.get()) if entry_total_sales_summary.get() else 0.0
            
            # ต้องเพิ่มตัวแปรสำหรับ payment_credit1, payment_credit2, etc. และรวมยอด
            # ตัวอย่างสำหรับตอนนี้
            conn = connect_db()
            cursor = conn.cursor()
            cursor.execute('''
                INSERT INTO shift_summary (
                    shift_date, shift_taker, fuel_sales, total_sales_summary,
                    payment_credit1, payment_credit2, payment_credit3, payment_credit4, payment_credit5,
                    total_credit_card, true_shoppe, points_redeem, member_card, credit_account,
                    coupon_payment, other_payment, cash_drop, cash_at_close, cash_discrepancy
                ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
            ''', (shift_date, shift_taker, fuel_sales, total_sales,
                  0, 0, 0, 0, 0, # แทนด้วยค่าจากช่องกรอกจริง
                  0, 0, 0, 0, 0, 0, 0, 0, 0, 0)) # แทนด้วยค่าจากช่องกรอกจริงและคำนวณ discrepancy
            conn.commit()
            conn.close()
            messagebox.showinfo("สำเร็จ", "บันทึกใบตัดกะเรียบร้อย!")
            # ล้างข้อมูลในฟอร์ม
        except ValueError:
            messagebox.showerror("ข้อผิดพลาด", "กรุณากรอกข้อมูลตัวเลขให้ถูกต้อง")
        except Exception as e:
            messagebox.showerror("ข้อผิดพลาด", f"เกิดข้อผิดพลาดในการบันทึก: {e}")

    btn_save_shift = tk.Button(frame, text="บันทึกใบตัดกะ", command=save_shift_summary)
    btn_save_shift.grid(row=4, column=0, columnspan=2, pady=10)

# (สร้างฟังก์ชันสำหรับแท็บอื่นๆ ที่เหลือในลักษณะเดียวกัน)
# เช่น create_susco_inventory_tab, create_lawson_inventory_tab, 
# create_cleaning_tasks_tab, create_electricity_log_tab, 
# create_manager_report_tab, create_links_tab, create_alerts_tab, create_notes_tab

def create_main_app():
    root = tk.Tk()
    root.title("Business Management App")
    root.geometry("800x600")

    # สร้าง Notebook (ระบบแท็บ)
    notebook = ttk.Notebook(root)
    notebook.pack(expand=True, fill="both", padx=10, pady=10)

    # สร้างแท็บต่างๆ
    create_employee_management_tab(notebook)
    create_sales_dashboard_tab(notebook)
    create_shift_summary_tab(notebook)
    # (เพิ่มการเรียกฟังก์ชันสร้างแท็บอื่นๆ ที่คุณต้องการ)

    root.mainloop()

# --- เริ่มต้นการทำงานของแอป ---
if __name__ == "__main__":
    connect_db() # ตรวจสอบ/สร้างฐานข้อมูลตอนเริ่มต้น
    create_main_app()
