# gerbeagn4
import tkinter as tk
from tkinter import ttk, messagebox
import json
import os
from datetime import datetime

# Путь к файлу данных
DATA_FILE = "expenses.json"

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.root.geometry("800x600")

        # Загрузка данных
        self.expenses = self.load_data()

        self.setup_ui()

    def setup_ui(self):
        # Фрейм для ввода данных
        input_frame = ttk.Frame(self.root)
        input_frame.pack(pady=10, fill="x")

        # Поле суммы
        ttk.Label(input_frame, text="Сумма:").grid(row=0, column=0, padx=5)
        self.amount_entry = ttk.Entry(input_frame, width=15)
        self.amount_entry.grid(row=0, column=1, padx=5)

        # Поле категории
        ttk.Label(input_frame, text="Категория:").grid(row=0, column=2, padx=5)
        self.category_var = tk.StringVar()
        self.category_combo = ttk.Combobox(
            input_frame,
            textvariable=self.category_var,
            values=["Еда", "Транспорт", "Развлечения", "Жильё", "Другое"],
            width=15
        )
        self.category_combo.grid(row=0, column=3, padx=5)

        # Поле даты
        ttk.Label(input_frame, text="Дата (ГГГГ-ММ-ДД):").grid(row=0, column=4, padx=5)
        self.date_entry = ttk.Entry(input_frame, width=15)
        self.date_entry.grid(row=0, column=5, padx=5)

        # Кнопка добавления
        add_button = ttk.Button(input_frame, text="Добавить расход", command=self.add_expense)
        add_button.grid(row=0, column=6, padx=5)

        # Таблица расходов
        columns = ("ID", "Сумма", "Категория", "Дата")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings", height=15)
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=100)
        self.tree.pack(pady=10, padx=10, fill="both", expand=True)

        # Фрейм фильтрации
        filter_frame = ttk.Frame(self.root)
        filter_frame.pack(pady=5, fill="x")

        # Фильтрация по категории
        ttk.Label(filter_frame, text="Фильтр по категории:").pack(side="left", padx=5)
        self.filter_category_var = tk.StringVar()
        filter_combo = ttk.Combobox(
            filter_frame,
            textvariable=self.filter_category_var,
            values=["Все"] + ["Еда", "Транспорт", "Развлечения", "Жильё", "Другое"]
        )
        filter_combo.set("Все")
        filter_combo.pack(side="left", padx=5)

        # Фильтрация по дате
        ttk.Label(filter_frame, text="С:").pack(side="left", padx=5)
        self.start_date_entry = ttk.Entry(filter_frame, width=12)
        self.start_date_entry.pack(side="left", padx=5)
        ttk.Label(filter_frame, text="По:").pack(side="left", padx=5)
        self.end_date_entry = ttk.Entry(filter_frame, width=12)
        self.end_date_entry.pack(side="left", padx=5)

        # Кнопка фильтрации
        filter_button = ttk.Button(filter_frame, text="Применить фильтр", command=self.apply_filter)
        filter_button.pack(side="left", padx=5)

        # Кнопка сброса фильтров
        reset_button = ttk.Button(filter_frame, text="Сбросить фильтры", command=self.reset_filter)
        reset_button.pack(side="left", padx=5)

        # Подсчёт суммы за период
        total_frame = ttk.Frame(self.root)
        total_frame.pack(pady=5, fill="x")
        ttk.Label(total_frame, text="Общая сумма за период:").pack(side="left", padx=5)
        self.total_label = ttk.Label(total_frame, text="0.00")
        self.total_label.pack(side="left", padx=5)

        self.update_table()

    def validate_input(self, amount_str, date_str):
        """Проверка корректности ввода"""
        try:
            amount = float(amount_str)
            if amount <= 0:
                raise ValueError("Сумма должна быть положительным числом")
        except ValueError:
            messagebox.showerror("Ошибка", "Некорректная сумма!")
            return False

        try:
            datetime.strptime(date_str, "%Y-%m-%d")
        except ValueError:
            messagebox.showerror("Ошибка", "Неверный формат даты! Используйте ГГГГ-ММ-ДД")
            return False
        return True

    def add_expense(self):
        """Добавление нового расхода"""
        amount_str = self.amount_entry.get().strip()
        category = self.category_var.get().strip()
        date_str = self.date_entry.get().strip()

        if not all([amount_str, category, date_str]):
            messagebox.showerror("Ошибка", "Все поля должны быть заполнены!")
            return

        if not self.validate_input(amount_str, date_str):
            return

        expense = {
            "id": len(self.expenses) + 1,
            "amount": float(amount_str),
            "category": category,
            "date": date_str
        }
        self.expenses.append(expense)
        self.save_data()
        self.update_table()

        # Очистка полей ввода
        self.amount_entry.delete(0, tk.END)
        self.date_entry.delete(0, tk.END)

    def load_data(self):
        """Загрузка данных из JSON"""
        if os.path.exists(DATA_FILE):
            with open(DATA_FILE, "r", encoding="utf-8") as f:
                return json.load(f)
        return []

    def save_data(self):
        """Сохранение данных в JSON"""
        with open(DATA_FILE, "w", encoding="utf-8") as f:
            json.dump(self.expenses, f, ensure_ascii=False, indent=4)

    def update_table(self, filtered_expenses=None):
        """Обновление таблицы"""
        self.tree.delete(*self.tree.get_children())
        expenses_to_show = filtered_expenses if filtered_expenses is not None else self.expenses


        for expense in expenses_to_show:
            self.tree.insert("", "end", values=(
                expense["id"],
                f"{expense['
