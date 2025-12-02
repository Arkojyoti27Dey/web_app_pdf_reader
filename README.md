live link:-
https://arkojyoti27dey.github.io/web_app_pdf_reader/

PDFVoice Canvas - Technical Documentation

PDFVoice Canvas is a single-file, client-side web application that renders PDF documents using HTML5 Canvas and reads them aloud using the browser's native Web Speech API. It features paragraph-aware navigation, text highlighting, and dynamic reading speed control.

🚀 Key Features

Secure Client-Side Processing: Files are processed locally in the browser; no data is sent to a server.

High-Fidelity Rendering: Uses PDF.js to render vector-based PDFs onto an HTML5 Canvas.

Text-to-Speech (TTS): Reads text aloud using the SpeechSynthesis API with auto-language detection (en-US/en-GB preferred).

Paragraph Navigation: Automatically groups text lines into paragraphs and allows skipping forward/backward.

Synchronized Highlighting: visually highlights the paragraph currently being read.

Continuous Reading: Automatically turns the page when the reader reaches the end of the current page.

Dynamic Speed Control: Adjust reading speed on the fly without losing position.

🛠️ Tech Stack & Libraries

Core: HTML5, Vanilla JavaScript (ES6+).

PDF Engine: PDF.js (v3.11.174)

Used for parsing the raw binary PDF data.

Renders pages to <canvas> context.

Extracts text content and coordinates for the invisible text layer.

Styling: Tailwind CSS (via CDN) for a responsive, utility-first UI.

Icons: Phosphor Icons for modern, consistent UI iconography.

🧠 "How It Works" - In-Depth Logic

1. The Rendering Pipeline

The app uses a hybrid approach for displaying content to ensure both visual fidelity and text selectability.

Canvas Layer: The PDF page is rendered as a bitmap image onto a <canvas> element using page.render(). This ensures fonts, images, and vectors look exactly as intended.

Text Layer: An invisible HTML layer is placed exactly on top of the canvas. pdf.js provides renderTextLayer(), which places transparent <span> elements over the text in the canvas. This allows users to drag-to-select text as if it were a normal web page.

2. Paragraph Detection Algorithm

PDFs don't natively understand "paragraphs"; they only understand "lines of text" (and sometimes just individual characters) with XY coordinates. To enable "Next Paragraph" navigation, we implemented a heuristic algorithm:

Extraction: We iterate through every text item extracted from the page.

Coordinate Transformation: We convert internal PDF coordinates to viewport pixels.

Heuristic: We calculate the vertical gap (diff) between the current line's Y-position and the previous line's Y-position.

Logic: if (diff > textHeight * 2.5) -> Treat as a new paragraph.

Bounding Box: We calculate a bounding box (minX, minY, maxX, maxY) for the group of lines to draw the yellow highlight box.

3. Audio & Speed Control Logic

The Web Speech API has a limitation: you cannot change the .rate (speed) of an utterance while it is speaking. To solve this, we implemented a State Preservation & Restart mechanism:

State Tracking: Global variables track currentTextBeingSpoken and currentParagraphIndex.

Debouncing: A debounce timer (300ms) prevents the audio from restarting rapidly while dragging the slider.

Hot Swapping: When the speed changes:

We flag isChangingSpeed = true.

We call synth.cancel() to stop the current voice.

We immediately create a new SpeechSynthesisUtterance with the preserved text and the new rate.

The onend event listener checks the flag; if we are just changing speed, it ignores the "end of speech" logic so it doesn't accidentally skip to the next paragraph.

4. Continuous Page Turning

To allow hands-free reading:

When the speakParagraph function reaches the last index of the array (index >= paragraphs.length), it checks if there are more pages.

If yes, it sets a flag shouldAutoPlayNextPage = true.

It calls onNextPage(), which triggers the async PDF rendering.

Once the new page render completes, the renderPage promise resolves, checks the flag, and automatically triggers speakParagraph(0) to start reading the new top paragraph.

📂 File Structure

The application is contained in a single file (pdf_reader.html) for portability.

<head>: CDNs for Tailwind, PDF.js, and Icons. CSS for the invisible Text Layer.

<body>:

Header: Logo and File Upload.

Controls Bar: Speed slider, Navigation buttons, Play/Pause.

Main: The Canvas wrapper and Status messages.

<script>:

renderPage(): Main loop for PDF processing.

processTextContent(): The paragraph detection heuristic.

speakText() / speakParagraph(): The core TTS engine wrapper.

playAudio(): The entry point that decides whether to read a user selection or auto-read paragraphs.

🚀 How to Run

Download the pdf_reader.html file.

Open it in any modern browser (Chrome, Edge, Safari, Firefox).


Upload a PDF and click Play.
