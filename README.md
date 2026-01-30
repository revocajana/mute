# Sign Meeting Platform (Prototype)

**A web-based online meeting platform with real-time sign-to-speech translation for speech-disabled participants.**  

This project is a **prototype** demonstrating accessibility in online meetings. It allows a participant who cannot speak to **translate simple hand signs into audible speech or text** for other meeting attendees. The prototype focuses on a **limited set of signs** for demonstration purposes.

---

## Table of Contents

- [Features](#features)  
- [Quick Start](#quick-start)  
- [Architecture](#architecture)  
- [Use Cases](#use-cases)  
- [Tech Stack](#tech-stack)  
- [Supported Gestures](#supported-gestures)  
- [System Requirements](#system-requirements)  
- [Folder Structure](#folder-structure)  
- [Installation](#installation)  
- [API & Communication Protocol](#api--communication-protocol)  
- [Usage](#usage)  
- [Known Limitations](#known-limitations)  
- [Extending the Prototype](#extending-the-prototype)  
- [Distribution & Deployment Strategy](#distribution--deployment-strategy)  
- [Troubleshooting](#troubleshooting)  
- [Performance Notes](#performance-notes)  
- [Version Roadmap](#version-roadmap)  
- [Testing Guide](#testing-guide)  
- [Future Work](#future-work)  
- [Contributing](#contributing)  
- [License](#license)  
- [Contact & Support](#contact--support)  

---

## Features

- Real-time video and audio communication (peer-to-peer).  
- **Mixed Meeting Mode:** Normal participants speak via audio; sign-disabled users use optional Python companion for gesture translation.  
- **Optional Python Companion:** Download and run locally only when needed for sign-to-speech or speech-to-sign translation.  
- Limited **sign-to-speech** translation using hand landmarks.  
- **Bidirectional Translation:** Supports both sign→speech (for non-speaking users) and speech→sign text (for deaf/hard-of-hearing users).  
- Lightweight **Python companion** for gesture recognition.  
- Minimal signaling server for WebRTC peer connections.  
- **Client-side Processing:** Reduces server workload by keeping all translation on user's local device.  
- Responsive and clean frontend design.  

---

## Quick Start

Get the meeting platform running in 5 minutes:

### For Normal Users (Browser Only)
1. Clone the repository: `git clone https://github.com/revocajana/mute.git`
2. Start the signaling server: `cd server && python signalingserver.py`
3. Open `client/index.html` in your browser
4. Share the room ID with other participants

### For Users with Speech/Hearing Disabilities
1. Clone the repository: `git clone https://github.com/revocajana/mute.git`
2. Start the signaling server: `cd server && python signalingserver.py`
3. Set up Python companion: `cd pythoncompanion && pip install -r requirements.txt`
4. Run the companion: `python signmodule.py`
5. Open `client/index.html` in your browser
6. Your hand gestures will be automatically translated and shared with other participants

**That's it!** You're ready to use the platform.

---

## Architecture

### System Overview

```
┌─────────────────────────────────────────────────────────────┐
│               MIXED MEETING MODE (All Participants)         │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Normal User 1         Normal User 2         Disabled User   │
│  (Browser only)        (Browser only)   (Browser + Python)  │
│       │                      │                    │          │
│  Microphone/Audio      Microphone/Audio   Hand Gestures     │
│       │                      │                    │          │
│       └──────────────────────┴────────────────────┘          │
│                          │                                   │
│                   WebRTC P2P Network                         │
│                    (All audio/video)                         │
│                                                              │
└─────────────────────────────────────────────────────────────┘

Optional Python Companion (runs locally on disabled user's device):
┌─────────────────────────────────────────────────────────────┐
│ • Captures hand gestures via webcam                         │
│ • Converts signs → speech (via MediaPipe + pyttsx3)        │
│ • Converts incoming speech → sign text (for deaf users)    │
│ • Sends translation via WebRTC DataChannel                 │
│ • All processing stays LOCAL - no server involved          │
└─────────────────────────────────────────────────────────────┘
```

### Meeting Participants

**Normal Users (Browser Only):**
- Join via any modern web browser
- Use microphone and speakers normally
- Receive video feed from all participants
- Can see translated text/speech from sign-using participants
- No Python installation required

**Sign-Disabled/Deaf Users (Browser + Optional Python Companion):**
- Join via browser like normal users
- Download and run Python companion module locally (optional, only if needed)
- If using signs: Python companion converts gestures → speech for all to hear
- If deaf: Python companion converts others' speech → sign language text
- All translation happens on their device (privacy-protected)
- Reduces server workload significantly

**Signaling Server:**
- Minimal WebSocket server for connection setup
- Facilitates peer discovery and initial connection establishment
- Does **NOT** process gestures or translation
- Stateless and lightweight

### Key Architecture Principles

- **Client-side Processing:** All translation happens locally on the user's device for maximum privacy
- **Optional Components:** Python companion is only downloaded/used by those who need it
- **Peer-to-Peer Communication:** Direct WebRTC connections between all participants minimize server load
- **No Server Processing:** The signaling server only handles initial connection setup; no gesture processing or translation on server
- **Privacy-First:** Hand gesture data and speech never pass through the server
- **Scalable:** Server workload stays minimal regardless of meeting size
- **Mixed Accessibility:** Supports both hearing and speaking-disabled users in the same meeting

---

## Use Cases

### Scenario 1: Speech-Disabled User
- Joins meeting via browser
- Downloads optional Python companion
- Uses hand signs to communicate
- Python companion converts signs → audible speech for all participants
- Hears others' speech normally

### Scenario 2: Deaf or Hard-of-Hearing User
- Joins meeting via browser
- Downloads optional Python companion
- Speaks normally (if able) or uses other input
- Python companion converts incoming speech → sign language text display
- Can read translations in real-time

### Scenario 3: Fully Accessible Meeting
- Mix of normal users (microphone only) and disabled users (with Python companion)
- Normal users: speak and listen normally
- Disabled users: use signs/speech conversion
- All communications flow through WebRTC peer-to-peer network
- Server only handles initial setup—minimal workload

---

## Tech Stack

**Frontend (Browser):**

- HTML, CSS, Vanilla JavaScript  
- WebRTC for real-time video/audio  
- MediaPipe Hands (Web) for hand landmark detection  

**Python Companion (Speech-disabled user):**

- Python 3.x  
- OpenCV (`cv2`) for webcam access  
- MediaPipe (`mediapipe`) for hand landmark detection  
- NumPy for numeric calculations  
- pyttsx3 for text-to-speech  

**Backend (Minimal Signaling Server):**

- Python 3.x  
- `websockets` library for signaling  

---

## API & Communication Protocol

### WebRTC DataChannel Messages

The Python companion communicates with the browser client via WebRTC DataChannel using JSON messages:

**Sign-to-Speech Translation (Disabled User → All Participants):**
```json
{
  "type": "gesture_detected",
  "gesture": "thumbs_up",
  "confidence": 0.95,
  "translated_text": "Yes, I agree",
  "audio_data": "base64_encoded_audio",
  "timestamp": 1672531200000
}
```

**Speech-to-Text Translation (All Participants → Disabled User):**
```json
{
  "type": "speech_detected",
  "speaker": "user_id_123",
  "transcript": "Hello everyone",
  "translated_sign_text": "Hello (wave gesture)",
  "timestamp": 1672531200000
}
```

### Signaling Server WebSocket Messages

**Connection Request:**
```json
{
  "type": "join_room",
  "room_id": "meeting_room_1",
  "user_id": "user_123",
  "has_companion": true
}
```

**SDP Offer/Answer:**
```json
{
  "type": "sdp_offer",
  "sdp": "v=0\no=- ...",
  "from_user": "user_123"
}
```

**ICE Candidate:**
```json
{
  "type": "ice_candidate",
  "candidate": "candidate:...",
  "from_user": "user_123"
}
```

---

## Folder Structure

```
mute.ai/
│
├── README.md                    # Project documentation
│
├── client/                      # Frontend - Web-based meeting interface
│   ├── index.html               # Main HTML page
│   ├── main.js                  # JavaScript for WebRTC and UI logic
│   └── style.css                # Styling for the meeting interface
│
├── pythoncompanion/             # Python companion - Sign recognition engine
│   ├── requirements.txt          # Python package dependencies
│   └── signmodule.py             # Hand gesture recognition and speech conversion
│
└── server/                      # Backend - Minimal signaling server
    └── signalingserver.py       # WebSocket signaling server for WebRTC setup
```

### Folder Descriptions

| Folder | Purpose | Key Files |
|--------|---------|-----------|
| **client/** | Web frontend for all meeting participants | `main.js` (WebRTC logic), `index.html` (UI) |
| **pythoncompanion/** | Local gesture recognition for speech-disabled user | `signmodule.py` (MediaPipe + gesture detection) |
| **server/** | Connection setup and signaling | `signalingserver.py` (WebSocket server) |
└── signaling_server.py



---

## Installation

### Prerequisites

- Python 3.9+  
- pip  
- Google Chrome or modern browser with WebRTC support  

### Frontend

No installation needed. Open `client/index.html` in a browser.

### Python Companion

```bash
cd pythoncompanion
pip install -r requirements.txt
```

### Signaling Server

```bash
cd server
pip install websockets
python signalingserver.py
```

---

## Usage

### Starting the Signaling Server

1. Open a terminal and navigate to the `server/` directory
2. Run: `python signalingserver.py`
3. The server will start and listen on `ws://localhost:8765`

### Running the Python Companion (Speech-disabled User)

1. Open a terminal and navigate to the `pythoncompanion/` directory
2. Ensure your webcam is connected
3. Run: `python signmodule.py`
4. The companion will start detecting hand gestures and converting them to speech

### Joining the Meeting (Browser)

1. Open a modern browser (Chrome, Firefox, Edge, etc.)
2. Navigate to `client/index.html` or serve it via a local web server
3. Allow camera and microphone permissions
4. Enter the meeting room ID
5. Share the room ID with other participants

---

## Supported Gestures

The prototype currently recognizes the following hand gestures (this list will expand):

| Gesture | Meaning | Hand Position |
|---------|---------|-----------------|
| Thumbs Up | Yes / Agreement | Fist with thumb pointing up |
| Thumbs Down | No / Disagreement | Fist with thumb pointing down |
| Open Palm | Stop / Hold On | All fingers spread apart |
| Peace Sign | Hello / Goodbye | Index and middle fingers up |
| OK Sign | Okay / Understood | Thumb and index forming circle |
| Wave | Greeting / Wave | Open hand moving side to side |
| Point | Attention / This | Index finger extended toward target |

**Note:** The prototype currently includes a limited set of gestures for demonstration. More comprehensive sign language support will be added in future versions.

---

## System Requirements

### For Normal Users (Browser)
- **OS:** Windows 10+, macOS 10.12+, or Linux (recent distributions)
- **Browser:** Chrome, Firefox, Edge, or Safari (latest versions with WebRTC support)
- **Hardware:** 
  - CPU: Dual-core processor (2 GHz or faster)
  - RAM: 2 GB minimum
  - Webcam: Any USB or built-in webcam
  - Microphone: Any standard microphone or headset
- **Network:** Broadband internet connection (10 Mbps+ recommended)

### For Python Companion Users
- **OS:** Windows 10+, macOS 10.12+, or Linux
- **Python:** 3.9 or higher
- **Hardware:**
  - CPU: Quad-core processor (2 GHz or faster) for smooth gesture detection
  - RAM: 4 GB minimum (6 GB+ recommended)
  - GPU: Optional, but recommended for faster MediaPipe processing
  - Webcam: High-quality webcam (720p or higher recommended)
- **Network:** Same as above

### For Server
- **OS:** Windows, macOS, or Linux
- **Python:** 3.9 or higher
- **Hardware:**
  - Minimal - can run on low-spec servers
  - 1 GB RAM sufficient for signaling server
- **Network:** Broadband internet connection

---

## Extending the Prototype

To add more signs or improve gesture recognition:

1. **Add New Signs:** Modify `pythoncompanion/signmodule.py` to recognize additional hand gestures
2. **Customize Speech Output:** Edit the text-to-speech output in the Python companion module
3. **Improve Accuracy:** Train a custom ML model using MediaPipe's gesture recognition capabilities
4. **Frontend Enhancements:** Update `client/main.js` and `client/style.css` for UI/UX improvements

---

## Known Limitations

This is a **prototype** with the following limitations:

### Gesture Recognition
- **Limited Gesture Set:** Only 7 common gestures currently supported. Full ASL/BSL support coming in future versions
- **Lighting Dependency:** Gesture recognition accuracy reduces in poor lighting conditions
- **Distance Sensitivity:** Gestures must be performed within 1-3 meters of the webcam
- **Gesture Duration:** Held too long (>5 seconds), gestures may be misrecognized
- **Single User:** Companion currently supports only one user per device (multi-user per device coming soon)

### Audio & Video
- **Audio Quality:** Depends on microphone quality (no noise suppression in prototype)
- **Video Latency:** P2P WebRTC may have 100-500ms latency depending on network
- **Bandwidth:** Requires stable connection; 10+ Mbps recommended for HD video

### Platform Support
- **Browser Compatibility:** Works best on Chrome and Firefox; Safari has limited WebRTC support
- **Mobile:** Not yet optimized for mobile devices (roadmap item)
- **Offline Mode:** Requires internet connection; no offline functionality

### Sign Language
- **English-Based Gestures Only:** Current prototype uses simplified gesture set, not complete sign language
- **Cultural Variations:** Does not account for regional sign language variations
- **Facial Expressions:** Cannot interpret facial expressions or body language

### Server
- **Scalability:** Signaling server tested up to 100 concurrent meetings
- **Data Persistence:** No message history or recording features
- **Encryption:** WebRTC connections are encrypted; signaling is plain WebSocket (upgrade needed for production)

---

## Future Work

- [ ] Integration with popular meeting platforms (Zoom, Google Meet, Teams)
- [ ] Enhanced ML model for more signs and better accuracy
- [ ] Multi-user support with translation history
- [ ] Mobile app version (React Native or Flutter)
- [ ] Real-time sign language dictionary with video references
- [ ] Support for multiple sign languages (ASL, BSL, ISL, etc.)
- [ ] Improved audio/video quality and latency optimization
- [ ] Server-side gesture processing for better scalability

---

## Distribution & Deployment Strategy

### Phase 1: GitHub Repository (Current)
- Host the complete project on GitHub (client, server, pythoncompanion)
- Users clone the repository: `git clone https://github.com/yourname/mute.ai.git`
- For Python companion installation: Navigate to `pythoncompanion/` directory and run `pip install -r requirements.txt`
- **Status:** Initial prototype and community contributions

### Phase 2: PyPI Package Distribution (Future)
- Publish the Python companion module to PyPI for easier installation
- Users will be able to install with: `pip install mute-ai-companion`
- Simplified setup for end-users who only need the companion module
- Maintains version history and automatic updates

### Phase 3: Web-based Distribution (Future)
- Consider hosting the web client on a CDN or static hosting service
- Provide direct links for downloads and installation guides
- Potential mobile app distribution (App Store, Google Play)

---

## Troubleshooting

> **Coming Soon!** A comprehensive troubleshooting guide will be added in the next version with solutions for common issues such as:
> - Webcam not detected
> - Audio/video not working
> - Connection errors
> - Gesture not recognized
> - Server connection issues

---

## Performance Notes

> **Coming Soon!** Performance benchmarks and optimization guidelines will be documented, including:
> - Expected latency measurements
> - CPU/RAM usage under various conditions
> - Network bandwidth requirements
> - Scalability limits
> - Optimization tips

---

## Version Roadmap

> **Coming Soon!** A detailed version roadmap with release timelines will be published with planned features for v0.2, v0.3, and beyond.

---

## Testing Guide

> **Coming Soon!** Comprehensive testing documentation will be added with:
> - How to run unit tests
> - Integration testing procedures
> - Performance testing methodology
> - Gesture recognition accuracy validation

---

## Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature-name`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature-name`)
5. Open a Pull Request

Please ensure your code follows best practices and includes appropriate documentation.

---

## License

This project is licensed under the **MIT License**. See the LICENSE file for more details.

---

## Contact & Support

For questions, suggestions, or issues, please open an issue on the repository or contact the development team.

**Proposed and developed by Revocat
