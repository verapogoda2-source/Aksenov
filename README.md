import tkinter as tk
from tkinter import ttk, messagebox
import random
import string
import json
import os

class PasswordGeneratorApp:
    def __init__(self, root):
        self.root = root
        self.root.title("Random Password Generator")
        self.root.geometry("600x500")

        # Переменные
        self.password_length = tk.IntVar(value=12)
        self.use_digits = tk.BooleanVar(value=True)
        self.use_letters = tk.BooleanVar(value=True)
        self.use_special = tk.BooleanVar(value=False)
        self.history = []

        self.setup_ui()
        self.load_history()

    def setup_ui(self):
        # Ползунок длины пароля
        ttk.Label(self.root, text="Длина пароля:").pack(pady=5)
        length_scale = ttk.Scale(
            self.root,
            from_=4,
            to=64,
            orient="horizontal",
            variable=self.password_length
        )
        length_scale.pack(fill="x", padx=20)

        length_label = ttk.Label(
            self.root,
            textvariable=self.password_length
        )
        length_label.pack()

        # Чекбоксы для выбора символов
        ttk.Checkbutton(
            self.root,
            text="Цифры (0-9)",
            variable=self.use_digits
        ).pack(anchor="w", padx=20)
        ttk.Checkbutton(
            self.root,
            text="Буквы (A-Z, a-z)",
            variable=self.use_letters
        ).pack(anchor="w", padx=20)
        ttk.Checkbutton(
            self.root,
            text="Спецсимволы (!@#$% и т.д.)",
            variable=self.use_special
        ).pack(anchor="w", padx=20)

        # Кнопка генерации
        generate_btn = ttk.Button(
            self.root,
            text="Сгенерировать пароль",
            command=self.generate_password
        )
        generate_btn.pack(pady=10)

        # Поле отображения пароля
        self.password_var = tk.StringVar()
        password_entry = ttk.Entry(
            self.root,
            textvariable=self.password_var,
            state="readonly",
            font=("Courier", 12)
        )
        password_entry.pack(fill="x", padx=20, pady=5)

        # Таблица истории
        columns = ("ID", "Пароль", "Длина", "Символы")
        self.tree = ttk.Treeview(self.root, columns=columns, show="headings")
        for col in columns:
            self.tree.heading(col, text=col)
            self.tree.column(col, width=100)
        self.tree.pack(fill="both", expand=True, padx=20, pady=10)

    def generate_password(self):
        length = self.password_length.get()

        # Проверка длины
        if length < 4:
            messagebox.showerror("Ошибка", "Минимальная длина пароля — 4 символа")
            return
        if length > 64:
            messagebox.showerror("Ошибка", "Максимальная длина пароля — 64 символа")
            return

        # Формирование набора символов
        chars = ""
        if self.use_digits.get():
            chars += string.digits
        if self.use_letters.get():
            chars += string.ascii_letters
        if self.use_special.get():
            chars += "!@#$%^&*()_+-=[]{}|;:,.<>?"

        if not chars:
            messagebox.showerror("Ошибка", "Выберите хотя бы один тип символов")
            return

        # Генерация пароля
        password = ''.join(random.choice(chars) for _ in range(length))
        self.password_var.set(password)

        # Добавление в историю
        symbols = ""
        if self.use_digits.get(): symbols += "Цифры "
        if self.use_letters.get(): symbols += "Буквы "
        if self.use_special.get(): symbols += "Спецсимволы"

        entry = {
            "id": len(self.history) + 1,
            "password": password,
            "length": length,
            "symbols": symbols.strip()
        }
        self.history.append(entry)
        self.update_history_table()
        self.save_history()

    def save_history(self):
        with open("password_history.json", "w", encoding="utf-8") as f:
            json.dump(self.history, f, indent=2, ensure_ascii=False)

    def load_history(self):
        if os.path.exists("password_history.json"):
            with open("password_history.json", "r", encoding="utf-8") as f:
                self.history = json.load(f)
            self.update_history_table()

    def update_history_table(self):
        # Очистка таблицы
        for item in self.tree.get_children():
            self.tree.delete(item)
        # Заполнение данными
        for entry in self.history:
            self.tree.insert("", "end", values=(
                entry["id"],
                entry["password"],
                entry["length"],
                entry["symbols"]
            ))

if __name__ == "__main__":
    root = tk.Tk()
    app = PasswordGeneratorApp(root)
    root.mainloop()

