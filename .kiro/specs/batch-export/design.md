# Design Document

## Overview

The batch export feature extends LabelImg's current single-file annotation saving capability to support bulk operations on multiple annotated images. The design leverages the existing format writers (PascalVocWriter, YOLOWriter, CreateMLWriter) and adds a new batch processing layer with a dedicated UI dialog for user interaction and progress tracking.

## Architecture

### High-Level Architecture

The batch export system follows a modular design pattern that integrates with the existing LabelImg architecture:

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Main Window   │───▶│ BatchExportDialog│───▶│ BatchProcessor  │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        ▼
                       ┌──────────────────┐    ┌─────────────────┐
                       │ Progress Tracker │    │ Format Converters│
                       └──────────────────┘    └─────────────────┘
                                                        │
                                                        ▼
                                              ┌─────────────────┐
                                              │ Existing Writers│
                                              │ - PascalVoc     │
                                              │ - YOLO          │
                                              │ - CreateML      │
                                              └─────────────────┘
```

### Component Integration

The batch export feature integrates with existing components:
- **MainWindow**: Adds new menu item and action for batch export
- **LabelFile**: Utilizes existing format detection and loading logic
- **Format Writers**: Reuses PascalVocWriter, YOLOWriter, and CreateMLWriter
- **Settings**: Stores user preferences for batch export operations

## Components and Interfaces

### BatchExportDialog

**Purpose**: Provides the user interface for configuring and initiating batch export operations.

**Key Methods**:
```python
class BatchExportDialog(QDialog):
    def __init__(self, parent=None)
    def select_source_directory(self)
    def select_target_directory(self) 
    def scan_source_directory(self)
    def update_file_count(self)
    def start_export(self)
    def cancel_export(self)
    def update_progress(self, current, total, filename)
```

**UI Elements**:
- Source directory selection with browse button
- Target directory selection with browse button
- Format selection dropdown (PASCAL VOC, YOLO, CreateML)
- Filter options (verified only, all images)
- File count display
- Progress bar and status label
- Export and Cancel buttons

### BatchProcessor

**Purpose**: Handles the core batch processing logic, file iteration, and format conversion coordination.

**Key Methods**:
```python
class BatchProcessor(QObject):
    # Signals
    progress_updated = pyqtSignal(int, int, str)  # current, total, filename
    export_completed = pyqtSignal(dict)  # results summary
    error_occurred = pyqtSignal(str, str)  # filename, error_message
    
    def __init__(self, source_dir, target_dir, target_format, filters)
    def start_processing(self)
    def stop_processing(self)
    def process_single_file(self, image_path, annotation_path)
    def convert_annotation_format(self, source_format, target_format, annotation_data)
```

**Processing Flow**:
1. Scan source directory for image and annotation file pairs
2. Apply user-specified filters
3. Process each file pair sequentially
4. Convert annotation format if needed
5. Save to target directory
6. Emit progress signals
7. Generate completion report

### FormatConverter

**Purpose**: Handles conversion between different annotation formats, bridging the existing format writers.

**Key Methods**:
```python
class FormatConverter:
    @staticmethod
    def detect_format(annotation_path)
    @staticmethod
    def load_annotation(annotation_path, format_type)
    @staticmethod
    def convert_to_format(annotation_data, target_format, image_info)
    @staticmethod
    def normalize_coordinates(bbox, image_size)  # For YOLO conversion
    @staticmethod
    def denormalize_coordinates(bbox, image_size)  # From YOLO conversion
```

**Conversion Matrix**:
- PASCAL VOC → YOLO: Extract bounding boxes, normalize coordinates, generate class mapping
- PASCAL VOC → CreateML: Transform XML structure to JSON format
- YOLO → PASCAL VOC: Denormalize coordinates, create XML structure
- YOLO → CreateML: Convert normalized coordinates to CreateML JSON format
- CreateML → Others: Parse JSON annotations and convert accordingly

### ProgressTracker

**Purpose**: Manages progress reporting and user feedback during batch operations.

**Key Methods**:
```python
class ProgressTracker(QObject):
    def __init__(self, total_files)
    def update_progress(self, completed_files, current_filename)
    def add_success(self, filename)
    def add_error(self, filename, error_message)
    def get_summary(self)
```

## Data Models

### BatchExportConfig

```python
@dataclass
class BatchExportConfig:
    source_directory: str
    target_directory: str
    target_format: LabelFileFormat
    include_verified_only: bool = False
    overwrite_existing: bool = False
    preserve_directory_structure: bool = True
```

### ExportResult

```python
@dataclass
class ExportResult:
    total_files: int
    successful_exports: int
    failed_exports: int
    errors: List[Tuple[str, str]]  # (filename, error_message)
    processing_time: float
```

### AnnotationData

```python
@dataclass
class AnnotationData:
    image_path: str
    image_size: Tuple[int, int, int]  # height, width, channels
    shapes: List[Dict]
    verified: bool
    source_format: LabelFileFormat
```

## Error Handling

### Error Categories

1. **File System Errors**:
   - Source directory not accessible
   - Target directory not writable
   - Missing annotation files
   - Corrupted image files

2. **Format Conversion Errors**:
   - Invalid annotation format
   - Unsupported format combinations
   - Malformed annotation data

3. **Processing Errors**:
   - Memory limitations for large datasets
   - User cancellation
   - System interruptions

### Error Handling Strategy

- **Graceful Degradation**: Continue processing remaining files when individual files fail
- **Error Logging**: Maintain detailed error log for troubleshooting
- **User Notification**: Display clear error messages with suggested actions
- **Recovery Options**: Allow users to retry failed operations

### Error Recovery

```python
class ErrorHandler:
    def handle_file_error(self, filename, error):
        # Log error, continue with next file
        pass
    
    def handle_format_error(self, filename, source_format, target_format):
        # Attempt alternative conversion method
        pass
    
    def handle_system_error(self, error):
        # Clean up partial operations, notify user
        pass
```

## Testing Strategy

### Unit Testing

1. **Format Conversion Tests**:
   - Test each format conversion combination
   - Validate coordinate transformations
   - Verify class mapping accuracy

2. **File Processing Tests**:
   - Test directory scanning logic
   - Validate file filtering
   - Test error handling for various file conditions

3. **UI Component Tests**:
   - Test dialog interactions
   - Validate progress updates
   - Test user input validation

### Integration Testing

1. **End-to-End Workflow Tests**:
   - Complete batch export scenarios
   - Multi-format conversion workflows
   - Large dataset processing

2. **Performance Tests**:
   - Memory usage with large datasets
   - Processing speed benchmarks
   - UI responsiveness during operations

### Test Data

- Sample datasets with various annotation formats
- Edge cases: empty annotations, malformed files
- Large datasets for performance testing
- Mixed format directories for conversion testing

## Performance Considerations

### Optimization Strategies

1. **Memory Management**:
   - Process files sequentially to limit memory usage
   - Release resources after each file processing
   - Implement progress checkpoints for large operations

2. **I/O Optimization**:
   - Batch file system operations where possible
   - Use efficient file reading/writing methods
   - Implement caching for repeated operations

3. **User Experience**:
   - Responsive UI during processing
   - Cancellation support
   - Progress feedback with time estimates

### Scalability

- Support for datasets with thousands of images
- Efficient handling of various image sizes
- Minimal memory footprint growth with dataset size