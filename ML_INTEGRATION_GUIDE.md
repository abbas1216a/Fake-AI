# VeriLens ML Integration Guide

Complete instructions for connecting real machine learning models to replace heuristic-based detection.

## 🎯 Integration Overview

Current System:
- ✓ Heuristic-based detection (72-88% accuracy)
- ✓ Client-side processing
- ✓ Instant results
- ✗ Limited accuracy
- ✗ No real ML

Target System:
- ✓ ML-powered detection (92%+ accuracy)
- ✓ Backend API integration
- ✓ Confidence scores
- ✓ Explainable signals
- ✓ Continuous improvement

---

## 📋 Pre-Integration Checklist

- [ ] Node.js/Python backend server ready
- [ ] ML models trained and exported
- [ ] API endpoints defined
- [ ] Database for logging results
- [ ] Testing dataset prepared
- [ ] Monitoring/metrics setup

---

## 🔄 Integration Paths

### **Option A: Browser-Based ML (TensorFlow.js)**

**Pros:** No server needed, instant results, privacy-preserving
**Cons:** Limited model size, slower for large files
**Best for:** Text analysis, lightweight image models

### **Option B: Backend API (Node.js/Python)**

**Pros:** Powerful models, scalable, fast
**Cons:** Server dependency, latency
**Best for:** Large video analysis, real-time processing

### **Option C: Hybrid**

**Pros:** Best of both worlds
**Cons:** More complex architecture
**Best for:** Production systems

---

## 📝 Part 1: Text Analysis Integration

### Method 1A: TensorFlow.js (Browser)

```javascript
// Load Hugging Face transformer model
const MODEL_URL = 'https://cdn-models.huggingface.co/...model.json';
let tfModel = null;

async function loadTextModel() {
  try {
    tfModel = await tf.loadGraphModel(MODEL_URL);
    console.log('Text model loaded');
  } catch (e) {
    console.error('Failed to load model:', e);
  }
}

// Replace analyzeNewsText function
async function analyzeNewsText(text) {
  const trimmed = text.trim();
  if (!trimmed) return null;

  // Tokenize text
  const tokens = tokenizer.encode(trimmed);
  const input = tf.tensor2d([tokens]);

  // Get predictions
  const predictions = await tfModel.predict(input);
  const scores = await predictions.data();
  const mlScore = Math.round(scores[0] * 100);

  // Combine with heuristics for better accuracy
  const heuristicScore = calculateHeuristicScore(trimmed);
  const finalScore = Math.round((mlScore * 0.7 + heuristicScore * 0.3));

  // Extract signals
  const signals = [
    {
      name: "🤖 ML Confidence",
      detail: `${Math.round(scores[0] * 100)}% likelihood of AI generation`,
    },
    {
      name: "📊 Perplexity",
      detail: calculatePerplexity(scores) > 100 ? "High (suspicious)" : "Normal",
    },
    {
      name: "🔍 Sensational markers",
      detail: countSensationalMarkers(trimmed) + " detected",
    },
  ];

  return {
    score: finalScore,
    verdict: verdictFromScore(finalScore),
    signals,
    note: `ML-powered analysis | Confidence: ${Math.round(scores[1] * 100)}%`,
  };
}

// Initialize on page load
document.addEventListener('DOMContentLoaded', async () => {
  await loadTextModel();
  initDetection();
});
```

### Method 1B: Backend API (Node.js)

**Server Setup (Express):**

```javascript
const express = require('express');
const bodyParser = require('body-parser');
const tf = require('@tensorflow/tfjs');
require('@tensorflow/tfjs-node');
const app = express();

app.use(bodyParser.json());

// Load pre-trained model
let textModel = null;
async function initModel() {
  textModel = await tf.loadLayersModel('file://./models/text-classifier.json');
}
initModel();

// API endpoint
app.post('/api/analyze/text', async (req, res) => {
  try {
    const { text } = req.body;

    // Preprocess text
    const tokens = preprocessText(text, 512); // Max 512 tokens
    const input = tf.tensor2d([tokens]);

    // Get prediction
    const prediction = textModel.predict(input);
    const scores = await prediction.data();
    const score = Math.round(scores[0] * 100);

    // Generate signals
    const signals = extractTextSignals(text, scores);

    res.json({
      score,
      verdict: getVerdict(score),
      signals,
      confidence: Math.round(scores[1] * 100),
    });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});

app.listen(3001, () => console.log('Server running on port 3001'));
```

**Client Integration:**

```javascript
// Replace analyzeNewsText in app.js
async function analyzeNewsText(text) {
  const trimmed = text.trim();
  if (!trimmed) return null;

  try {
    const response = await fetch('/api/analyze/text', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text: trimmed }),
    });

    if (!response.ok) throw new Error('API error');
    return await response.json();
  } catch (e) {
    console.error('Analysis failed:', e);
    // Fallback to heuristics
    return analyzeNewsTextHeuristics(trimmed);
  }
}
```

---

## 🖼️ Part 2: Image Analysis Integration

### Method 2A: TensorFlow.js CNN (Browser)

```javascript
const IMAGE_MODEL_URL = 'https://tfhub.dev/google/tf2-preview/mobilenet_v2/classification/2';
let imageModel = null;

async function loadImageModel() {
  try {
    imageModel = await tf.loadGraphModel(IMAGE_MODEL_URL);
    console.log('Image model loaded');
  } catch (e) {
    console.error('Failed to load image model:', e);
  }
}

async function analyzeImageFile(file) {
  const buffer = await file.arrayBuffer();
  const uint8Array = new Uint8Array(buffer);

  // Create image tensor
  const img = new Image();
  img.src = URL.createObjectURL(file);

  await new Promise(resolve => img.onload = resolve);

  // Preprocess image
  let tensor = tf.browser.fromPixels(img);
  tensor = tf.image.resizeBilinear(tensor, [224, 224]); // MobileNet input size
  tensor = tf.cast(tensor, 'float32');
  tensor = tensor.div(tf.scalar(255));
  tensor = tensor.expandDims(0);

  // Run prediction
  const predictions = await imageModel.predict(tensor);
  const scores = await predictions.data();

  // Calculate forensic score
  const forensicScore = calculateForensicScore(uint8Array);
  const mlScore = Math.max(...scores) * 100;
  const finalScore = Math.round((mlScore * 0.6 + forensicScore * 0.4));

  // Generate signals
  const signals = [
    {
      name: "🤖 ML Classification",
      detail: mlScore > 70 ? "🚨 Likely AI-generated" : "✓ Natural image",
    },
    {
      name: "📊 Frequency analysis",
      detail: forensicScore > 60 ? "⚠️ GAN artifacts detected" : "Normal",
    },
    {
      name: "🎨 Style consistency",
      detail: analyzeStyleConsistency(img) ? "Consistent" : "Anomalies found",
    },
  ];

  // Cleanup
  tensor.dispose();
  predictions.dispose();

  return {
    score: finalScore,
    verdict: verdictFromScore(finalScore),
    signals,
    note: `Advanced ML + forensic analysis | Confidence: ${Math.round(Math.max(...scores) * 100)}%`,
  };
}

document.addEventListener('DOMContentLoaded', async () => {
  await loadImageModel();
  initDetection();
});
```

### Method 2B: Backend API (Python + FastAPI)

**Server Setup:**

```python
# server.py
from fastapi import FastAPI, File, UploadFile
from fastapi.middleware.cors import CORSMiddleware
import tensorflow as tf
import numpy as np
from PIL import Image
import io

app = FastAPI()

# Enable CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
)

# Load model
model = tf.keras.models.load_model('models/deepfake_image_detector.h5')

@app.post("/api/analyze/image")
async def analyze_image(file: UploadFile = File(...)):
    try:
        # Read image
        contents = await file.read()
        image = Image.open(io.BytesIO(contents))
        
        # Preprocess
        image = image.resize((256, 256))
        image_array = np.array(image) / 255.0
        image_array = np.expand_dims(image_array, axis=0)
        
        # Predict
        predictions = model.predict(image_array)
        score = int(predictions[0][0] * 100)
        
        # Extract signals
        signals = [
            {
                "name": "🤖 ML Detection",
                "detail": f"{score}% likelihood of AI generation"
            },
            {
                "name": "📊 Feature confidence",
                "detail": f"{int(predictions[0][1] * 100)}% certain"
            }
        ]
        
        return {
            "score": score,
            "verdict": get_verdict(score),
            "signals": signals,
            "confidence": int(predictions[0][1] * 100)
        }
        
    except Exception as e:
        return {"error": str(e)}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=3001)
```

**Client Integration:**

```javascript
async function analyzeImageFile(file) {
  const formData = new FormData();
  formData.append('file', file);

  try {
    showLoading(photoResult, 'Running ML image analysis...');
    
    const response = await fetch('/api/analyze/image', {
      method: 'POST',
      body: formData,
    });

    if (!response.ok) throw new Error('API error');
    const result = await response.json();
    
    return result;
  } catch (e) {
    console.error('Image analysis failed:', e);
    // Fallback to heuristics
    return await analyzeImageFileHeuristics(file);
  }
}
```

---

## 🎬 Part 3: Video Analysis Integration

### Method 3A: Complex Backend (Python)

**Server with frame extraction:**

```python
# video_analyzer.py
from fastapi import FastAPI, File, UploadFile
import cv2
import mediapipe as mp
import numpy as np
from collections import deque

app = FastAPI()

# Load face detection model
mp_face = mp.solutions.face_mesh
mp_drawing = mp.solutions.drawing_utils

@app.post("/api/analyze/video")
async def analyze_video(file: UploadFile = File(...)):
    try:
        # Save uploaded file
        contents = await file.read()
        temp_path = f"/tmp/{file.filename}"
        with open(temp_path, 'wb') as f:
            f.write(contents)

        # Extract frames
        cap = cv2.VideoCapture(temp_path)
        frame_count = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))
        fps = cap.get(cv2.CAP_PROP_FPS)

        landmarks_history = deque(maxlen=30)  # 30 frames
        blink_count = 0
        last_eye_open = True

        with mp_face.FaceMesh(
            max_num_faces=1,
            refine_landmarks=True,
            min_detection_confidence=0.5,
            min_tracking_confidence=0.5
        ) as face_mesh:

            while cap.isOpened() and cap.get(cv2.CAP_PROP_POS_FRAMES) < min(300, frame_count):
                ret, frame = cap.read()
                if not ret:
                    break

                # Detect landmarks
                frame_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                results = face_mesh.process(frame_rgb)

                if results.multi_face_landmarks:
                    landmarks = results.multi_face_landmarks[0]
                    landmarks_history.append(landmarks)

                    # Check for blinks
                    left_eye_open = detect_eye_open(landmarks, "left")
                    if not left_eye_open and last_eye_open:
                        blink_count += 1
                    last_eye_open = left_eye_open

            cap.release()

        # Calculate metrics
        temporal_jitter = calculate_jitter(landmarks_history)
        expected_blinks = (frame_count / fps) / 4  # ~4 sec per blink
        blink_anomaly = abs(blink_count - expected_blinks) / max(expected_blinks, 1)

        score = calculate_deepfake_score(temporal_jitter, blink_anomaly)

        signals = [
            {
                "name": "👁️ Facial landmark stability",
                "detail": f"Jitter: {temporal_jitter:.2f}px (threshold: <2px)"
            },
            {
                "name": "👀 Blinking pattern",
                "detail": f"Detected {blink_count} blinks (expected ~{int(expected_blinks)})"
            }
        ]

        return {
            "score": int(score),
            "verdict": get_verdict(score),
            "signals": signals,
            "metrics": {
                "temporal_jitter": temporal_jitter,
                "blink_count": blink_count,
                "frames_analyzed": min(300, frame_count)
            }
        }

    except Exception as e:
        return {"error": str(e)}

def calculate_jitter(landmarks):
    """Calculate how much facial landmarks move"""
    if len(landmarks) < 2:
        return 0
    
    jitters = []
    for i in range(1, len(landmarks)):
        prev = landmarks[i-1]
        curr = landmarks[i]
        
        # Calculate movement for key points (nose, eyes)
        for point_idx in [4, 33, 133]:  # Nose, left eye, right eye
            dx = curr.landmark[point_idx].x - prev.landmark[point_idx].x
            dy = curr.landmark[point_idx].y - prev.landmark[point_idx].y
            jitter = (dx**2 + dy**2) ** 0.5
            jitters.append(jitter)
    
    return np.mean(jitters) if jitters else 0
```

---

## 🔌 Configuration & Deployment

### Environment Variables (.env)

```bash
# API Configuration
API_PORT=3001
API_HOST=0.0.0.0

# Model Paths
TEXT_MODEL_PATH=./models/text-classifier.h5
IMAGE_MODEL_PATH=./models/image-detector.h5
VIDEO_MODEL_PATH=./models/video-classifier.h5

# Thresholds
HIGH_RISK_THRESHOLD=65
MEDIUM_RISK_THRESHOLD=40

# Features
ENABLE_CACHING=true
CACHE_TTL=3600
MAX_FILE_SIZE=50000000  # 50MB

# Logging
LOG_LEVEL=info
ENABLE_METRICS=true
```

### Docker Deployment

```dockerfile
FROM node:18-alpine

WORKDIR /app

# Install Python and dependencies
RUN apk add --no-cache python3 py3-pip

COPY package.json .
RUN npm install

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

EXPOSE 3001
CMD ["npm", "start"]
```

---

## ✅ Testing & Validation

### Unit Tests

```javascript
// test.js
const assert = require('assert');

describe('Text Analysis', () => {
  it('should detect AI-generated text', async () => {
    const aiText = "Furthermore, it is important to note that...";
    const result = await analyzeNewsText(aiText);
    assert(result.score > 60);
  });

  it('should pass authentic text', async () => {
    const realText = "Breaking news: Local community organizes charity drive.";
    const result = await analyzeNewsText(realText);
    assert(result.score < 40);
  });
});

describe('Image Analysis', () => {
  it('should detect GAN-generated images', async () => {
    const dalleImage = fs.readFileSync('./test-images/dalle.jpg');
    const result = await analyzeImageFile(dalleImage);
    assert(result.score > 70);
  });
});
```

---

## 📊 Monitoring & Metrics

```javascript
// metrics.js
const prometheus = require('prom-client');

const analysisCounter = new prometheus.Counter({
  name: 'verilens_analysis_total',
  help: 'Total analyses performed',
  labelNames: ['type', 'result']
});

const analysisLatency = new prometheus.Histogram({
  name: 'verilens_analysis_duration_seconds',
  help: 'Analysis latency',
  labelNames: ['type'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

// Track metrics
async function trackAnalysis(type, fn) {
  const timer = analysisLatency.startTimer({ type });
  try {
    const result = await fn();
    analysisCounter.inc({ type, result: result.score > 65 ? 'high_risk' : 'low_risk' });
    return result;
  } finally {
    timer();
  }
}
```

---

## 🚀 Gradual Rollout Strategy

1. **Phase 1 - Beta**: 10% of users → Real ML models
2. **Phase 2 - Ramp**: 50% of users → Continue monitoring
3. **Phase 3 - Full**: 100% of users → Production deployment

```javascript
// A/B testing
const useMLModels = Math.random() < (getUserBetaPercentage() / 100);

async function analyzeNewsText(text) {
  if (useMLModels) {
    return await analyzeNewsTextML(text);  // Real ML
  } else {
    return analyzeNewsTextHeuristics(text);  // Fallback
  }
}
```

---

## 📚 Resources

- [TensorFlow.js Documentation](https://www.tensorflow.org/js)
- [Hugging Face Models](https://huggingface.co/models)
- [MediaPipe Face Mesh](https://mediapipe.dev/)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)

---

## ✨ Next: Production Launch Checklist

- [ ] All APIs tested with edge cases
- [ ] Error handling and fallbacks
- [ ] Performance optimized (<2s latency)
- [ ] Monitoring and alerting setup
- [ ] Load testing completed
- [ ] Documentation updated
- [ ] User feedback collected
- [ ] Model versioning strategy
- [ ] A/B testing framework
- [ ] Rollback procedures

---

**Ready to deploy? Start with integration path that matches your infrastructure!**

