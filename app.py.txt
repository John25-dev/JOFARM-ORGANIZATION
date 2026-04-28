import os
import datetime
import requests
from workers import asgi
from flask import Flask, jsonify, request, send_from_directory
from flask_cors import CORS
from flask_bcrypt import Bcrypt
from functools import wraps

app = Flask(__name__, static_folder="static", template_folder=".")
CORS(app)
bcrypt = Bcrypt(app)

# --- Role-based Access Control ---
def roles_required(*allowed_roles):
    def decorator(f):
        @wraps(f)
        async def decorated_function(env, *args, **kwargs):
            user_role = request.headers.get('X-User-Role')
            if not user_role or user_role not in allowed_roles:
                return jsonify({"error": "Forbidden: Access restricted"}), 403
            return await f(env, *args, **kwargs)
        return decorated_function
    return decorator

# --- Landing Page ---
@app.route('/')
def index():
    return send_from_directory('.', 'index.html')

# --- Registration ---
@app.route('/api/register', methods=['POST'])
async def register(env):
    data = request.get_json()
    email = data.get('email', '').lower()
    username = data.get('username')
    password = data.get('password')

    if not email.endswith("@jofarm.com"):
        return jsonify({"error": "Only @jofarm.com emails allowed"}), 403

    hashed_pw = bcrypt.generate_password_hash(password).decode('utf-8')
    role = "SUBORDINATE"

    try:
        await env.DB.prepare(
            "INSERT INTO User (email, username, password_hash, role) VALUES (?, ?, ?, ?)"
        ).bind(email, username, hashed_pw, role).run()
        return jsonify({"status": "success"}), 201
    except:
        return jsonify({"error": "User already exists"}), 400

# --- Login ---
@app.route('/api/login', methods=['POST'])
async def login(env):
    data = request.get_json()
    username = data.get('username')
    password = data.get('password')

    user = await env.DB.prepare(
        "SELECT * FROM User WHERE username = ? OR email = ?"
    ).bind(username, username).first()

    if user and bcrypt.check_password_hash(user['password_hash'], password):
        return jsonify({
            "status": "success",
            "role": user['role'],
            "username": user['username']
        })

    return jsonify({"error": "Invalid credentials"}), 401

# --- Client Financial Transaction ---
@app.route('/api/transaction', methods=['POST'])
async def transaction(env):
    data = request.get_json()
    client_id = data.get('client_id')
    amount = data.get('amount')

    # Capture live location, time, and date
    timestamp = datetime.datetime.utcnow().isoformat()
    ip = request.remote_addr
    location_data = requests.get(f"https://ipapi.co/{ip}/json/").json()

    await env.DB.prepare(
        "INSERT INTO Transactions (client_id, amount, timestamp, location) VALUES (?, ?, ?, ?)"
    ).bind(client_id, amount, timestamp, location_data.get("city")).run()

    return jsonify({
        "status": "success",
        "amount": amount,
        "timestamp": timestamp,
        "location": location_data.get("city")
    }), 201

# --- Admin Dashboard ---
@app.route('/api/admin/financials', methods=['GET'])
@roles_required('CEO', 'COUNTRY MANAGER')
async def get_financials(env):
    return jsonify({"revenue": "UGX 14,500,000", "scope": "Global"})

# Worker Entrypoint
async def on_fetch(request, env):
    return await asgi.fetch(app, request, env)
