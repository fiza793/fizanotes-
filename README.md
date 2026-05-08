# fizanotes-from flask import Flask, request, redirect
import sqlite3

app = Flask(__name__)

# ---------- DATABASE ----------
def init_db():
    conn = sqlite3.connect("notes.db")
    c = conn.cursor()
    c.execute("""
        CREATE TABLE IF NOT EXISTS notes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            note TEXT
        )
    """)
    conn.commit()
    conn.close()

init_db()

# ---------- LOGIN ----------
USERNAME = "admin"
PASSWORD = "1234"

@app.route('/')
def login():
    return '''
    <h1>🔐 Smart Study Portal</h1>

    <form action="/check" method="post">
        <input name="username" placeholder="Username"><br><br>
        <input name="password" type="password" placeholder="Password"><br><br>
        <button>Login</button>
    </form>
    '''

@app.route('/check', methods=['POST'])
def check():
    u = request.form['username']
    p = request.form['password']

    if u == USERNAME and p == PASSWORD:
        return redirect('/home')
    return "<h2>❌ Wrong Login</h2>"

# ---------- HOME ----------
@app.route('/home')
def home():
    return '''
    <h1>🔐 FizaNotes Login</h1>

    <form action="/add" method="post">
        <input name="note" placeholder="Enter your note">
        <button>Add Note</button>
    </form>

    <br>
    <a href="/notes">📖 View Notes</a>
    '''

# ---------- ADD ----------
@app.route('/add', methods=['POST'])
def add():
    note = request.form['note']

    conn = sqlite3.connect("notes.db")
    c = conn.cursor()
    c.execute("INSERT INTO notes (note) VALUES (?)", (note,))
    conn.commit()
    conn.close()

    return redirect('/home')

# ---------- SHOW ----------
@app.route('/notes')
def notes():
    conn = sqlite3.connect("notes.db")
    c = conn.cursor()
    c.execute("SELECT * FROM notes")
    data = c.fetchall()
    conn.close()

    html = "<h1>📖 Your Notes</h1>"

    for row in data:
        html += f"""
        <div style='padding:10px;background:#eee;margin:10px'>
            {row[1]}<br><br>

            <a href="/edit/{row[0]}">✏ Edit</a> |
            <a href="/delete/{row[0]}">❌ Delete</a>
        </div>
        """

    html += "<br><a href='/home'>🏠 Home</a>"
    return html

# ---------- DELETE ----------
@app.route('/delete/<int:id>')
def delete(id):
    conn = sqlite3.connect("notes.db")
    c = conn.cursor()
    c.execute("DELETE FROM notes WHERE id=?", (id,))
    conn.commit()
    conn.close()

    return redirect('/notes')

# ---------- EDIT ----------
@app.route('/edit/<int:id>')
def edit(id):
    conn = sqlite3.connect("notes.db")
    c = conn.cursor()
    c.execute("SELECT * FROM notes WHERE id=?", (id,))
    note = c.fetchone()
    conn.close()

    return f"""
    <h1>Edit Note</h1>

    <form action="/update/{id}" method="post">
        <input name="note" value="{note[1]}">
        <button>Update</button>
    </form>
    """

# ---------- UPDATE ----------
@app.route('/update/<int:id>', methods=['POST'])
def update(id):
    new_note = request.form['note']

    conn = sqlite3.connect("notes.db")
    c = conn.cursor()
    c.execute("UPDATE notes SET note=? WHERE id=?", (new_note, id))
    conn.commit()
    conn.close()

    return redirect('/notes')

app.run(host='0.0.0.0', port=5000)
