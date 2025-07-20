from flask import Flask, render_template_string, request, redirect, session, url_for

app = Flask(__name__)
app.secret_key = 'your_secret_key'

users = {
    "admin": {"password": "admin123", "role": "admin"},
    "niyamul": {"password": "1234", "role": "user"},
}

posts = []

login_html = '''
<!DOCTYPE html>
<html>
<head>
  <title>Login</title>
  <style>
    body { background: linear-gradient(to right, #00b09b, #96c93d); font-family: sans-serif; display: flex; justify-content: center; align-items: center; height: 100vh; }
    .box { background: white; padding: 30px; border-radius: 10px; box-shadow: 0 0 15px rgba(0,0,0,0.3); width: 300px; }
    input, button { width: 100%; padding: 10px; margin-top: 10px; border-radius: 8px; border: 1px solid #ccc; }
    button { background: #00b09b; color: white; font-weight: bold; border: none; }
  </style>
</head>
<body>
  <div class="box">
    <h2>🔐 Login</h2>
    <form method="POST">
      <input type="text" name="username" placeholder="Username" required>
      <input type="password" name="password" placeholder="Password" required>
      <button type="submit">Login</button>
    </form>
    {% if error %}<p style="color:red;">{{ error }}</p>{% endif %}
  </div>
</body>
</html>
'''

dashboard_html = '''
<!DOCTYPE html>
<html>
<head>
  <title>Dashboard</title>
  <style>
    body { font-family: Arial; padding: 20px; background: #f4f4f4; }
    .post-box { background: white; padding: 15px; margin-bottom: 20px; border-radius: 10px; box-shadow: 0 0 10px #ccc; }
    form textarea, form input { width: 100%; margin-top: 5px; padding: 10px; border-radius: 8px; border: 1px solid #ccc; }
    form button { background: #2980b9; color: white; padding: 10px; margin-top: 10px; border: none; border-radius: 10px; cursor: pointer; }
    .like-btn { background: #27ae60; }
    .comment-box { background: #ecf0f1; padding: 10px; border-radius: 8px; margin-top: 10px; }
  </style>
</head>
<body>
  <h2>👋 Welcome, {{ username }} ({{ role }})</h2>
  {% if role == 'admin' %}
  <form method="post" action="/post">
    <textarea name="text" placeholder="Write your post..."></textarea>
    <input type="text" name="image_url" placeholder="Image URL (optional)">
    <button type="submit">Post</button>
  </form>
  {% endif %}

  <hr>

  {% for post in posts %}
  <div class="post-box">
    <p>{{ post.text }}</p>
    {% if post.image_url %}<img src="{{ post.image_url }}" width="100%"><br>{% endif %}
    <form method="post" action="/like">
      <input type="hidden" name="index" value="{{ loop.index0 }}">
      <button type="submit" class="like-btn">👍 Like ({{ post.likes }})</button>
    </form>
    <div class="comment-box">
      <form method="post" action="/comment">
        <input type="hidden" name="index" value="{{ loop.index0 }}">
        <input type="text" name="comment" placeholder="Write a comment...">
        <button type="submit">Comment</button>
      </form>
      <ul>{% for comment in post.comments %}<li>{{ comment }}</li>{% endfor %}</ul>
    </div>
  </div>
  {% endfor %}
</body>
</html>
'''

@app.route('/', methods=['GET', 'POST'])
def login():
    error = None
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']
        user = users.get(username)
        if user and user['password'] == password:
            session['username'] = username
            session['role'] = user['role']
            return redirect('/dashboard')
        else:
            error = "❌ Username বা Password ভুল!"
    return render_template_string(login_html, error=error)

@app.route('/dashboard')
def dashboard():
    if 'username' not in session:
        return redirect('/')
    return render_template_string(dashboard_html, username=session['username'], role=session['role'], posts=posts)

@app.route('/post', methods=['POST'])
def post():
    if session.get('role') != 'admin':
        return redirect('/')
    text = request.form['text']
    image_url = request.form.get('image_url', '')
    posts.insert(0, {'text': text, 'image_url': image_url, 'likes': 0, 'comments': []})
    return redirect('/dashboard')

@app.route('/like', methods=['POST'])
def like():
    index = int(request.form['index'])
    posts[index]['likes'] += 1
    return redirect('/dashboard')

@app.route('/comment', methods=['POST'])
def comment():
    index = int(request.form['index'])
    comment = request.form['comment']
    if comment.strip():
        posts[index]['comments'].append(comment)
    return redirect('/dashboard')

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
