### Valenfind

## Description

Can you find vulnerabilities in this new dating app?

-img

## Solution

First of all let's connect to the site

-1page

start our jurney and we arrive in a register page

-register

after submitting all the profile information we finally login in the site and we arrive in the dashboard

-dashboard

at this poi it took me a while exploring the source code of the site analyzing all the code in every page, it's a dating site and it display various users profile where we can send them likes.

I found something interesting in the Cupid profile

``` html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ValenFind - Secure Dating</title>
    <style>
        :root { --primary: #ff4757; --secondary: #ff6b81; --bg: #ffe2e6; --card: #fff; --text: #2f3542; }
        body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; background: var(--bg); color: var(--text); margin: 0; padding: 0; min-height: 100vh; display: flex; flex-direction: column; }
        .nav { background: var(--primary); padding: 1rem 2rem; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        .nav a { color: white; text-decoration: none; margin-left: 20px; font-weight: 600; }
        .brand { font-size: 1.5rem; font-weight: bold; color: white; display: flex; align-items: center; gap: 10px; }
        .container { flex: 1; padding: 2rem; max-width: 900px; margin: 0 auto; width: 100%; box-sizing: border-box; }
        .card { background: var(--card); border-radius: 12px; padding: 2rem; box-shadow: 0 4px 6px rgba(0,0,0,0.05); margin-bottom: 1.5rem; }
        .btn { background: var(--primary); color: white; border: none; padding: 10px 20px; border-radius: 25px; cursor: pointer; font-size: 0.95rem; text-decoration: none; display: inline-block; transition: 0.2s; }
        .btn:hover { background: var(--secondary); transform: translateY(-1px); }
        .avatar { width: 60px; height: 60px; border-radius: 50%; display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; font-size: 1.5rem; }
        input, textarea { width: 100%; padding: 12px; margin: 8px 0; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; font-family: inherit; }
        .flash { background: #ff7675; color: white; padding: 10px; border-radius: 8px; margin-bottom: 15px; text-align: center; }
    </style>
</head>
<body>
    <div class="nav">
        <div class="brand"><span>\U0001f498</span> ValenFind</div>
        <div>
            
                <a href="/dashboard">User Profiles</a>
                <a href="/my_profile">My Profile</a>
                <a href="/logout">Logout</a>
            
        </div>
    </div>
    <div class="container">
        
            
        
        
<div class="card" style="max-width: 600px; margin: 0 auto; text-align: center;">
    
    <div style="width: 150px; height: 150px; margin: 0 auto 20px auto; position: relative; border-radius: 50%; overflow: hidden; border: 4px solid #fff; box-shadow: 0 5px 15px rgba(0,0,0,0.1);">
        <img src="/static/avatars/cupid.jpg" 
             alt="cupid" 
             style="width: 100%; height: 100%; object-fit: cover;"
             onerror="this.style.display='none'; this.nextElementSibling.style.display='flex'">
        
        <div style="display: none; width: 100%; height: 100%; background-color: #d13a66; align-items: center; justify-content: center; color: white; font-size: 4rem; position: absolute; top: 0; left: 0;">
            C
        </div>
    </div>

    <div style="margin-bottom: 20px; text-align: right;">
        <label for="theme-selector" style="font-size: 0.8rem; color: #666;">Profile Theme:</label>
        <select id="theme-selector" onchange="loadTheme(this.value)" style="padding: 5px; border-radius: 5px; border: 1px solid #ddd;">
            <option value="theme_classic.html">Classic Romance</option>
            <option value="theme_modern.html">Modern Dark</option>
            <option value="theme_romance.html">Cupid's Choice</option>
        </select>
    </div>

    <div id="bio-container">
        <p style="color:#999;">Loading layout...</p>
    </div>

    <hr style="border: 0; border-top: 1px solid #eee; margin: 20px 0;">

    <form action="/like/8" method="POST">
        <button class="btn" style="width: 100%; font-size: 1.1rem; padding: 15px;">\U0001f498 Send Valentine</button>
    </form>
</div>

<script>
    // Initial load
    document.addEventListener("DOMContentLoaded", function() {
        loadTheme('theme_classic.html');
    });

    function loadTheme(layoutName) {
        // Feature: Dynamic Layout Fetching
        // Vulnerability: 'layout' parameter allows LFI
        fetch(`/api/fetch_layout?layout=${layoutName}`)
            .then(r => r.text())
            .then(html => {
                const bioText = "I keep the database secure. No peeking.";
                const username = "cupid";
                
                // Client-side rendering of the fetched template
                let rendered = html.replace('__USERNAME__', username)
                                   .replace('__BIO__', bioText);
                
                document.getElementById('bio-container').innerHTML = rendered;
            })
            .catch(e => {
                console.error(e);
                document.getElementById('bio-container').innerText = "Error loading theme.";
            });
    }
</script>

    </div>
</body>
</html>
```
Here we find an important hin: 

```
 // Vulnerability: 'layout' parameter allows LFI
```

This is the vulnerability we were searching, Local File Inclusion (LFI)

let's try to read file /etc/passwd

-etcpsswd

IT WORKS!

So the LFI is confirmed!

Now we have to read the app flask file of the server

http://10.64.172.1:5000/api/fetch_layout?layout=../../app.py

-appy

reviewing the source code we can find an ADMIN API KEY hardcoded

ADMIN_API_KEY = "CUPID_MASTER_KEY_2024_XOXO"

also this section of the code means we can export the full DB

```python
@app.route('/api/admin/export_db')
def export_db():
    auth_header = request.headers.get('X-Valentine-Token')
    if auth_header == ADMIN_API_KEY:
        return send_file(DATABASE)
```

So lets execute a curl request to get the DB, and than exploring whats inside we can find the flag

-flag




