from flask import Flask, request, jsonify, render_template
from moviepy.video.io.ffmpeg_tools import ffmpeg_extract_subclip
import os
import uuid

app = Flask(_name_)
UPLOAD_FOLDER = 'uploads'
OUTPUT_FOLDER = 'highlights'

os.makedirs(UPLOAD_FOLDER, exist_ok=True)
os.makedirs(OUTPUT_FOLDER, exist_ok=True)


@app.route('/')
def index():
    return render_template("index.html")


@app.route('/process', methods=['POST'])
def process():
    data = request.get_json()
    timestamps = data.get('timestamps', [])
    video_filename = data.get('filename', 'uploaded.mp4')

    input_path = os.path.join(UPLOAD_FOLDER, video_filename)
    output_clips = []

    for i, ts in enumerate(timestamps):
        start = max(0, ts - 1)
        end = ts + 1
        out_file = f'highlight_{i}_{uuid.uuid4().hex}.mp4'
        out_path = os.path.join(OUTPUT_FOLDER, out_file)
        ffmpeg_extract_subclip(input_path, start, end, targetname=out_path)
        output_clips.append(out_file)

    return jsonify({'clips': output_clips})


@app.route('/upload', methods=['POST'])
def upload():
    video = request.files['video']
    filename = 'uploaded.mp4'
    path = os.path.join(UPLOAD_FOLDER, filename)
    video.save(path)
    return jsonify({'filename': filename})


if _name_ == '_main_':
    app.run(debug=True)# -
