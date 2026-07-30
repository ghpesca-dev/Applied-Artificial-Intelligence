# Web AI Demo - Multimodal

A front-end demo of Chrome's built-in AI APIs (Gemini Nano) showing multimodal prompting — text, image, and audio input — plus on-device translation, entirely in the browser with no server or API key.

## How it works

1. On load, the app checks that `LanguageModel`, `Translator`, and `LanguageDetector` are available (prompting flag/download instructions if not) and initializes the translation and language-detection sessions.
2. It reads the model's default/min/max `temperature` and `topK` from `LanguageModel.params()` and configures the sliders/inputs accordingly.
3. The user asks a question in English and, optionally, attaches an image or an English-language audio file. A live preview of the attached file is rendered.
4. Submitting creates a `LanguageModel` session accepting text, image, and audio input, streaming the English response token-by-token (`session.promptStreaming`).
5. Once the full response arrives, it's run through the on-device `Translator` (auto-detecting language via `LanguageDetector` first, skipping translation if already Portuguese) and the Portuguese translation replaces the output.
6. Submitting again while a response is streaming aborts the in-flight generation (`AbortController`) and starts a fresh session.

## Requirements

- Google Chrome (or Chrome Canary), recent version
- The following flags enabled at `chrome://flags/`:
  - `Prompt API for Gemini Nano` (`#prompt-api-for-gemini-nano`)
  - `Translation API` (`#translation-api`)
  - `Language Detection API` (`#language-detector-api`)
- The on-device models downloaded (the page prompts for this automatically if needed)

## Project Structure

- `index.html` - UI: temperature/topK controls, file attachment, question form, output area
- `index.js` - Entry point; wires up services, view, and controller
- `services/aiService.js` - Requirements/availability checks, session/parameter management, and multimodal streaming prompt logic
- `services/translationService.js` - On-device language detection and translation to Portuguese
- `controllers/formController.js` - Connects view events to the AI and translation services
- `views/view.js` - DOM rendering: parameter displays, file preview, output, button state
- `style.css` - Styling

## Setup and Run

1. Install dependencies:
```
npm install
```

2. Start the application:
```
npm start
```

3. Open the printed local URL in Chrome.

## Features

- Multimodal prompts: text plus an attached image or audio file
- Live adjustment of `temperature` (creativity) and `topK` (vocabulary breadth) per request
- Streamed English responses, automatically translated to Portuguese on-device
- File attachment preview (image thumbnail / audio player) with removal
- Ability to stop generation mid-stream
- Automatic on-device model availability/download checks with user-facing status messages
