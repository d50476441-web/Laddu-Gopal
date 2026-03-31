from flask import Flask, render_template, request, redirect, session
import sqlite3
import stripe

app = Flask(__name__)
app.secret_key = "secret123"

# Stripe test key (replace with your own)
stripe.api_key = "sk_test_yourkey"

# Database setup
def init_db():
    conn = sqlite3.connect("database.db")
    conn.execute('''CREATE TABLE IF NOT EXISTS users
                 (id INTEGER PRIMARY KEY, username TEXT, password TEXT)''')
    conn.close()

init_db()

# Home
@app.route('/')
def home():
    return render_template("home.html")

# Register
@app.route('/register', methods=['GET','POST'])
def register():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']

        conn = sqlite3.connect("database.db")
        conn.execute("INSERT INTO users (username, password) VALUES (?,?)",
                     (username, password))
        conn.commit()
        conn.close()
        return redirect('/login')

    return render_template("register.html")

# Login
@app.route('/login', methods=['GET','POST'])
def login():
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']

        conn = sqlite3.connect("database.db")
        user = conn.execute("SELECT * FROM users WHERE username=? AND password=?",
                            (username, password)).fetchone()
        conn.close()

        if user:
            session['user'] = username
            return redirect('/')
        else:
            return "Invalid login"

    return render_template("login.html")

# Checkout Page
@app.route('/checkout')
def checkout():
    return render_template("checkout.html")

# Payment
@app.route('/create-checkout-session')
def create_checkout_session():
    session = stripe.checkout.Session.create(
        payment_method_types=['card'],
        line_items=[{
            'price_data': {
                'currency': 'inr',
                'product_data': {'name': 'Laddu Gopal Dress'},
                'unit_amount': 50000,  # ₹500
            },
            'quantity': 1,
        }],
        mode='payment',
        success_url='http://localhost:5000',
        cancel_url='http://localhost:5000',
    )
    return redirect(session.url, code=303)

if __name__ == '__main__':
    app.run(debug=True)
    
