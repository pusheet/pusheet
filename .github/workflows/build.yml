import json
import os
import requests
from datetime import datetime

# ==========================================
# ⚙️ FULL ENGINE SETTINGS
# ==========================================
DB_FILE = "database.json"
CLIENT_ID = "iZIs9mchV7lxYDLnu7X797ps9SInC37T" 
SEARCH_QUERY = "2026" 
LIMIT = 40 # Увеличил лимит для объема

# Создаем структуру папок (Твой план)
os.makedirs("track", exist_ok=True)
os.makedirs("artist", exist_ok=True)
os.makedirs("album", exist_ok=True)
os.makedirs("css", exist_ok=True)

# ==========================================
# 1. DATABASE SYSTEM
# ==========================================
print("💾 Initializing Pusheet Database...")
db = {"releases": {}, "artists": {}, "albums": {}}

if os.path.exists(DB_FILE):
    try:
        with open(DB_FILE, "r", encoding='utf-8') as f:
            db = json.load(f)
            print(f"✅ Loaded {len(db['releases'])} existing tracks.")
    except Exception as e:
        print(f"⚠️ DB Load Error: {e}")

# ==========================================
# 2. SC SCANNER (DEEP SEARCH)
# ==========================================
print(f"🔍 Searching for new 2026 sounds...")

try:
    url = f"https://api-v2.soundcloud.com/search/tracks?q={SEARCH_QUERY}&client_id={CLIENT_ID}&limit={LIMIT}&filter.created_at=last_day"
    response = requests.get(url, timeout=15)
    data = response.json()
    
    new_tracks = 0
    for item in data.get('collection', []):
        rid = str(item['id'])
        aid = str(item['user']['id'])
        
        # Инъекция в базу треков
        if rid not in db['releases']:
            new_tracks += 1
            db['releases'][rid] = {
                "id": rid,
                "title": item['title'].replace('"', '').replace("'", ""),
                "artist": item['user']['username'],
                "artist_id": aid,
                "image": (item.get('artwork_url') or item['user'].get('avatar_url', '')).replace('large', 't500x500'),
                "url": item['permalink_url'],
                "genre": item.get('genre', 'Unknown'),
                "date_added": datetime.now().strftime("%d.%m.%Y"),
                "sc_date": item.get('created_at', '')
            }

        # Инъекция в базу артистов
        if aid not in db['artists']:
            db['artists'][aid] = {
                "id": aid,
                "name": item['user']['username'],
                "avatar": item['user'].get('avatar_url', '').replace('large', 't500x500'),
                "description": item['user'].get('description', 'Pusheet Artist'),
                "releases": []
            }
        
        if rid not in db['artists'][aid]['releases']:
            db['artists'][aid]['releases'].append(rid)

    print(f"🔥 Found {new_tracks} new releases to process.")
    
    with open(DB_FILE, "w", encoding='utf-8') as f:
        json.dump(db, f, indent=2, ensure_ascii=False)

except Exception as e:
    print(f"❌ Scanner Error: {e}")

# ==========================================
# 3. HTML ARCHITECT (DELA GOTHIC STYLE)
# ==========================================

def generate_header(title, depth=0):
    # depth=1 означает, что файл в подпапке, нужно ../ для путей
    prefix = "../" * depth
    return f"""<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{title} | PUSHEET</title>
    <link href="https://fonts.googleapis.com/css2?family=Dela+Gothic+One&display=swap" rel="stylesheet">
    <style>
        :root {{ --red: #ff0000; --black: #000; --dark: #0a0a0a; --font: 'Dela Gothic One', cursive; }}
        body {{ background: var(--black); color: #fff; font-family: sans-serif; margin: 0; }}
        .font-dela {{ font-family: var(--font); text-transform: uppercase; }}
        header {{ padding: 25px; border-bottom: 5px solid var(--red); display: flex; justify-content: space-between; align-items: center; }}
        .logo {{ font-family: var(--font); color: var(--red); text-decoration: none; font-size: 2rem; }}
        .container {{ max-width: 1200px; margin: 0 auto; padding: 20px; }}
        .grid {{ display: grid; grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); gap: 20px; }}
        .card {{ background: var(--dark); border: 1px solid #222; text-decoration: none; color: #fff; transition: 0.3s; }}
        .card:hover {{ border-color: var(--red); transform: translateY(-5px); }}
        .card img {{ width: 100%; aspect-ratio: 1; object-fit: cover; }}
        .card-info {{ padding: 15px; }}
        .btn-main {{ background: var(--red); color: #fff; padding: 15px 30px; text-decoration: none; display: inline-block; font-family: var(--font); }}
        .track-hero {{ display: flex; gap: 50px; padding: 60px 0; flex-wrap: wrap; }}
        .track-cover {{ width: 400px; height: 400px; border: 4px solid var(--red); box-shadow: 0 0 30px rgba(255,0,0,0.3); }}
    </style>
</head>
<body>
<header>
    <a href="/" class="logo">PUSHEET</a>
</header>
"""

# --- ГЕНЕРАЦИЯ СТРАНИЦ ТРЕКОВ (/track/) ---
for rid, t in db['releases'].items():
    html = generate_header(t['title'], depth=1)
    html += f"""
    <div class="container">
        <div class="track-hero">
            <img src="{t['image']}" class="track-cover">
            <div class="track-data">
                <div style="background:var(--red); padding:5px 10px; display:inline-block; font-size:12px;" class="font-dela">NEW RELEASE</div>
                <h1 class="font-dela" style="font-size:3.5rem; margin:15px 0;">{t['title']}</h1>
                <a href="../artist/{t['artist_id']}.html" class="font-dela" style="color:#888; text-decoration:none; font-size:1.5rem;">{t['artist']}</a>
                <p style="margin: 30px 0; color:#555;">GENRE: {t['genre']} | ADDED: {t['date_added']}</p>
                <a href="{t['url']}" target="_blank" class="btn-main">▶ LISTEN ON SOUNDCLOUD</a>
            </div>
        </div>
    </div>
    </body></html>
    """
    with open(f"track/{rid}.html", "w", encoding='utf-8') as f:
        f.write(html)

# --- ГЕНЕРАЦИЯ СТРАНИЦ АРТИСТОВ (/artist/) ---
for aid, art in db['artists'].items():
    html = generate_header(art['name'], depth=1)
    html += f"""
    <div style="background: linear-gradient(180deg, #200 0%, #000 100%); text-align:center; padding: 80px 20px;">
        <img src="{art['avatar']}" style="width:180px; height:180px; border-radius:50%; border:5px solid var(--red); object-fit:cover; margin-bottom:20px;">
        <h1 class="font-dela" style="font-size:4rem; margin:0;">{art['name']}</h1>
    </div>
    <div class="container">
        <h2 class="font-dela" style="border-left:8px solid var(--red); padding-left:15px; margin-bottom:40px;">DISCOGRAPHY</h2>
        <div class="grid">
    """
    for rid in reversed(art['releases']):
        if rid in db['releases']:
            rel = db['releases'][rid]
            html += f"""
            <a href="../track/{rel['id']}.html" class="card">
                <img src="{rel['image']}">
                <div class="card-info">
                    <div class="font-dela" style="color:var(--red); font-size:0.9rem;">{rel['title']}</div>
                </div>
            </a>
            """
    html += "</div></div></body></html>"
    with open(f"artist/{aid}.html", "w", encoding='utf-8') as f:
        f.write(html)

# --- ГЕНЕРАЦИЯ ГЛАВНОЙ (index.html) ---
html = generate_header("HOME", depth=0)
html += """
<div style="text-align:center; padding: 100px 20px; background: #050505;">
    <h1 class="font-dela" style="font-size:5rem; margin:0; line-height:0.9;">PUSHEET<br><span style="color:var(--red)">DATABASE</span></h1>
    <p style="color:#444; font-weight:bold; letter-spacing:3px; margin-top:20px;">AUTOMATED MUSIC ARCHIVE 2026</p>
</div>
<div class="container">
    <h2 class="font-dela" style="border-left:8px solid var(--red); padding-left:15px; margin-bottom:40px;">LATEST DROPS</h2>
    <div class="grid">
"""
# Сортировка: последние добавленные треки
sorted_ids = list(db['releases'].keys())[::-1][:50]
for rid in sorted_ids:
    rel = db['releases'][rid]
    html += f"""
    <a href="track/{rel['id']}.html" class="card">
        <img src="{rel['image']}">
        <div class="card-info">
            <div class="font-dela" style="color:var(--red); font-size:0.8rem; white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">{rel['title']}</div>
            <div style="font-size:0.7rem; color:#666; margin-top:5px; font-weight:bold;">{rel['artist']}</div>
        </div>
    </a>
    """
html += "</div></div><footer style='padding:100px; text-align:center; color:#222;' class='font-dela'>PUSHEET SYSTEM ONLINE</footer></body></html>"

with open("index.html", "w", encoding='utf-8') as f:
    f.write(html)

print("✅ BUILD FINISHED: All pages generated in /track/ and /artist/.")
