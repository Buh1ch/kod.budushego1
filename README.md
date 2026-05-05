# kod.budushego1
__pycache__/
*.pyc
.vscode/
.idea/
venv/
env/
*.json
git add .gitignore
git commit -m "Initial commit: add .gitignore"
pip install requests tkinter
import tkinter as tk
from tkinter import messagebox, ttk
import requests
import json
import os

# Конфигурация
FAVORITES_FILE = "favorites.json"
GITHUB_API_URL = "https://api.github.com/users/"

class GitHubUserFinder:
    def __init__(self, root):
        self.root = root
        self.root.title("GitHub User Finder")
        self.root.geometry("800x600")

        # Загрузка избранных пользователей
        self.favorites = self.load_favorites()

        # Создание интерфейса
        self.create_widgets()

    def create_widgets(self):
        # Поле ввода
        input_frame = tk.Frame(self.root)
        input_frame.pack(pady=10)

        tk.Label(input_frame, text="GitHub username:").pack(side=tk.LEFT)
        self.search_entry = tk.Entry(input_frame, width=40)
        self.search_entry.pack(side=tk.LEFT, padx=5)
        tk.Button(input_frame, text="Search", command=self.search_user).pack(side=tk.LEFT)

        # Результаты поиска
        results_frame = tk.Frame(self.root)
        results_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

        self.results_tree = ttk.Treeview(results_frame, columns=("Username", "Name", "Location", "Public Repos"), show="headings")
        self.results_tree.heading("Username", text="Username")
        self.results_tree.heading("Name", text="Name")
        self.results_tree.heading("Location", text="Location")
        self.results_tree.heading("Public Repos", text="Public Repos")
        self.results_tree.pack(fill=tk.BOTH, expand=True)

        # Кнопки управления
        button_frame = tk.Frame(self.root)
        button_frame.pack(pady=5)

        tk.Button(button_frame, text="Add to Favorites", command=self.add_to_favorites).pack(side=tk.LEFT, padx=5)
        tk.Button(button_frame, text="Show Favorites", command=self.show_favorites).pack(side=tk.LEFT, padx=5)

        # Список избранных
        favorites_frame = tk.LabelFrame(self.root, text="Favorites")
        favorites_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

        self.favorites_listbox = tk.Listbox(favorites_frame)
        self.favorites_listbox.pack(fill=tk.BOTH, expand=True)
        self.update_favorites_display()

    def search_user(self):
        username = self.search_entry.get().strip()

        if not username:
            messagebox.showerror("Error", "Search field cannot be empty!")
            return

        try:
            response = requests.get(f"{GITHUB_API_URL}{username}")

            if response.status_code == 200:
                user_data = response.json()
                self.display_user(user_data)
            else:
                messagebox.showerror("Error", f"User not found: {username}")
        except Exception as e:
            messagebox.showerror("Error", f"Connection error: {str(e)}")

    def display_user(self, user_data):
        # Очистка предыдущих результатов
        for item in self.results_tree.get_children():
            self.results_tree.delete(item)

        # Добавление данных пользователя
        self.results_tree.insert("", "end", values=(
            user_data.get("login", "N/A"),
            user_data.get("name", "N/A"),
            user_data.get("location", "N/A"),
            user_data.get("public_repos", "N/A")
        ))

    def add_to_favorites(self):
        selection = self.results_tree.selection()

        if not selection:
            messagebox.showwarning("Warning", "Select a user to add to favorites!")
            return

        user_data = self.results_tree.item(selection[0])["values"]
        username = user_data[0]

        if username not in self.favorites:
            self.favorites.append(username)
            self.save_favorites()
            self.update_favorites_display()
            messagebox.showinfo("Success", f"{username} added to favorites!")
        else:
            messagebox.showinfo("Info", f"{username} is already in favorites!")

    def show_favorites(self):
        if not self.favorites:
            messagebox.showinfo("Info", "No favorites yet!")
            return

        # Очистка результатов и показ избранных
        for item in self.results_tree.get_children():
            self.results_tree.delete(item)

        for username in self.favorites:
            try:
                response = requests.get(f"{GITHUB_API_URL}{username}")
                if response.status_code == 200:
                    user_data = response.json()
                    self.results_tree.insert("", "end", values=(
                        user_data.get("login", "N/A"),
                        user_data.get("name", "N/A"),
                        user_data.get("location", "N/A"),
                        user_data.get("public_repos", "N/A")
                    ))
            except:
                continue

    def load_favorites(self):
        if os.path.exists(FAVORITES_FILE):
            try:
                with open(FAVORITES_FILE, "r") as f:
                    return json.load(f)
            except:
                return []
        return []

    def save_favorites(self):
        with open(FAVORITES_FILE, "w") as f:
            json.dump(self.favorites, f)

    def update_favorites_display(self):
        self.favorites_listbox.delete(0, tk.END)
        for username in self.favorites:
            self.favorites_listbox.insert(tk.END, username)

if __name__ == "__main__":
    root = tk.Tk()
    app = GitHubUserFinder(root)
    root.mainloop()
# GitHub User Finder

**Автор:** Кривда Даниил

## Описание

Приложение для поиска пользователей GitHub с возможностью сохранения избранных пользователей.

## Функциональность

- Поиск пользователей GitHub по имени
- Отображение информации о пользователе (имя, местоположение, количество публичных репозиториев)
- Добавление пользователей в избранное
- Сохранение избранных пользователей в JSON‑файл
- Просмотр списка избранных пользователей

## Использование API GitHub

Приложение использует GitHub API v3 для получения информации о пользователях. Базовый URL: `https://api.github.com/users/`.

Для поиска пользователя отправляется GET‑запрос: `GET https://api.github.com/users/{username}`

## Установка и запуск

1. Клонируйте репозиторий:
```bash
git clone [URL вашего репозитория]
pip install requests
python main.py
