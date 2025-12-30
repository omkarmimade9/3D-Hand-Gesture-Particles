# 3D Hand Gesture Particle System

## Overview
A real-time interactive 3D particle system built with Three.js and MediaPipe's HandLandmarker for hand gesture tracking. The system dynamically controls particle behavior including expansion, color changes, and switching between multiple particle templates.

## Features
- **Real-time Hand Tracking**: Uses MediaPipe's HandLandmarker to detect hand landmarks
- **Dynamic Particle System**: 3000 particles that respond to hand gestures
- **Gesture Control**: Pinch to contract, open hand to expand particles
- **3D Rotation**: Particles rotate based on hand position
- **Saturn Shape Template**: Current implementation features a Saturn-like shape with core sphere and rings
- **WebGL Rendering**: Uses Three.js for high-performance 3D graphics

## Code Analysis - Issues Found

### ✅ **No Critical Errors**
The code has been analyzed and is **syntactically correct**. All major components are properly structured.

### ⚠️ **Potential Issues & Recommendations**

#### 1. **Missing Window Resize Handler**
**Issue**: The renderer size is set once but never updates on window resize.
**Fix**: Add this code after renderer setup:
```javascript
window.addEventListener('resize', () => {
    const width = window.innerWidth;
    const height = window.innerHeight;
    camera.aspect = width / height;
    camera.updateProjectionMatrix();
    renderer.setSize(width, height);
});
```

#### 2. **Random Rings Issue in getSaturnPoints()**
**Issue**: The `else` block generates random positions every frame for ring particles, causing them to "jitter" unpredictably.
**Fix**: Pre-generate ring positions or use Fibonacci sphere distribution for consistency:
```javascript
else {
    const phi = Math.acos(-1 + (2 * (i - PARTICLE_COUNT * 0.6)) / (PARTICLE_COUNT * 0.4));
    const theta = Math.sqrt(PARTICLE_COUNT * 0.4 * Math.PI) * phi;
    const radius = 8 + (Math.sin(theta) * 2);
    return new THREE.Vector3(Math.cos(theta) * radius, Math.cos(phi), Math.sin(theta) * radius);
}
```

#### 3. **Unused Variable**
**Issue**: `targetPositions` is declared but never used (line with `const targetPositions = ...`).
**Fix**: Remove unused variable or implement proper target position tracking.

#### 4. **MediaPipe Model Loading Delays**
**Issue**: The model loads asynchronously, which may cause console errors if `detectForVideo` is called before loading completes.
**Recommendation**: Already handled with checks, but consider adding error handling:
```javascript
const createHandLandmarker = async () => {
    try {
        const vision = await FilesetResolver.forVisionTasks(...);
        handLandmarker = await HandLandmarker.createFromOptions(...);
        startWebcam();
    } catch (error) {
        console.error('Failed to load HandLandmarker:', error);
    }
};
```

#### 5. **Camera Permission May Fail**
**Issue**: If user denies camera access, `navigator.mediaDevices.getUserMedia` will throw an error.
**Fix**: Add error handling:
```javascript
const startWebcam = async () => {
    try {
        const video = document.getElementById("webcam");
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        video.srcObject = stream;
        video.onloadedmetadata = () => {
            webcamRunning = true;
        };
    } catch (error) {
        console.error('Camera access denied:', error);
        alert('Please allow camera access to use this feature.');
    }
};
```

#### 6. **Performance Optimization**
**Issue**: Recalculating `getSaturnPoints(i)` every frame for all 3000 particles is expensive.
**Recommendation**: Consider caching positions or using GPU-based calculations for large particle counts.

#### 7. **HTTPS Requirement**
**Note**: MediaPipe and camera access require HTTPS or localhost. File protocol (`file://`) will NOT work.

## Running the Project

### Requirements
- Modern browser with WebGL support
- Camera access (HTTPS or localhost)
- Recent versions of Chrome, Firefox, Safari, or Edge

### Local Testing
1. Use Python HTTP server (from project directory):
   ```bash
   python -m http.server 8000
   ```
2. Open `http://localhost:8000` in your browser
3. Grant camera permission when prompted

### Deployment
For production, ensure:
- Site is served over HTTPS
- Appropriate CORS headers are set
- Modern browser support is available

## External Dependencies
- **Three.js** (`v0.160.0`) - 3D Graphics Library
- **MediaPipe Tasks Vision** (`v0.10.0`) - Hand Detection Model
- **Google MediaPipe Models** - Pre-trained hand landmarker model

## Browser Compatibility
- ✅ Chrome 90+
- ✅ Firefox 88+
- ✅ Safari 14+
- ✅ Edge 90+

## Future Enhancements
1. Add more particle templates (hearts, flowers, fireworks)
2. Implement color changing based on hand position
3. Multi-hand support
4. Gesture recognition for shape switching
5. Performance optimization for mobile devices
6. Audio reactivity

## License
Open source - Feel free to modify and use!
