# PDF Table Extractor & OCR Converter (PaddleOCR)

## Project Overview

This project is an on-device solution for automatically detecting and extracting tables from PDF documents, then converting them to HTML format using PaddleOCR's PP-Structure model. The system operates completely offline without requiring external APIs.

## Core Objectives

- **Complete offline operation**: No external API dependencies
- **Automatic table detection**: Identify table regions in PDF documents
- **OCR processing**: Handle image-based tables with OCR
- **Structure preservation**: Maintain table structure when converting to HTML
- **Simple CLI interface**: Easy-to-use command-line tool

## Technical Stack

### PDF Processing
- **pdfplumber**: Text extraction from PDFs
- **PyMuPDF (fitz)**: PDF image rendering

### OCR Engine
- **PaddleOCR PP-Structure**: Table-specific OCR with SLANet model
- Optimized for table structure recognition
- Model size: ~10-20MB (lightweight)

### Image Processing
- **OpenCV**: Image manipulation and preprocessing
- **Pillow**: Image I/O and format conversion

### Output Format
- **HTML/CSS**: Structured table output

## Key Features

1. **Table-Specialized Processing**
   - PP-Structure's SLANet model for accurate table structure recognition
   - Handles complex table layouts and nested structures

2. **Korean Language Support**
   - Simultaneous Korean and English text recognition
   - Optimized for bilingual documents

3. **CPU Operation**
   - Sufficient performance without GPU
   - Accessible on standard hardware

4. **Lightweight Design**
   - Small model footprint (~10-20MB)
   - Minimal dependencies

## MVP Scope

### Phase 1: Core Functionality
1. Accept PDF file input
2. Extract text-based tables using pdfplumber (priority)
3. Fallback to image conversion + PaddleOCR on extraction failure
4. Convert extracted data to HTML tables
5. Save output to files
6. Provide basic CLI interface

### Processing Pipeline
```
PDF Input
    ↓
Text-based extraction (pdfplumber)
    ↓ (if fails)
Convert to image (PyMuPDF)
    ↓
OCR processing (PaddleOCR PP-Structure)
    ↓
Structure analysis
    ↓
HTML table generation
    ↓
Output file
```

## Post-MVP Features (Future)

The following features are explicitly excluded from MVP and planned for later iterations:
- GUI interface
- Complex merged cell handling
- Batch processing (multiple PDFs simultaneously)
- GPU optimization
- Real-time processing capabilities

## Development Guidelines

### Code Structure
- Modular design with clear separation of concerns
- PDF processing module
- OCR processing module
- HTML generation module
- CLI interface module

### Testing Strategy
- Unit tests for each module
- Integration tests for end-to-end pipeline
- Test with various PDF formats (text-based, image-based, mixed)
- Korean and English text validation

### Performance Considerations
- Optimize for CPU execution
- Minimize memory footprint
- Efficient image processing pipeline
- Cache intermediate results where appropriate

### Error Handling
- Graceful degradation on processing failures
- Clear error messages for users
- Logging for debugging
- Fallback mechanisms for edge cases

## Dependencies

### Required Python Packages
```python
pdfplumber          # PDF text extraction
PyMuPDF (fitz)      # PDF image rendering
paddlepaddle        # PaddleOCR base
paddleocr           # OCR engine with PP-Structure
opencv-python       # Image processing
Pillow              # Image I/O
```

### Optional Dependencies
```python
numpy               # Numerical operations
pandas              # Data manipulation (for structured output)
```

## Installation & Setup

### Prerequisites
- Python 3.8+
- pip package manager

### Environment Setup
```bash
pip install pdfplumber pymupdf paddlepaddle paddleocr opencv-python pillow
```

### Model Download
PaddleOCR will automatically download required models on first use:
- Table detection model
- Table structure recognition model (SLANet)
- OCR text recognition model (Korean + English)

## Usage

### Basic CLI Usage
```bash
# Extract tables from PDF
python main.py input.pdf --output output.html

# Process specific pages
python main.py input.pdf --pages 1-5 --output output.html

# Enable verbose logging
python main.py input.pdf --verbose
```

### Expected Output
- HTML file with extracted tables
- Preserved table structure and formatting
- Recognized text content in both Korean and English

## Project Status

**Current Phase**: Planning & Architecture
**Next Steps**:
1. Set up project structure
2. Implement PDF text extraction
3. Integrate PaddleOCR PP-Structure
4. Build HTML generation pipeline
5. Create CLI interface

## Notes

### Language Support
- Primary: Korean (한국어)
- Secondary: English
- OCR model supports both languages simultaneously

### Performance Expectations
- Processing speed depends on:
  - PDF complexity
  - Number of tables per page
  - Image quality (for image-based tables)
  - CPU performance

### Limitations (MVP)
- Single PDF processing at a time
- Basic merged cell support
- Simple table layouts prioritized
- No GPU acceleration in initial version
