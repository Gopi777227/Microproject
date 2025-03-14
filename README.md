import tkinter as tk
from tkinter import messagebox
import sqlite3
import random

# Connect to SQLite Database
conn = sqlite3.connect("quiz_game.db")
cursor = conn.cursor()

# Create Table for Questions (Run only once)
cursor.execute('''
    CREATE TABLE IF NOT EXISTS quiz_questions (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        question TEXT NOT NULL,
        answer TEXT NOT NULL
    )
''')
conn.commit()

# Insert Sample Questions (Run only once)
sample_questions = [
    ("What is the output of print(2 ** 3)?", "8"),
    ("Which keyword is used to define a function in Python?", "def"),
    ("What data type is the result of: 5 / 2 in Python 3?", "float"),
    ("What is the correct file extension for Python files?", ".py"),
    ("Which function is used to take input from the user?", "input"),
    ("Which library is used for GUI programming in Python?", "tkinter"),
    ("Which statement is used to exit a loop in Python?", "break"),
    ("What is the default port for HTTP?", "80"),
    ("Which operator is used for exponentiation in Python?", "**"),
    ("What is the value of 5 + 5?", "10")
]

# Insert sample questions only if the table is empty
cursor.execute("SELECT COUNT(*) FROM quiz_questions")
if cursor.fetchone()[0] == 0:
    cursor.executemany("INSERT INTO quiz_questions (question, answer) VALUES (?, ?)", sample_questions)
    conn.commit()

# Fetch Questions from SQLite
cursor.execute("SELECT question, answer FROM quiz_questions")
all_questions = cursor.fetchall()
random.shuffle(all_questions)  # Shuffle questions for randomness
selected_questions = all_questions[:5]  # Select 5 random questions

class QuizGame:
    def __init__(self, root):
        self.root = root
        self.root.title("Python Quiz Game")
        self.root.geometry("500x400")
        self.root.configure(bg="#f0f0f0")  # Light background color
        
        self.questions = selected_questions
        self.current_question = 0
        self.score = 0
        self.time_left = 15  # Timer for each question
        
        # Frame for quiz content
        self.frame = tk.Frame(root, bg="white", padx=20, pady=20)
        self.frame.pack(pady=30, fill="both", expand=True)
        
        # Question Label
        self.question_label = tk.Label(self.frame, text=self.questions[self.current_question][0], font=("Arial", 12, "bold"), wraplength=400, fg="black", bg="white")
        self.question_label.pack(pady=10)
        
        # Answer Entry
        self.answer_entry = tk.Entry(self.frame, font=("Arial", 12), width=30)
        self.answer_entry.pack(pady=5)
        
        # Timer Label
        self.timer_label = tk.Label(self.frame, text=f"Time Left: {self.time_left} sec", font=("Arial", 10, "bold"), fg="red", bg="white")
        self.timer_label.pack(pady=5)
        
        # Submit Button
        self.submit_button = tk.Button(self.frame, text="Submit", command=self.check_answer, font=("Arial", 12, "bold"), bg="#4CAF50", fg="white", padx=10, pady=5)
        self.submit_button.pack(pady=10)
        
        # Result Label
        self.result_label = tk.Label(self.frame, text="", font=("Arial", 10, "bold"), fg="blue", bg="white")
        self.result_label.pack(pady=5)
        
        # Score Label
        self.score_label = tk.Label(self.frame, text=f"Score: {self.score}", font=("Arial", 10, "bold"), fg="black", bg="white")
        self.score_label.pack(pady=5)
        
        # Start Timer
        self.update_timer()
    
    def update_timer(self):
        """Updates the timer countdown"""
        if self.time_left > 0:
            self.time_left -= 1
            self.timer_label.config(text=f"Time Left: {self.time_left} sec")
            self.root.after(1000, self.update_timer)
        else:
            self.check_answer(time_up=True)
    
    def check_answer(self, time_up=False):
        """Checks if the answer is correct and moves to the next question"""
        correct_answer = self.questions[self.current_question][1]
        answer = self.answer_entry.get().strip()
        
        if time_up:
            self.result_label.config(text=f"Time's Up! Correct: {correct_answer}", fg="red")
        else:
            if answer.lower() == correct_answer.lower():
                self.score += 1
                self.result_label.config(text="Correct!", fg="green")
            else:
                self.result_label.config(text=f"Wrong! Correct answer: {correct_answer}", fg="red")
        
        self.current_question += 1
        self.answer_entry.delete(0, tk.END)
        self.time_left = 15  # Reset timer for next question
        
        # Update Score Display
        self.score_label.config(text=f"Score: {self.score}")
        
        if self.current_question < len(self.questions):
            self.question_label.config(text=self.questions[self.current_question][0])
            self.update_timer()
        else:
            messagebox.showinfo("Quiz Completed", f"Final Score: {self.score}/{len(self.questions)}")
            self.root.quit()

# Start the Quiz App
if __name__ == "__main__":
    root = tk.Tk()
    app = QuizGame(root)
    root.mainloop()

# Close SQLite Connection
cursor.close()
conn.close()
