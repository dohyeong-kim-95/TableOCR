# API Specification

This document defines the public APIs for all TableOCR modules. Use this as a reference when implementing and integrating modules.

## Table of Contents

1. [Data Structures](#data-structures)
2. [PDF Processor API](#pdf-processor-api)
3. [OCR Processor API](#ocr-processor-api)
4. [HTML Generator API](#html-generator-api)
5. [CLI API](#cli-api)
6. [Configuration](#configuration)
7. [Exceptions](#exceptions)

---

## Data Structures

### TableCell

Represents a single cell in a table.

```python
from dataclasses import dataclass
from typing import Optional

@dataclass
class TableCell:
    """Single table cell with position and content"""
    text: str                    # Cell content
    row: int                     # Row index (0-based)
    col: int                     # Column index (0-based)
    rowspan: int = 1            # Number of rows spanned
    colspan: int = 1            # Number of columns spanned
    confidence: float = 1.0     # OCR confidence score (0.0-1.0)
    bbox: Optional[tuple] = None # Bounding box (x1, y1, x2, y2)

    def __post_init__(self):
        """Validate cell data"""
        assert self.row >= 0, "Row must be non-negative"
        assert self.col >= 0, "Column must be non-negative"
        assert self.rowspan > 0, "Rowspan must be positive"
        assert self.colspan > 0, "Colspan must be positive"
        assert 0.0 <= self.confidence <= 1.0, "Confidence must be between 0 and 1"
```

### TableData

Represents a complete table with all cells.

```python
from dataclasses import dataclass
from typing import List, Optional
from enum import Enum

class ExtractionMethod(Enum):
    """Source of table extraction"""
    TEXT = "text"      # Extracted from PDF text layer
    OCR = "ocr"        # Extracted via OCR

@dataclass
class TableData:
    """Complete table representation"""
    cells: List[TableCell]              # All cells in the table
    num_rows: int                        # Total number of rows
    num_cols: int                        # Total number of columns
    page_num: int                        # PDF page number (1-based)
    source: ExtractionMethod             # Extraction method used
    confidence: Optional[float] = None   # Overall table confidence (OCR only)

    def to_2d_array(self) -> List[List[str]]:
        """
        Convert table to 2D array representation.

        Returns:
            2D list where array[row][col] = cell text

        Note: Merged cells will have their content in the top-left position
        """
        array = [["" for _ in range(self.num_cols)] for _ in range(self.num_rows)]
        for cell in self.cells:
            array[cell.row][cell.col] = cell.text
        return array

    def get_cell(self, row: int, col: int) -> Optional[TableCell]:
        """Get cell at specific position"""
        for cell in self.cells:
            if cell.row == row and cell.col == col:
                return cell
        return None

    def validate(self) -> bool:
        """Validate table structure consistency"""
        if not self.cells:
            return False
        if self.num_rows <= 0 or self.num_cols <= 0:
            return False
        # Check all cells are within bounds
        for cell in self.cells:
            if cell.row >= self.num_rows or cell.col >= self.num_cols:
                return False
        return True
```

---

## PDF Processor API

Module: `src/pdf_processor/`

### text_extractor.py

#### PDFTextExtractor

```python
from typing import List, Optional
from pathlib import Path

class PDFTextExtractor:
    """Extract tables from PDF using text-based method (pdfplumber)"""

    def __init__(self):
        """Initialize text extractor"""
        pass

    def extract_tables(
        self,
        pdf_path: Path,
        pages: Optional[List[int]] = None
    ) -> List[TableData]:
        """
        Extract tables from PDF text layer.

        Args:
            pdf_path: Path to PDF file
            pages: List of page numbers to process (1-based).
                   If None, process all pages.

        Returns:
            List of TableData objects, one per table found

        Raises:
            FileNotFoundError: If PDF file doesn't exist
            PDFProcessingError: If PDF cannot be opened or is corrupted
            ValueError: If pages list contains invalid page numbers

        Example:
            >>> extractor = PDFTextExtractor()
            >>> tables = extractor.extract_tables(Path("doc.pdf"), pages=[1, 2])
            >>> print(f"Found {len(tables)} tables")
        """
        pass

    def extract_from_page(
        self,
        pdf_path: Path,
        page_num: int
    ) -> List[TableData]:
        """
        Extract tables from a single page.

        Args:
            pdf_path: Path to PDF file
            page_num: Page number (1-based)

        Returns:
            List of TableData objects found on the page

        Raises:
            ValueError: If page_num is invalid
            PDFProcessingError: If extraction fails
        """
        pass

    @staticmethod
    def is_text_extractable(pdf_path: Path) -> bool:
        """
        Check if PDF has extractable text layer.

        Args:
            pdf_path: Path to PDF file

        Returns:
            True if PDF contains text that can be extracted

        Example:
            >>> if PDFTextExtractor.is_text_extractable(Path("doc.pdf")):
            ...     # Use text extraction
            ... else:
            ...     # Fall back to OCR
        """
        pass
```

### image_converter.py

#### PDFImageConverter

```python
from typing import List, Optional
from pathlib import Path
from PIL import Image

class PDFImageConverter:
    """Convert PDF pages to images for OCR processing"""

    def __init__(self, dpi: int = 300):
        """
        Initialize image converter.

        Args:
            dpi: Resolution for image conversion (default: 300)
                 Higher values = better quality but slower
        """
        self.dpi = dpi

    def convert_pages_to_images(
        self,
        pdf_path: Path,
        pages: Optional[List[int]] = None
    ) -> List[tuple[Image.Image, int]]:
        """
        Convert PDF pages to PIL Images.

        Args:
            pdf_path: Path to PDF file
            pages: List of page numbers to convert (1-based).
                   If None, convert all pages.

        Returns:
            List of (Image, page_number) tuples

        Raises:
            FileNotFoundError: If PDF file doesn't exist
            PDFProcessingError: If conversion fails

        Example:
            >>> converter = PDFImageConverter(dpi=300)
            >>> images = converter.convert_pages_to_images(Path("doc.pdf"))
            >>> for img, page_num in images:
            ...     print(f"Page {page_num}: {img.size}")
        """
        pass

    def convert_page(
        self,
        pdf_path: Path,
        page_num: int
    ) -> Image.Image:
        """
        Convert a single page to image.

        Args:
            pdf_path: Path to PDF file
            page_num: Page number (1-based)

        Returns:
            PIL Image object

        Raises:
            ValueError: If page_num is invalid
            PDFProcessingError: If conversion fails
        """
        pass

    def get_page_count(self, pdf_path: Path) -> int:
        """
        Get total number of pages in PDF.

        Args:
            pdf_path: Path to PDF file

        Returns:
            Number of pages

        Raises:
            FileNotFoundError: If PDF doesn't exist
            PDFProcessingError: If PDF cannot be opened
        """
        pass
```

---

## OCR Processor API

Module: `src/ocr_processor/`

### paddle_ocr.py

#### PaddleOCRProcessor

```python
from typing import List, Optional
from pathlib import Path
from PIL import Image

class PaddleOCRProcessor:
    """Process images using PaddleOCR PP-Structure for table extraction"""

    def __init__(
        self,
        lang: str = 'korean',
        use_gpu: bool = False,
        show_log: bool = False
    ):
        """
        Initialize PaddleOCR processor.

        Args:
            lang: Language for OCR ('korean', 'en', 'korean,en')
            use_gpu: Whether to use GPU acceleration
            show_log: Whether to show PaddleOCR logs

        Example:
            >>> processor = PaddleOCRProcessor(lang='korean', use_gpu=False)
        """
        self.lang = lang
        self.use_gpu = use_gpu
        self.show_log = show_log
        self._engine = None  # Lazy initialization

    def process_image(
        self,
        image: Image.Image,
        page_num: int = 1
    ) -> List[TableData]:
        """
        Extract tables from image using OCR.

        Args:
            image: PIL Image object
            page_num: Page number for metadata (default: 1)

        Returns:
            List of TableData objects found in image

        Raises:
            OCRError: If OCR processing fails
            ValueError: If image is invalid

        Example:
            >>> from PIL import Image
            >>> processor = PaddleOCRProcessor()
            >>> img = Image.open("page.png")
            >>> tables = processor.process_image(img, page_num=1)
        """
        pass

    def extract_table_structure(
        self,
        image: Image.Image
    ) -> dict:
        """
        Extract raw table structure from image.

        Args:
            image: PIL Image object

        Returns:
            Dictionary containing:
                - 'cells': List of cell dictionaries
                - 'structure': Table structure info
                - 'confidence': Overall confidence score

        Raises:
            OCRError: If structure extraction fails
        """
        pass

    def _initialize_engine(self):
        """Initialize PaddleOCR engine (lazy loading)"""
        pass

    def _parse_ocr_result(
        self,
        result: dict,
        page_num: int
    ) -> List[TableData]:
        """
        Parse PaddleOCR output into TableData objects.

        Args:
            result: Raw PaddleOCR output
            page_num: Page number

        Returns:
            List of TableData objects
        """
        pass
```

### preprocessor.py

#### ImagePreprocessor

```python
from PIL import Image
import numpy as np

class ImagePreprocessor:
    """Preprocess images for optimal OCR performance"""

    @staticmethod
    def enhance_image(image: Image.Image) -> Image.Image:
        """
        Enhance image quality for better OCR.

        Applies:
        - Contrast enhancement
        - Noise reduction
        - Sharpening

        Args:
            image: Input PIL Image

        Returns:
            Enhanced PIL Image
        """
        pass

    @staticmethod
    def deskew(image: Image.Image) -> Image.Image:
        """
        Correct image skew/rotation.

        Args:
            image: Input PIL Image

        Returns:
            Deskewed PIL Image
        """
        pass

    @staticmethod
    def binarize(image: Image.Image, threshold: int = 128) -> Image.Image:
        """
        Convert image to binary (black and white).

        Args:
            image: Input PIL Image
            threshold: Binarization threshold (0-255)

        Returns:
            Binarized PIL Image
        """
        pass

    @staticmethod
    def resize_if_needed(
        image: Image.Image,
        max_size: tuple = (4000, 4000)
    ) -> Image.Image:
        """
        Resize image if it exceeds max dimensions.

        Args:
            image: Input PIL Image
            max_size: (max_width, max_height) tuple

        Returns:
            Resized image (or original if within limits)
        """
        pass
```

---

## HTML Generator API

Module: `src/html_generator/`

### table_builder.py

#### HTMLTableBuilder

```python
from typing import List

class HTMLTableBuilder:
    """Build HTML table markup from TableData"""

    def __init__(self, include_confidence: bool = False):
        """
        Initialize table builder.

        Args:
            include_confidence: Whether to include OCR confidence in output
        """
        self.include_confidence = include_confidence

    def build_table(self, table_data: TableData) -> str:
        """
        Generate HTML table from TableData.

        Args:
            table_data: TableData object to convert

        Returns:
            HTML string representing the table

        Example:
            >>> builder = HTMLTableBuilder()
            >>> html = builder.build_table(table_data)
            >>> print(html)
            <table class="extracted-table">
              <tr><td>Cell 1</td><td>Cell 2</td></tr>
            </table>
        """
        pass

    def build_multiple_tables(
        self,
        tables: List[TableData]
    ) -> List[str]:
        """
        Generate HTML for multiple tables.

        Args:
            tables: List of TableData objects

        Returns:
            List of HTML strings, one per table
        """
        pass

    def _create_cell(self, cell: TableCell) -> str:
        """
        Create HTML for a single cell.

        Args:
            cell: TableCell object

        Returns:
            HTML string for the cell (<td> or <th>)
        """
        pass

    def _handle_merged_cells(self, table_data: TableData) -> str:
        """
        Handle cells with rowspan/colspan.

        Args:
            table_data: TableData with potential merged cells

        Returns:
            HTML with proper rowspan/colspan attributes
        """
        pass
```

### formatter.py

#### HTMLFormatter

```python
from typing import List, Optional
from pathlib import Path

class HTMLFormatter:
    """Format and style HTML output"""

    DEFAULT_CSS = """
    <style>
        .extracted-table {
            border-collapse: collapse;
            width: 100%;
            margin: 20px 0;
        }
        .extracted-table td, .extracted-table th {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }
        .extracted-table tr:nth-child(even) {
            background-color: #f2f2f2;
        }
        .low-confidence {
            background-color: #fff3cd;
        }
    </style>
    """

    def __init__(self, custom_css: Optional[str] = None):
        """
        Initialize formatter.

        Args:
            custom_css: Custom CSS to use instead of default
        """
        self.css = custom_css or self.DEFAULT_CSS

    def generate_html_document(
        self,
        tables_html: List[str],
        title: str = "Extracted Tables"
    ) -> str:
        """
        Create complete HTML document with all tables.

        Args:
            tables_html: List of HTML table strings
            title: Document title

        Returns:
            Complete HTML document string

        Example:
            >>> formatter = HTMLFormatter()
            >>> doc = formatter.generate_html_document(
            ...     tables_html=["<table>...</table>"],
            ...     title="My Tables"
            ... )
        """
        pass

    def save_to_file(
        self,
        html_content: str,
        output_path: Path
    ) -> None:
        """
        Save HTML content to file.

        Args:
            html_content: HTML string to save
            output_path: Output file path

        Raises:
            OutputError: If file cannot be written
        """
        pass

    def add_metadata(
        self,
        html: str,
        metadata: dict
    ) -> str:
        """
        Add metadata comments to HTML.

        Args:
            html: HTML content
            metadata: Dictionary of metadata (source_file, date, etc.)

        Returns:
            HTML with metadata comments
        """
        pass
```

---

## CLI API

Module: `src/cli/`

### main.py

```python
import argparse
from pathlib import Path
from typing import Optional, List

def parse_arguments() -> argparse.Namespace:
    """
    Parse command-line arguments.

    Returns:
        Parsed arguments namespace
    """
    parser = argparse.ArgumentParser(
        description="Extract tables from PDF and convert to HTML"
    )
    parser.add_argument(
        "pdf_file",
        type=Path,
        help="Path to PDF file"
    )
    parser.add_argument(
        "-o", "--output",
        type=Path,
        default=Path("output/extracted_tables.html"),
        help="Output HTML file path (default: output/extracted_tables.html)"
    )
    parser.add_argument(
        "-p", "--pages",
        type=str,
        help="Pages to process (e.g., '1,3,5' or '1-5' or '1,3-5')"
    )
    parser.add_argument(
        "-v", "--verbose",
        action="store_true",
        help="Enable verbose logging"
    )
    parser.add_argument(
        "--dpi",
        type=int,
        default=300,
        help="DPI for image conversion (default: 300)"
    )
    parser.add_argument(
        "--use-gpu",
        action="store_true",
        help="Use GPU for OCR (if available)"
    )
    parser.add_argument(
        "--force-ocr",
        action="store_true",
        help="Force OCR even if text extraction is possible"
    )
    return parser.parse_args()

def parse_page_range(page_string: str) -> List[int]:
    """
    Parse page range string.

    Args:
        page_string: String like "1,3,5" or "1-5" or "1,3-5"

    Returns:
        List of page numbers

    Example:
        >>> parse_page_range("1,3-5,8")
        [1, 3, 4, 5, 8]
    """
    pass

def run_extraction(args: argparse.Namespace) -> int:
    """
    Main extraction logic.

    Args:
        args: Parsed command-line arguments

    Returns:
        Exit code (0 for success, non-zero for error)
    """
    pass

def main() -> int:
    """
    Main entry point.

    Returns:
        Exit code
    """
    pass

if __name__ == "__main__":
    exit(main())
```

---

## Configuration

### config.py

```python
from dataclasses import dataclass
from pathlib import Path

@dataclass
class Config:
    """Global configuration"""

    # PDF Processing
    DEFAULT_DPI: int = 300
    MAX_IMAGE_SIZE: tuple = (4000, 4000)

    # OCR Settings
    OCR_LANGUAGES: list = None  # Will be set in __post_init__
    USE_GPU: bool = False
    OCR_CONFIDENCE_THRESHOLD: float = 0.5

    # Output Settings
    OUTPUT_DIR: Path = Path("output")
    DEFAULT_OUTPUT_NAME: str = "extracted_tables.html"

    # Performance
    MAX_WORKERS: int = 4  # For future parallel processing

    # Logging
    LOG_LEVEL: str = "INFO"
    LOG_FILE: Path = Path("tableocr.log")

    def __post_init__(self):
        if self.OCR_LANGUAGES is None:
            self.OCR_LANGUAGES = ['korean', 'en']

# Global config instance
config = Config()
```

---

## Exceptions

### exceptions.py

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

class ValidationError(TableOCRError):
    """Data validation failed"""
    pass
```

---

## Usage Examples

### Complete Workflow

```python
from pathlib import Path
from src.pdf_processor.text_extractor import PDFTextExtractor
from src.pdf_processor.image_converter import PDFImageConverter
from src.ocr_processor.paddle_ocr import PaddleOCRProcessor
from src.html_generator.table_builder import HTMLTableBuilder
from src.html_generator.formatter import HTMLFormatter

def extract_tables_from_pdf(pdf_path: Path, output_path: Path):
    """Complete extraction workflow"""

    # Step 1: Try text extraction
    text_extractor = PDFTextExtractor()
    tables = text_extractor.extract_tables(pdf_path)

    # Step 2: Fall back to OCR if no tables found
    if not tables:
        print("Text extraction failed, using OCR...")
        converter = PDFImageConverter(dpi=300)
        images = converter.convert_pages_to_images(pdf_path)

        ocr_processor = PaddleOCRProcessor(lang='korean')
        tables = []
        for img, page_num in images:
            tables.extend(ocr_processor.process_image(img, page_num))

    # Step 3: Generate HTML
    builder = HTMLTableBuilder()
    tables_html = builder.build_multiple_tables(tables)

    formatter = HTMLFormatter()
    html_doc = formatter.generate_html_document(tables_html)
    formatter.save_to_file(html_doc, output_path)

    print(f"Extracted {len(tables)} tables to {output_path}")
```

---

## Type Hints

All APIs use Python type hints for better IDE support and type checking:

```bash
# Run type checking
mypy src/
```

## Testing APIs

Each module should have corresponding test files:

```python
# tests/unit/test_pdf_processor.py
def test_text_extractor():
    extractor = PDFTextExtractor()
    tables = extractor.extract_tables(Path("test.pdf"))
    assert len(tables) > 0
    assert all(t.validate() for t in tables)
```

---

This API specification serves as a contract for implementation. Stick to these interfaces to ensure smooth integration between modules.
