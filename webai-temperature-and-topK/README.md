# Web AI Demo - Temperature & Top-K

A minimal front-end demo of Chrome's built-in **Prompt API** (`window.LanguageModel`, backed by Gemini Nano), showing how the `temperature` and `topK` sampling parameters affect the model's generated text — entirely on-device, no server or API key involved.

## How it works

1. On load, the page checks that the browser exposes `LanguageModel` and that the on-device model is available (prompting a download via `chrome://flags` if needed).
2. It reads the model's default/min/max `temperature` and `topK` from `LanguageModel.params()` and uses them to configure the sliders/inputs.
3. Typing a question and submitting creates a `LanguageModel` session with the chosen `temperature`/`topK` and streams the response token-by-token (`session.promptStreaming`) into the page.
4. Submitting again while a response is streaming aborts the in-flight generation (`AbortController`) and starts a fresh session with the current parameter values.

## Requirements

- Google Chrome (or Chrome Canary), recent version
- The `Prompt API for Gemini Nano` flag enabled at `chrome://flags/#prompt-api-for-gemini-nano`
- The on-device model downloaded (the page will prompt for this automatically if needed)

## Project Structure

- `index.html` - UI: temperature/topK controls, question form, output area
- `index.js` - Model availability checks, session/parameter management, and streaming prompt logic
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

- Live adjustment of `temperature` (creativity) and `topK` (vocabulary breadth) per request
- Streamed responses with the ability to stop generation mid-stream
- Automatic on-device model availability/download checks with user-facing status messages
