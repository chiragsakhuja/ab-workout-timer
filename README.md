# Hansel's Ab Workout Timer

A Vue.js single-page webapp designed for mobile devices to guide through a structured ab workout routine. The app provides audio and visual feedback throughout the workout and keeps the screen awake during the session.

## Features

- **3-Round Workout**: Automatically guides through 3 rounds of the complete ab routine
- **Audio Feedback**: Distinct sounds for:
  - Exercise start (high-pitched double beep)
  - 5-second warning (medium-pitched single beep)
  - Exercise end (low-pitched long beep)
  - Workout completion (celebratory sound sequence)
- **Visual Feedback**: Color-coded timer display and progress bar
- **Screen Wake Lock**: Keeps the screen awake during the entire workout
- **Mobile Optimized**: Responsive design for mobile devices in portrait and landscape modes
- **Pause/Resume**: Control workout flow as needed

## Workout Routine (Repeated 3 Times)

1. **Side Plank (Right)** - 45s
2. **Rest** - 15s
3. **Side Plank (Left)** - 45s
4. **Rest** - 15s
5. **Plank - Straight Arms Tap Forward** - 45s
6. **Front Plank on Forearms** - 45s (no break between exercises 5 & 6)
7. **Rest** - 10s
8. **Russian Twists** - 20s
9. **Rest** - 15s
10. **Dead Bug** - 45s
11. **Rest** - 60s

## Setup & Installation

1. **Install dependencies:**
   ```bash
   npm install
   ```

2. **Start the development server:**
   ```bash
   npm run dev
   ```

3. **Access the app:**
   Open your browser to `http://localhost:3000`

## Building for Production

```bash
npm run build
```

The built files will be in the `dist` folder.

## Usage

1. **Start**: Tap "Start Workout" to begin the first round
2. **Controls**: During workout, use Pause/Resume and Reset buttons as needed
3. **Audio**: Ensure device volume is on for audio cues
4. **Screen**: The app will automatically keep your screen awake during the workout
5. **Completion**: After 3 rounds, the app will show a completion message

## Browser Compatibility

- Requires a modern browser with support for:
  - Web Audio API (for sound generation)
  - Screen Wake Lock API (for keeping screen awake)
  - ES6+ features

## Mobile Usage

For best experience on mobile:
- Add to home screen for full-screen app experience
- Ensure device is not in silent mode for audio cues
- Grant necessary permissions when prompted (wake lock, audio)

## Technologies Used

- Vue.js 3
- Vite (build tool)
- Web Audio API (for sound generation)
- Screen Wake Lock API (for screen management)
- CSS3 (for responsive design and animations) 