# SmartCart — Clean UI and OCR Fixes

This revision fixes the hidden map SVG, replaces the empty-cart product illustration with a simple inline cart icon, and makes OCR setup alerts readable.

## Important: install into the same Python environment that runs SmartCart

The screenshot showed SmartCart running with this interpreter:

`/Users/mihiredeue/Downloads/.venv/bin/python`

The Tesseract desktop executable and the Python package `pytesseract` are two different things. Tesseract may be installed on macOS while the active virtual environment still lacks `pytesseract`.

Run this exact command in Terminal to install the missing Python package into the interpreter shown by SmartCart:

```bash
"/Users/mihiredeue/Downloads/.venv/bin/python" -m pip install pytesseract Pillow
```

Then restart SmartCart using that same interpreter:

```bash
cd ~/Downloads
SMARTCART_PORT=5050 "/Users/mihiredeue/Downloads/.venv/bin/python" -u SmartCart.py
```

If the project is in a different folder, change to the folder containing `SmartCart.py` first. Keep the interpreter path exactly as reported by the app.

## Install all project requirements

From the folder containing `requirements.txt`, run:

```bash
"/Users/mihiredeue/Downloads/.venv/bin/python" -m pip install -r requirements.txt
```

If Tesseract itself is installed somewhere other than `/opt/homebrew/bin/tesseract` or `/usr/local/bin/tesseract`, start the app with its full path:

```bash
TESSERACT_CMD="/full/path/to/tesseract" SMARTCART_PORT=5050 "/Users/mihiredeue/Downloads/.venv/bin/python" -u SmartCart.py
```

## Verify OCR

Open `http://127.0.0.1:5050/api/ocr/status`. The JSON response reports whether the Python packages and the Tesseract executable are available to the running process.

## Changes in this revision

- Map button SVG is explicitly displayed, overriding an earlier CSS rule that hid it.
- Empty cart uses a minimal inline shopping-cart icon, not a grocery photograph.
- Long OCR errors wrap inside a compact alert; missing `pytesseract` shows the exact environment-specific installation command.
- The welcome hero image remains locally embedded in the Python app, with a local SVG fallback.
- Cart begins empty; only supported product labels should be added through the recognition endpoint.

## Prototype limitations

The browser camera preview does not itself run YOLO. The separate trained YOLO inference process must send confirmed product detections to `/api/cart/recognized`. The five supported prototype classes are Banana, Kinder Joy, Maggi, Smoodh and SuperYou Chips.
