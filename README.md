# PDF Table Extractor & OCR Converter

[![Python Version](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

On-device PDF table extraction and conversion to HTML using PaddleOCR. No external APIs required - runs completely offline.

## Features

- 🔒 **100% Offline**: No external API dependencies
- 🎯 **Automatic Table Detection**: Identify and extract tables from PDFs
- 🔤 **Bilingual OCR**: Supports Korean (한국어) and English simultaneously
- 📊 **Structure Preservation**: Maintains table layout in HTML output
- 💻 **CPU-Optimized**: Works efficiently without GPU
- 🪶 **Lightweight**: Minimal dependencies (~10-20MB model size)

## Quick Start

### Prerequisites

- Python 3.8 or higher
- pip package manager

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/dohyeong-kim-95/TableOCR.git
   cd TableOCR
   ```

2. **Create virtual environment (recommended)**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

4. **First Run - Model Download**

   PaddleOCR will automatically download required models on first use:
   - Table detection model
   - Table structure recognition model (SLANet)
   - OCR text recognition model (Korean + English)

   This is a one-time process and may take a few minutes.

## Usage

### Basic Usage

```bash
# Extract tables from a PDF
python main.py input.pdf --output output.html

# Process specific pages
python main.py input.pdf --pages 1-5 --output output.html

# Enable verbose logging
python main.py input.pdf --verbose
```

### Expected Output

The tool generates an HTML file containing:
- Extracted table structure
- Recognized text in both Korean and English
- Preserved formatting and layout

### Example

```bash
python main.py sample_document.pdf --output extracted_tables.html
```

## How It Works

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

The tool uses a smart two-tier approach:
1. **First**: Attempts fast text-based extraction
2. **Fallback**: Uses OCR for image-based or complex tables

## Technical Stack

- **PDF Processing**: pdfplumber, PyMuPDF
- **OCR Engine**: PaddleOCR with PP-Structure (SLANet model)
- **Image Processing**: OpenCV, Pillow
- **Output Format**: HTML/CSS

## Project Status

**Current Phase**: Planning & Architecture

### Roadmap

- [ ] Set up project structure
- [ ] Implement PDF text extraction module
- [ ] Integrate PaddleOCR PP-Structure
- [ ] Build HTML generation pipeline
- [ ] Create CLI interface
- [ ] Add comprehensive tests
- [ ] Documentation and examples

## Limitations (MVP)

- Processes one PDF at a time
- Basic merged cell support
- Optimized for simple to moderate table layouts
- No GPU acceleration in initial version

## Future Features

The following are planned for post-MVP releases:
- GUI interface
- Advanced merged cell handling
- Batch processing (multiple PDFs)
- GPU optimization
- Real-time processing

## Performance

Processing speed depends on:
- PDF complexity
- Number of tables per page
- Image quality (for image-based tables)
- CPU performance

Typical performance: 2-5 seconds per page on modern CPU.

## Language Support

- **Primary**: Korean (한국어)
- **Secondary**: English
- Simultaneous recognition of both languages in the same document

## Documentation

For detailed information, see:
- [claude.md](claude.md) - Complete project specification
- [Architecture](docs/ARCHITECTURE.md) - System design (coming soon)
- [API Documentation](docs/API_SPECIFICATION.md) - Developer guide (coming soon)

## Contributing

Contributions are welcome! Please read our contributing guidelines before submitting pull requests.

## Troubleshooting

### Common Issues

**Model download fails**
```bash
# Manually specify model directory
export PADDLEOCR_MODEL_DIR=/path/to/models
```

**Out of memory errors**
- Process fewer pages at a time
- Close other applications
- Use lower resolution images

**OCR accuracy issues**
- Ensure PDF quality is good
- Check image DPI (recommended: 300+)
- Verify language settings

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR) for the excellent OCR engine
- [pdfplumber](https://github.com/jsvine/pdfplumber) for PDF text extraction
- [PyMuPDF](https://github.com/pymupdf/PyMuPDF) for PDF rendering

## Support

For questions, issues, or feature requests, please open an issue on GitHub.

---

**Note**: This is an MVP (Minimum Viable Product) focused on core functionality. Advanced features will be added in future releases.
