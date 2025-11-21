# TableOCR Architecture

## Overview

TableOCR follows a modular pipeline architecture with clear separation of concerns. Each module handles a specific responsibility in the table extraction process.

## Project Structure

```
TableOCR/
├── src/                          # Source code
│   ├── __init__.py
│   ├── pdf_processor/            # PDF handling and extraction
│   │   ├── __init__.py
│   │   ├── text_extractor.py    # pdfplumber-based text extraction
│   │   └── image_converter.py   # PyMuPDF-based image conversion
│   ├── ocr_processor/            # OCR processing
│   │   ├── __init__.py
│   │   ├── paddle_ocr.py        # PaddleOCR PP-Structure integration
│   │   └── preprocessor.py      # Image preprocessing
│   ├── html_generator/           # HTML output generation
│   │   ├── __init__.py
│   │   ├── table_builder.py     # HTML table construction
│   │   └── formatter.py         # CSS styling and formatting
│   └── cli/                      # Command-line interface
│       ├── __init__.py
│       └── main.py               # Entry point and argument parsing
├── tests/                        # Test suite
│   ├── unit/                     # Unit tests
│   └── integration/              # Integration tests
├── examples/                     # Sample PDFs and outputs
├── docs/                         # Documentation
├── output/                       # Generated output files
└── main.py                       # Application entry point
```

## Module Breakdown

### 1. PDF Processor (`src/pdf_processor/`)

**Responsibility**: Extract table data from PDF documents using text-based or image-based methods.

**Components**:

#### `text_extractor.py`
- Uses `pdfplumber` for text-based extraction
- Attempts to extract tables directly from PDF text layer
- Fast and accurate for native PDF tables
- Returns structured table data or None if extraction fails

#### `image_converter.py`
- Uses `PyMuPDF (fitz)` to render PDF pages as images
- Converts pages to PIL Image objects
- Handles page selection and resolution settings
- Used as fallback when text extraction fails

**Key Classes/Functions**:
```python
class PDFTextExtractor:
    def extract_tables(pdf_path: str, pages: List[int]) -> List[TableData]

class PDFImageConverter:
    def convert_pages_to_images(pdf_path: str, pages: List[int], dpi: int) -> List[Image]
```

---

### 2. OCR Processor (`src/ocr_processor/`)

**Responsibility**: Perform OCR on images to extract table structure and content.

**Components**:

#### `paddle_ocr.py`
- Integrates PaddleOCR PP-Structure model
- Initializes OCR engine with Korean + English support
- Processes images and extracts table structure
- Returns structured table data with bounding boxes and text

#### `preprocessor.py`
- Image enhancement and preprocessing
- Noise reduction, deskewing, contrast adjustment
- Prepares images for optimal OCR accuracy
- Optional depending on image quality

**Key Classes/Functions**:
```python
class PaddleOCRProcessor:
    def __init__(lang: str = 'korean', use_gpu: bool = False)
    def process_image(image: Image) -> TableData
    def extract_table_structure(image: Image) -> Dict

class ImagePreprocessor:
    def enhance_image(image: Image) -> Image
    def deskew(image: Image) -> Image
```

---

### 3. HTML Generator (`src/html_generator/`)

**Responsibility**: Convert extracted table data into HTML format with proper styling.

**Components**:

#### `table_builder.py`
- Constructs HTML table elements from table data
- Handles cell positioning, rowspan, colspan
- Generates semantic HTML markup
- Preserves table structure

#### `formatter.py`
- Applies CSS styling to tables
- Formats text content (alignment, spacing)
- Generates complete HTML document
- Handles multiple tables in single output

**Key Classes/Functions**:
```python
class HTMLTableBuilder:
    def build_table(table_data: TableData) -> str
    def create_cell(content: str, row: int, col: int) -> str

class HTMLFormatter:
    def generate_html_document(tables: List[str]) -> str
    def apply_styling(html: str, style: str) -> str
```

---

### 4. CLI Interface (`src/cli/`)

**Responsibility**: Provide command-line interface for user interaction.

**Components**:

#### `main.py`
- Argument parsing using `argparse`
- Orchestrates the processing pipeline
- Error handling and user feedback
- Progress reporting for long operations

**Key Functions**:
```python
def parse_arguments() -> argparse.Namespace
def run_extraction(args: argparse.Namespace) -> None
def main() -> int
```

---

## Data Flow

```
User Input (PDF file, options)
          ↓
    CLI Interface
          ↓
    PDF Processor
          ↓
   [Text Extraction]
          ↓
   Success? ──No──→ [Image Conversion]
     │                     ↓
     Yes              OCR Processor
     │                     │
     └──────┬──────────────┘
            ↓
      Table Data (Internal Format)
            ↓
      HTML Generator
            ↓
      HTML Output File
```

## Core Data Structures

### TableData

Internal representation of a table used between modules:

```python
@dataclass
class TableCell:
    """Represents a single table cell"""
    text: str
    row: int
    col: int
    rowspan: int = 1
    colspan: int = 1
    confidence: float = 1.0  # OCR confidence (0-1)

@dataclass
class TableData:
    """Represents a complete table"""
    cells: List[TableCell]
    num_rows: int
    num_cols: int
    page_num: int
    source: str  # 'text' or 'ocr'

    def to_2d_array(self) -> List[List[str]]:
        """Convert to 2D array representation"""
        pass
```

## Processing Pipeline Details

### Phase 1: PDF Input Processing

1. **Validate Input**
   - Check file exists and is readable
   - Verify PDF format
   - Validate page numbers

2. **Attempt Text Extraction**
   - Use pdfplumber to extract tables
   - Parse table structure
   - Convert to TableData format

3. **Fallback to Image Processing** (if text extraction fails)
   - Convert PDF pages to images
   - Pass to OCR processor

### Phase 2: OCR Processing (if needed)

1. **Preprocess Images**
   - Enhance contrast
   - Denoise
   - Deskew if needed

2. **Run PaddleOCR**
   - Initialize PP-Structure model
   - Detect table regions
   - Extract table structure
   - Recognize text content

3. **Parse Results**
   - Convert OCR output to TableData
   - Validate structure consistency

### Phase 3: HTML Generation

1. **Build HTML Tables**
   - Iterate through TableData objects
   - Generate HTML table markup
   - Handle merged cells

2. **Apply Styling**
   - Add CSS for readability
   - Format bilingual text
   - Ensure responsive design

3. **Write Output**
   - Generate complete HTML document
   - Save to specified output path
   - Validate output file

## Error Handling Strategy

### Error Types

1. **Input Errors**
   - File not found
   - Invalid PDF format
   - Password-protected PDFs
   - **Action**: Clear error message, exit gracefully

2. **Processing Errors**
   - No tables found
   - OCR initialization failure
   - Image conversion failure
   - **Action**: Log warning, attempt fallback, or skip page

3. **Output Errors**
   - Write permission denied
   - Disk space full
   - **Action**: Clear error message with suggested fixes

### Error Handling Approach

```python
class TableOCRError(Exception):
    """Base exception for TableOCR"""
    pass

class PDFProcessingError(TableOCRError):
    """PDF processing failed"""
    pass

class OCRError(TableOCRError):
    """OCR processing failed"""
    pass

class OutputError(TableOCRError):
    """Output generation failed"""
    pass
```

**Principles**:
- Fail fast for invalid inputs
- Graceful degradation for processing errors
- Clear, actionable error messages
- Comprehensive logging for debugging

## Logging Strategy

### Log Levels

- **DEBUG**: Detailed processing steps, intermediate results
- **INFO**: Major pipeline steps, successful operations
- **WARNING**: Fallback activations, partial failures
- **ERROR**: Failed operations, exceptions
- **CRITICAL**: Unrecoverable errors

### Log Format

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler('tableocr.log'),
        logging.StreamHandler()
    ]
)
```

## Configuration

### Default Configuration

```python
# config.py
class Config:
    # PDF Processing
    DEFAULT_DPI = 300
    MAX_IMAGE_SIZE = (4000, 4000)

    # OCR Settings
    OCR_LANG = ['korean', 'en']
    USE_GPU = False
    OCR_CONFIDENCE_THRESHOLD = 0.5

    # Output Settings
    OUTPUT_DIR = 'output'
    DEFAULT_OUTPUT_NAME = 'extracted_tables.html'

    # Performance
    MAX_WORKERS = 4  # For future parallel processing
```

## Testing Strategy

### Unit Tests

Each module has isolated unit tests:
- `tests/unit/test_pdf_processor.py`
- `tests/unit/test_ocr_processor.py`
- `tests/unit/test_html_generator.py`
- `tests/unit/test_cli.py`

**Approach**:
- Mock external dependencies (PaddleOCR, file I/O)
- Test individual functions and classes
- Edge cases and error conditions

### Integration Tests

End-to-end pipeline tests:
- `tests/integration/test_pipeline.py`
- `tests/integration/test_text_extraction.py`
- `tests/integration/test_ocr_extraction.py`

**Approach**:
- Use real test PDFs (text-based, image-based, mixed)
- Validate complete processing pipeline
- Check output quality and accuracy

### Test Data

```
tests/fixtures/
├── text_based_table.pdf
├── image_based_table.pdf
├── korean_table.pdf
├── mixed_content.pdf
└── expected_outputs/
    ├── text_based_table.html
    └── ...
```

## Performance Considerations

### CPU Optimization

- Use efficient data structures (numpy arrays)
- Minimize image copying
- Cache PaddleOCR model initialization
- Process pages sequentially (parallel processing post-MVP)

### Memory Management

- Stream large PDFs page by page
- Release images after processing
- Limit concurrent page processing
- Monitor memory usage for large batches

### Benchmarks

Target performance (MVP):
- Text extraction: <0.5s per page
- OCR processing: 2-5s per page (CPU)
- HTML generation: <0.1s per table

## Future Enhancements (Post-MVP)

1. **Parallel Processing**
   - Multi-page concurrent processing
   - Thread pool for independent pages

2. **GPU Acceleration**
   - Optional GPU support for PaddleOCR
   - 5-10x speedup on compatible hardware

3. **Advanced Table Handling**
   - Complex merged cells
   - Nested tables
   - Table classification

4. **Batch Processing**
   - Multiple PDF processing
   - Directory scanning
   - Progress tracking

## Dependencies Between Modules

```
CLI
 ├── depends on → PDF Processor
 ├── depends on → OCR Processor
 └── depends on → HTML Generator

PDF Processor
 └── (independent)

OCR Processor
 └── (independent)

HTML Generator
 └── (independent)
```

**Design Principle**: Modules are loosely coupled. Each can be tested and developed independently. Communication happens through well-defined data structures (TableData).

## Development Workflow

1. **Start with PDF Processor**
   - Implement text extraction first
   - Test with text-based PDFs
   - Add image conversion capability

2. **Build HTML Generator**
   - Can develop in parallel with OCR
   - Test with mock TableData
   - Validate HTML output

3. **Integrate OCR Processor**
   - Most complex component
   - Test with sample images
   - Optimize for accuracy

4. **Connect with CLI**
   - Wire all modules together
   - Add error handling
   - Implement user feedback

5. **Testing & Refinement**
   - Integration testing
   - Performance optimization
   - Bug fixes and edge cases

---

This architecture provides a solid foundation for development while maintaining flexibility for future enhancements.
