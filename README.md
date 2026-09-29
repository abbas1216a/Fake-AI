VeriLens — AI Fake Media Detection (University Project)

A presentation-ready website for detecting AI-generated fake news, synthetic photos, and deepfake-style video. The UI is fully interactive; analysis in the browser uses demo heuristics so you can demo the flow before connecting real ML models.

Open the site

Option A — double-click

Open index.html in Chrome or Edge.

Option B — local server (recommended)

cd C:\Users\DELL\Projects\ai-fake-media-detector
npx --yes serve .

Then visit the URL shown (usually http://localhost:3000).

What to show in class





Fake news text — paste article text or click Load sample, then Analyze text.



AI photos — upload a JPG/PNG and review the signal breakdown.



Deepfake video — upload a short MP4/WebM clip.

Explain that scores are explainable signals (sensational language, compression, temporal jitter) and that your final submission can plug in Python/TensorFlow or API endpoints in app.js.

Connect real models later

In app.js, replace the functions analyzeNewsText, analyzeImageFile, and analyzeVideoFile with fetch() calls to your backend, for example:

const res = await fetch("/api/analyze/news", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ text }),
});
return res.json();



Project structure







File



Purpose





index.html



Layout, sections, accessibility





styles.css



Dark investigative theme





app.js



Tabs, uploads, demo analysis



License

Use freely for your university coursework and presentations.