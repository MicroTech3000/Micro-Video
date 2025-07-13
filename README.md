# Micro-Video from flask import Flask, render_template, request, jsonify
from moviepy.editor import TextClip
import os
import uuid

app = Flask(__name__, static_url_path='/static')

# مجلد الفيديوهات الناتجة
VIDEO_DIR = "static/videos"
os.makedirs(VIDEO_DIR, exist_ok=True)

@app.route("/", methods=["GET"])
def index():
    return render_template("index.html")

@app.route("/generate", methods=["POST"])
def generate():
    prompt = request.form.get("prompt")
    if not prompt:
        return jsonify({"error": "الوصف فارغ."}), 400

    filename = f"{uuid.uuid4().hex}.mp4"
    filepath = os.path.join(VIDEO_DIR, filename)

    try:
        clip = TextClip(prompt, fontsize=72, font="Amiri-Bold", color='white',
                        size=(1280, 720), bg_color='#111', method='caption')
        clip = clip.set_duration(7)
        clip.write_videofile(filepath, fps=24, codec='libx264', audio=False)
    except Exception as e:
        return jsonify({"error": str(e)}), 500

    return jsonify({"video_url": f"/{filepath}"})

if __name__ == '__main__':
    app.run(debug=True, host="0.0.0.0", port=5000)
  flask
moviepy
imageio
imageio-ffmpeg
<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <title>🎬 Micro Video Master</title>
  <link rel="stylesheet" href="/static/style.css" />
  <script src="https://unpkg.com/lucide@latest"></script>
</head>
<body>
  <div class="container">
    <h1><i data-lucide="video"></i> Micro Video Master</h1>
    <form id="videoForm">
      <textarea name="prompt" placeholder="اكتب وصف الفيديو الإبداعي..." required></textarea>
      <button type="submit"><i data-lucide="wand-2"></i> توليد الفيديو</button>
    </form>
    <div class="loader" id="loader" style="display:none;">⏳ يتم إنشاء الفيديو...</div>
    <div id="videoResult" style="display:none;">
      <h3>✅ تم إنشاء الفيديو:</h3>
      <video id="videoPlayer" controls></video>
    </div>
  </div>

  <script>
    lucide.createIcons();

    const form = document.getElementById('videoForm');
    const loader = document.getElementById('loader');
    const result = document.getElementById('videoResult');
    const player = document.getElementById('videoPlayer');

    form.onsubmit = async (e) => {
      e.preventDefault();
      loader.style.display = 'block';
      result.style.display = 'none';

      const formData = new FormData(form);
      const res = await fetch('/generate', { method: 'POST', body: formData });
      const data = await res.json();
      loader.style.display = 'none';

      if (data.video_url) {
        player.src = data.video_url;
        result.style.display = 'block';
      } else {
        alert('حدث خطأ: ' + (data.error || 'غير معروف'));
      }
    };
  </script>
</body>
</html>

body {
  font-family: 'Tajawal', sans-serif;
  background: linear-gradient(to right, #0f2027, #203a43, #2c5364);
  color: white;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0;
}

.container {
  background: #1b1b1b;
  padding: 30px;
  border-radius: 16px;
  box-shadow: 0 0 25px rgba(0, 255, 255, 0.1);
  width: 600px;
  text-align: center;
}

h1 {
  font-size: 28px;
  margin-bottom: 20px;
}

textarea {
  width: 100%;
  height: 100px;
  border-radius: 12px;
  border: none;
  padding: 15px;
  font-size: 16px;
  background: #2c2c2c;
  color: white;
  margin-bottom: 15px;
  resize: none;
}

button {
  background: #00cec9;
  color: white;
  padding: 12px 25px;
  font-size: 16px;
  border: none;
  border-radius: 10px;
  cursor: pointer;
  transition: 0.3s;
  display: inline-flex;
  align-items: center;
  gap: 8px;
}

button:hover {
  background: #00b894;
}

video {
  margin-top: 15px;
  width: 100%;
  border-radius: 10px;
}

.loader {
  margin-top: 15px;
  font-size: 18px;
  color: #ffeaa7;
}

mkdir -p static/videos

