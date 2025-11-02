# 🔬 Optical Setup Designer

A powerful web application for designing optical setups visually with real-time ray tracing simulation. Users can drag-and-drop optical components, adjust properties, visualize light paths, and export/import configurations.

![Optical Setup Designer](https://img.shields.io/badge/Vue.js-3.x-brightgreen) ![Python](https://img.shields.io/badge/Python-3.8+-blue) ![Flask](https://img.shields.io/badge/Flask-3.0-lightgrey) ![License](https://img.shields.io/badge/license-MIT-green)

---


## ✨ Features

### Frontend
- 🎨 **Drag-and-drop interface** - Intuitive component placement
- 🔄 **Real-time ray visualization** - See light paths instantly
- 📐 **Component rotation** - 360° rotation with handles
- ⚙️ **Properties editor** - Adjust focal length, reflectivity, etc.
- 📊 **Live statistics** - Ray segments, reflections, refractions
- 💾 **JSON export/import** - Save and load optical setups
- 🎯 **Interaction tracking** - Visual badges showing ray interactions
- 📈 **Detector readings** - Bar graphs showing light intensity

### Backend
- 🚀 **Fast ray tracing engine** - Accurate light path calculations
- 🔬 **Physics simulation** - Reflection, refraction, beam splitting
- 📏 **Path length computation** - Total optical path tracking
- 📊 **Comprehensive statistics** - Detailed simulation metrics
- 🔌 **RESTful API** - Easy integration with any frontend
- ⚡ **CORS enabled** - Cross-origin resource sharing

### Optical Components
- 💡 **Laser Source** - Configurable wavelength and intensity
- 🪞 **Planar Mirror** - Adjustable reflectivity
- 🔍 **Convex Lens** - Variable focal length
- ◇ **Beam Splitter** - Configurable split ratio
- 📡 **Photo Detector** - Sensitivity adjustment

---

## 🎥 Demo

**Live Demo:** [https://optical-designer.vercel.app](https://optical-designer-setup.vercel.app/)

---

## 🛠️ Tech Stack

### Frontend
- **Framework:** Vue.js 3 (Options API)
- **HTTP Client:** Axios
- **Build Tool:** Vue CLI / Vite
- **Styling:** Custom CSS with CSS Grid & Flexbox
- **Deployment:** Vercel

### Backend
- **Language:** Python 3.8+
- **Framework:** Flask 3.0
- **CORS:** Flask-CORS
- **Server:** Gunicorn (production)
- **Deployment:** Render

---

## 📦 Installation

### Prerequisites
- Node.js 16+ and npm
- Python 3.8+
- Git

### 1. Clone the Repository

```bash
git clone https://github.com/shreyasaxena21/optical-setup-designer.git
cd optical-setup-designer
```

### 2. Backend Setup

```bash
# Navigate to backend directory
cd backend

# Create virtual environment
python -m venv venv

# Activate virtual environment
# On Windows:
venv\Scripts\activate
# On Mac/Linux:
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run backend server
python app.py
```

Backend will run on **http://localhost:5000**

### 3. Frontend Setup

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Run development server
npm run dev
```

Frontend will run on **http://localhost:8000**

---

## 🎮 Usage

### Basic Workflow

1. **Add Components**
   - Drag components from the left palette to the grid
   - Components snap to a 20px grid for precision

2. **Position & Rotate**
   - Click and drag to move components
   - Use the blue rotation handle (bottom-right corner) to rotate

3. **Edit Properties**
   - Click a component to select it
   - Modify properties in the right panel:
     - **Source:** Wavelength, Intensity, Spread
     - **Mirror:** Reflectivity, Diameter
     - **Lens:** Focal Length, Diameter, Transmission
     - **Beam Splitter:** Split Ratio, Diameter
     - **Detector:** Sensitivity, Diameter

4. **View Simulation**
   - Rays are visualized automatically in real-time
   - Check statistics in the info overlay (top-left)
   - See detailed results in the right panel

5. **Export/Import**
   - Click **Download JSON** to save your setup
   - Click **Import JSON** (if implemented) to load a previous setup


## 📡 API Documentation

### Base URL
- **Development:** `http://localhost:5000`
- **Production:** `https://your-backend.onrender.com`

### Endpoints

#### 1. Health Check
```http
GET /health
```

**Response:**
```json
{
  "status": "healthy",
  "service": "optical-simulator"
}
```

#### 2. Simulate Optical Setup
```http
POST /simulate
```

**Request Body:**
```json
[
  {
    "id": "source-123",
    "type": "Source",
    "x": 200,
    "y": 300,
    "angle": 0,
    "properties": {
      "wavelength": 632,
      "intensity": 1.0,
      "spread_deg": 0
    }
  },
  {
    "id": "mirror-456",
    "type": "Mirror",
    "x": 400,
    "y": 300,
    "angle": 45,
    "properties": {
      "reflectivity": 0.95,
      "diameter": 50
    }
  }
]
```

**Response:**
```json
{
  "status": "success",
  "results": {
    "rays_out": [
      {
        "x1": 200,
        "y1": 300,
        "x2": 400,
        "y2": 300,
        "int": 1.0,
        "color": "rgba(255, 255, 100, 0.8)",
        "start_port": 0,
        "end_port": 0
      }
    ],
    "detectors": {
      "detector-789": 0.95
    },
    "statistics": {
      "total_path_length_mm": 1234.56,
      "total_ray_segments": 15,
      "total_reflections": 3,
      "total_refractions": 2,
      "total_splits": 1,
      "component_interactions": {
        "mirror-456": {
          "type": "Mirror",
          "interactions": 3
        }
      }
    }
  }
}
```

---

## 🚀 Deployment

### Backend Deployment (Render)

1. **Create `requirements.txt`:**
```txt
flask==3.0.0
flask-cors==4.0.0
gunicorn==21.2.0
```

2. **Update `app.py` for production:**
```python
if __name__ == '__main__':
    import os
    port = int(os.environ.get('PORT', 5000))
    app.run(debug=False, host='0.0.0.0', port=port)
```

3. **Deploy to Render:**
   - Connect GitHub repository
   - Build Command: `pip install -r requirements.txt`
   - Start Command: `gunicorn app:app`
   - Copy the deployed URL

### Frontend Deployment (Vercel)

1. **Update `src/views/HomeView.vue`:**
```javascript
data() {
  const getBackendURL = () => {
    const isProduction = process.env.NODE_ENV === 'production';
    const isLocalhost = typeof window !== 'undefined' && 
                       (window.location.hostname === 'localhost');
    
    if (!isProduction || isLocalhost) {
      return 'http://localhost:5000/simulate';
    }
    return 'https://your-backend.onrender.com/simulate'; // Your Render URL
  };
  
  return {
    BACKEND_URL: getBackendURL(),
    // ... other data
  };
}
```

2. **Deploy to Vercel:**
   - Connect GitHub repository
   - Framework Preset: Vue.js
   - Build Command: `npm run build`
   - Output Directory: `dist`
   - Deploy!

3. **Update CORS in Backend:**
```python
CORS(app, origins=[
    "http://localhost:5173",
    "https://optical-designer-setup.vercel.app/"  # Your Vercel URL
])
```

---

## 📂 Project Structure

```
optical-setup-designer/
├── frontend/
│   ├── public/
│   │   └── index.html
│   ├── src/
│   │   ├── assets/
│   │   │   ├── laser-source.png
│   │   │   ├── planar-mirror.png
│   │   │   ├── convex-lens.png
│   │   │   ├── beam-splitter.png
│   │   │   └── photo-detector.png
│   │   ├── components/
│   │   ├── views/
│   │   │   └── HomeView.vue          # Main optical designer
│   │   ├── App.vue
│   │   └── main.js
│   ├── package.json
│   └── vue.config.js
│
├── backend/
│   ├── app.py                         # Flask application
│   ├── requirements.txt               # Python dependencies
│   └── README.md
│
├── LICENSE
└── README.md
```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. **Fork the repository**
2. **Create a feature branch**
   ```bash
   git checkout -b feature/amazing-feature
   ```
3. **Commit your changes**
   ```bash
   git commit -m "Add amazing feature"
   ```
4. **Push to the branch**
   ```bash
   git push origin feature/amazing-feature
   ```
5. **Open a Pull Request**

### Coding Standards
- Follow Vue.js style guide
- Use ESLint for JavaScript
- Follow PEP 8 for Python
- Write meaningful commit messages
- Add tests for new features

---


