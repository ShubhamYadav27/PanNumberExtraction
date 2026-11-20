# PAN Number Extraction Using OpenCV and Tesseract OCR

<p align="center">

**An Image Processing and Optical Character Recognition (OCR) Pipeline for PAN Card Number Extraction**

## 1. Project Overview

**PAN Number Extraction** is a Python-based computer vision and Optical Character Recognition (OCR) project designed to identify and extract Permanent Account Number (PAN) candidates from PAN card images.

The system combines traditional image processing techniques, morphological operations, contour-based region detection, Tesseract OCR, and regular-expression pattern matching to locate and extract text matching the standard 10-character PAN format.

The extracted candidates are exported into an Excel spreadsheet for further inspection and processing.

The project provides two execution paths:

- **Python CLI implementation:** Processes images from a user-provided directory using command-line arguments.
- **Jupyter Notebook implementation:** Provides an interactive, notebook-based workflow, originally configured for Google Colab.

### Key Capabilities

- Batch processing of images from a directory.
- Image resizing and grayscale conversion.
- Noise reduction using Gaussian blurring.
- Text-region enhancement using morphological operations.
- Horizontal gradient analysis using the Sobel operator.
- Candidate text-region localization using contour detection.
- OCR-based text recognition using Tesseract.
- PAN-format candidate extraction using regular expressions.
- Excel-based result export.
- Visual inspection of detected regions in the Python CLI implementation.

---

## 2. Problem Statement

Extracting structured information from identity-document images can be challenging because of variations in image quality, text positioning, lighting, resolution, and background complexity.

Manual extraction is repetitive and does not scale efficiently when processing multiple images.

This project explores an automated approach that combines computer vision and OCR to:

1. Process input PAN card images.
2. Enhance relevant text regions.
3. Identify potential text-containing regions.
4. Recognize characters using OCR.
5. Extract candidates matching the PAN format.
6. Store the extracted information in an Excel workbook.

The implementation is based on classical computer vision techniques rather than a trained deep learning detection model.

---

## 3. System Architecture

The processing pipeline follows these stages:

```text
                 INPUT
                   |
                   v
          PAN Card Image Directory
                   |
                   v
           Image Loading (OpenCV)
                   |
                   v
              Image Resize
                   |
                   v
            Grayscale Conversion
                   |
                   v
             Gaussian Blur
                   |
                   v
           Black-Hat Morphology
                   |
                   v
          Sobel X-Gradient Analysis
                   |
                   v
           Gradient Normalization
                   |
                   v
          Morphological Closing
                   |
                   v
            Otsu Thresholding
                   |
                   v
        Morphological Closing + Erosion
                   |
                   v
            Contour Detection
                   |
                   v
        Candidate ROI Extraction
                   |
                   v
             Tesseract OCR
                   |
                   v
         Regular Expression Matching
                   |
                   v
           PAN Candidate Extraction
                   |
                   v
             Excel Export
```

---

## 4. Technology Stack

| Technology | Purpose |
|---|---|
| Python | Core programming language |
| OpenCV 4 | Image processing and computer vision |
| NumPy | Numerical operations and gradient normalization |
| Tesseract OCR | Optical character recognition |
| Pytesseract | Python interface for Tesseract |
| Pillow | Image loading for OCR |
| imutils | Image resizing, directory traversal, and contour compatibility |
| XlsxWriter | Excel workbook generation |
| Regular Expressions | PAN-format candidate extraction |
| Jupyter Notebook | Interactive execution environment |

### External System Dependency

Tesseract OCR is an external executable dependency. Installing the Python `pytesseract` package alone does not install the Tesseract OCR engine.

---

## 5. Core Algorithm and Implementation

### 5.1 Image Acquisition

The CLI implementation accepts an image directory through the `--images` command-line argument.

The `imutils.paths.list_images()` utility iterates over images in the supplied directory.

The notebook implementation uses:

```python
images_path = "/content/images"
```

This path is configured for the original Google Colab workflow.

### 5.2 Image Resizing

Images are resized to a fixed height of 600 pixels using `imutils.resize()`.

This provides a more consistent processing scale.

### 5.3 Grayscale Conversion

The color image is converted to grayscale using:

```python
cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
```

Grayscale conversion reduces the processing complexity by removing unnecessary color-channel information.

### 5.4 Gaussian Blurring

A Gaussian blur with a 3 × 3 kernel is applied:

```python
cv2.GaussianBlur(gray, (3, 3), 0)
```

This reduces small image variations and noise before morphological processing.

### 5.5 Black-Hat Morphological Transformation

The black-hat transformation highlights dark structures against lighter backgrounds.

The implementation uses a rectangular structuring element of size 13 × 5:

```python
rectKernel = cv2.getStructuringElement(
    cv2.MORPH_RECT, (13, 5)
)

blackhat = cv2.morphologyEx(
    gray, cv2.MORPH_BLACKHAT, rectKernel
)
```

This operation helps emphasize text-like structures.

### 5.6 Sobel X-Gradient

A horizontal image gradient is calculated using the Sobel operator:

```python
gradX = cv2.Sobel(
    blackhat,
    ddepth=cv2.CV_32F,
    dx=1,
    dy=0,
    ksize=-1
)
```

The absolute gradient values are normalized to the 0–255 intensity range.

This emphasizes horizontal intensity transitions associated with text regions.

### 5.7 Thresholding and Morphological Processing

The normalized gradient undergoes:

- Morphological closing using the rectangular kernel.
- Otsu binary thresholding.
- Additional morphological closing using a 21 × 21 kernel.
- Four iterations of erosion.

The notebook and CLI implementations use these operations to connect relevant image structures and isolate potential text-containing regions.

### 5.8 Contour Detection

External contours are identified using:

```python
cv2.findContours(
    thresh.copy(),
    cv2.RETR_EXTERNAL,
    cv2.CHAIN_APPROX_SIMPLE
)
```

Contours are sorted in descending order of contour area.

Bounding rectangles are calculated to identify candidate regions.

### 5.9 Region of Interest Extraction

The detected bounding rectangle is expanded by approximately 3% padding based on its dimensions.

The resulting Region of Interest (ROI) is extracted from the original resized image.

This ROI is passed to the OCR engine.

### 5.10 Optical Character Recognition

The project uses:

```python
pytesseract.image_to_string()
```

The notebook explicitly uses:

```python
config="--psm 6"
```

Page segmentation mode 6 assumes a uniform block of text.

The CLI implementation uses the default Tesseract configuration.

### 5.11 Regular Expression Matching

The recognized OCR text is filtered using:

```python
r"[A-Z]{5}\d{4}[A-Z]"
```

The expression matches a 10-character string containing:

- Five uppercase alphabetic characters.
- Four numeric digits.
- One uppercase alphabetic character.

This extracts candidates that conform to the expected PAN format.

**Important:** Pattern matching establishes format compatibility only. It does not establish that a PAN number is genuine, active, or associated with a particular individual.

### 5.12 Excel Export

The project uses XlsxWriter to create an Excel workbook.

The implementation writes:

- The image processing index.
- The extracted PAN-format candidate.

The notebook saves its workbook at:

```text
/content/example.xlsx
```

The CLI implementation currently uses a hardcoded output path that must be adjusted for the local environment.

---

## 6. Repository Structure

```text
PanNumberExtraction/
│
├── Pan_Number_Extraction.ipynb
│   └── Interactive notebook implementation
│
├── pan.py
│   └── Command-line Python implementation
│
├── p1.jpg
├── p5.jpg
├── p7.jpg
├── p8.jpg
├── p10.jpg
├── p11.jpg
├── p12.jpg
│   └── Sample image assets
│
├── pan.mov
│   └── Project demonstration video
│
├── README.md
│   └── Project documentation
│
└── LICENSE
    └── MIT License
```

---

## 7. Installation and Environment Setup

### Prerequisites

- Python environment.
- pip package manager.
- Tesseract OCR engine.
- Access to the input image directory.

The original project documentation specifies Python 3.7 and OpenCV 4. The dependency versions are not pinned in the repository.

For a modern setup, use a compatible Python environment and verify package compatibility.

### Step 1: Clone the Repository

```bash
git clone https://github.com/ShubhamYadav27/PanNumberExtraction.git

cd PanNumberExtraction
```

### Step 2: Create a Virtual Environment

Windows:

```powershell
python -m venv venv

venv\Scripts\activate
```

Linux/macOS:

```bash
python3 -m venv venv

source venv/bin/activate
```

### Step 3: Install Python Dependencies

```bash
pip install numpy opencv-python imutils pytesseract pillow xlsxwriter
```

### Step 4: Install Tesseract OCR

Install the Tesseract OCR engine separately.

- Windows: Install a compatible Tesseract OCR distribution and configure its executable path.
- Ubuntu/Debian:

```bash
sudo apt update

sudo apt install tesseract-ocr
```

- macOS:

```bash
brew install tesseract
```

Verify installation:

```bash
tesseract --version
```

If Tesseract is not available in the system PATH, configure its executable location in Python:

```python
pytesseract.pytesseract.tesseract_cmd = (
    r"C:\Program Files\Tesseract-OCR\tesseract.exe"
)
```

Adjust the path to match your installation.

---

## 8. Running the Application

### Option A: Python CLI

The CLI accepts the image directory as a required argument.

```bash
python pan.py --images ./images
```

Equivalent short argument:

```bash
python pan.py -i ./images
```

The directory should contain the input images.

**Configuration note:** Before execution, update the hardcoded Excel output path in `pan.py`:

```python
workbook = xlsxwriter.Workbook(
    r'/Users/vd/Downloads/PAN Number extraction/example.xlsx'
)
```

Replace it with a valid output location on your machine.

The CLI implementation also opens OpenCV visualization windows and waits for keyboard input during image processing.

### Option B: Jupyter Notebook

Start Jupyter:

```bash
pip install notebook

jupyter notebook
```

Open:

```text
Pan_Number_Extraction.ipynb
```

For Google Colab:

1. Upload the notebook to Google Colab.
2. Install the required Python packages.
3. Ensure the Tesseract executable is installed in the runtime.
4. Upload or mount the input images.
5. Configure the notebook image directory.
6. Execute the notebook cells sequentially.

The original notebook uses `/content/images` as its image directory and `/content/example.xlsx` as its Excel output path.

---

## 9. Output Format

The Excel workbook contains the following logical fields:

| Column | Description |
|---|---|
| Image Index | Sequential index assigned during image processing |
| Extracted PAN Candidate | Text matching the PAN-format regular expression |

The workbook is generated using the XlsxWriter library.

The output is intended for inspection and downstream processing rather than official PAN verification.

---

## 10. Technical Design Decisions

| Component | Design decision | Rationale |
|---|---|---|
| Image scaling | Fixed-height resizing | Standardizes image dimensions |
| Noise reduction | Gaussian blur | Reduces small image variations |
| Text enhancement | Black-hat morphology | Highlights dark structures |
| Feature extraction | Sobel X-gradient | Emphasizes horizontal transitions |
| Segmentation | Otsu thresholding | Automatically determines a binary threshold |
| Region localization | External contours | Identifies candidate image regions |
| Text recognition | Tesseract OCR | Converts image text into machine-readable text |
| Candidate filtering | Regex | Filters OCR output using the expected PAN format |
| Result persistence | Excel workbook | Enables structured inspection and processing |

---

## 11. Current Implementation Limitations

The repository represents an experimental computer vision and OCR implementation. The following limitations should be considered when using it.

### Input Validation

- Image loading failures are not explicitly handled.
- Empty or invalid input directories do not have dedicated error handling.
- Image dimension and file-format validation are not implemented.

### Computer Vision

- The pipeline relies on fixed morphological kernel sizes.
- Fixed image resizing may affect images with substantially different aspect ratios.
- Contour-based detection can identify irrelevant regions.
- ROI boundary clipping is not explicitly implemented.
- Gradient normalization does not explicitly handle a zero intensity range.

### OCR and Extraction

- OCR accuracy depends on image quality and text clarity.
- The implementation does not include OCR confidence thresholds.
- The regular expression validates only the expected character format.
- OCR character substitutions can produce false candidates.
- The CLI and notebook use different Tesseract configurations.

### Result Management

- The CLI uses a machine-specific output path.
- Multiple candidate contours can overwrite values in the same image row.
- The CLI uses a condition that does not properly distinguish empty extraction results.
- The implementation does not provide structured extraction logs or detailed processing reports.

These limitations are documented to distinguish the existing implementation from potential future improvements.

---

## 12. Security and Privacy Considerations

PAN numbers are sensitive personal identifiers.

When using this project:

- Process only images for which you have appropriate authorization.
- Avoid uploading real PAN-card images to public repositories.
- Do not publish extracted PAN numbers in logs, screenshots, or demonstrations.
- Restrict access to generated Excel workbooks.
- Store temporary images and generated outputs securely.
- Delete temporary files and sensitive data when no longer required.
- Use synthetic or appropriately redacted images for public demonstrations.

This repository should not be considered a production-grade identity verification system.

---

## 13. Potential Future Enhancements

The following are possible future engineering improvements and are not claimed as existing features.

- Configurable input and output paths.
- Improved image validation and exception handling.
- ROI coordinate clipping and validation.
- Contour filtering using aspect ratio, area, and geometric constraints.
- OCR confidence scoring and candidate ranking.
- Character normalization and OCR error correction.
- Support for multiple image formats and more challenging image conditions.
- Structured CSV and JSON exports.
- Duplicate detection and result deduplication.
- Automated unit and integration testing.
- Secure handling of sensitive extracted information.
- Modular architecture separating preprocessing, detection, OCR, validation, and export.

---

## 14. License

This project is distributed under the MIT License.

Refer to the `LICENSE` file for complete licensing terms.

---

## 15. Demonstration

A demonstration video is available in the repository:

...

Sample image assets are also included for understanding the project's image-processing workflow.

---

## 16. Author

**Shubham Yadav**

GitHub: https://github.com/ShubhamYadav27

Project Repository: https://github.com/ShubhamYadav27/PanNumberExtraction

---

*PAN Number Extraction — Classical Computer Vision, Image Processing and OCR-based Information Extraction.*

