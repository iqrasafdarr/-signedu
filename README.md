# SignEdu 🤟

**An accessibility-focused e-learning platform that teaches American Sign Language (ASL) through real-time, webcam-based sign recognition.**

SignEdu pairs a Laravel web application with a Python computer-vision microservice: a student signs a letter in front of their webcam, and the app tells them — in real time — which letter they signed, how confident it is, and how to improve their hand shape.

---

## ✨ Key Features

- **Real-time ASL fingerspelling recognition** — captures webcam frames in the browser and classifies the ASL alphabet sign being shown, with a live confidence score and a written description of the correct handshape.
- **Sample & reference images** — pulls a matching reference image for the detected/selected letter so learners can visually compare their hand shape.
- **Role-based access control** — separate **Admin** and **Student** roles, enforced via dedicated middleware, so admin-only features (like managing student records) are properly gated.
- **Student management (CRUD)** — searchable, paginated student directory with create/edit/delete, built on Laravel's validated Form Requests.
- **Authentication & profiles** — registration, login, password reset, and profile management via Laravel Breeze.
- **Dashboard** — at-a-glance stats (total students, total users, recently added students) for admins.

## 🧠 How the Recognition Works

The core ML pipeline lives in `python-api/`:

1. **Hand landmark extraction** — [MediaPipe Hands](https://developers.google.com/mediapipe) detects a hand in the incoming frame and returns 21 3D landmarks.
2. **Feature engineering** — each landmark is normalized relative to the wrist (x, y, z offsets), producing a 63-dimensional feature vector that's invariant to the hand's position in the frame.
3. **Classification** — a **Random Forest** classifier (scikit-learn, 200 estimators) maps that feature vector to one of the 26 ASL alphabet letters.
4. **Training data** — the model is trained on the [ASL Alphabet dataset](https://www.kaggle.com/datasets/grassknoted/asl-alphabet) (~87K images across 26 classes), sampling per-class images and running them through the same landmark-extraction pipeline used at inference time.
5. **Serving** — a lightweight Flask API (`app.py`) loads the trained model (`model.pkl`) and exposes `/detect`, `/explain`, `/sample/<letter>`, and `/health` endpoints, called directly from the browser-facing Blade view.

This landmark-based approach (rather than classifying raw pixels) keeps the model small and fast enough for real-time, in-browser use.

## 🏗️ Architecture

```
Browser (webcam capture, JS)
        │  frame (multipart/form-data)
        ▼
Flask API  (python-api/app.py)
  MediaPipe Hands → feature extraction → Random Forest (model.pkl)
        │  JSON {letter, confidence, description, sample_image}
        ▼
Laravel App  (auth, roles, dashboard, student records, views)
```

The Laravel app owns everything web-facing — auth, authorization, UI, and the student-management module — while the Flask service is a focused, single-purpose inference API that keeps the ML code decoupled from the web app.

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend (web) | PHP 8.3+, Laravel 13, Laravel Breeze |
| Frontend | Blade, Tailwind CSS, Alpine.js, Vite |
| ML / computer vision service | Python, Flask, OpenCV, MediaPipe, scikit-learn, Pillow |
| Database | SQLite (default) / MySQL-compatible via Laravel's query builder |
| Testing | Pest |
| Containerization | Docker |

## 🚀 Getting Started

### Prerequisites
- PHP 8.3+, Composer
- Node.js & npm
- Python 3.9+
- A webcam (for the sign-detector feature)

### 1. Clone & install the Laravel app
```bash
git clone https://github.com/iqrasafdarr/-signedu.git
cd -signedu

composer install
npm install

cp .env.example .env
php artisan key:generate

touch database/database.sqlite   # if using SQLite
php artisan migrate --seed

npm run build      # or `npm run dev` while developing
php artisan serve
```

### 2. Set up the ML service
```bash
cd python-api
pip install flask flask-cors opencv-python mediapipe numpy scikit-learn pillow

python app.py       # serves the recognition API on http://127.0.0.1:5000
```

> To retrain the model on your own copy of the ASL Alphabet dataset, update `DATASET_PATH` in `train.py` / `app.py` and run `python train.py`. This regenerates `model.pkl`.

With both servers running, log in, navigate to **Sign Detector**, allow webcam access, and start signing.

## 📁 Project Structure

```
├── app/                    # Laravel application code
│   ├── Http/Controllers/   # Dashboard, Students, SignDetector, Auth
│   ├── Http/Middleware/    # AdminMiddleware, StudentMiddleware
│   └── Models/             # User, Student
├── database/migrations/    # Users, students, roles
├── resources/views/        # Blade templates (dashboard, students, sign-detector)
├── python-api/
│   ├── app.py               # Flask inference API
│   ├── train.py              # Model training pipeline
│   └── model.pkl             # Trained Random Forest classifier
└── routes/web.php           # Application routes
```

## 🗺️ Roadmap

- [ ] Deploy the Flask service alongside the web app instead of relying on `127.0.0.1:5000` for production use
- [ ] Expand beyond static fingerspelling to dynamic/word-level signs
- [ ] Track per-student learning progress over time
- [ ] Add automated tests for the recognition API

## 📄 License

This project is open-sourced for educational purposes.

## 👤 Author

Built by [Iqra Safdar](https://github.com/iqrasafdarr).
