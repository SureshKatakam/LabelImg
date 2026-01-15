# Requirements Document

## Introduction

Currently when rotating images in labelImg, the label rectangles are transformed along with the image rotation. Users need the ability to rotate only the image while keeping the label rectangles in their original positions, allowing them to re-annotate objects in the new orientation without losing existing label data.

## Requirements

### Requirement 1

**User Story:** As a user, I want to rotate the image without rotating the existing label rectangles, so that I can view the image in different orientations while preserving my annotation work.

#### Acceptance Criteria

1. WHEN I rotate an image THEN the system SHALL rotate only the image pixels
2. WHEN I rotate an image THEN the system SHALL keep all existing label rectangles in their original screen positions
3. WHEN I rotate an image THEN the system SHALL preserve all label text and properties unchanged

### Requirement 2

**User Story:** As a user, I want existing labels to remain visible and editable after rotation, so that I can modify or delete them if needed.

#### Acceptance Criteria

1. WHEN an image is rotated THEN existing label rectangles SHALL remain selectable
2. WHEN an image is rotated THEN existing label rectangles SHALL remain editable (movable, resizable)
3. WHEN an image is rotated THEN existing label rectangles SHALL maintain their original colors and properties

### Requirement 3

**User Story:** As a user, I want to be able to create new labels on the rotated image, so that I can annotate objects that are now in better positions after rotation.

#### Acceptance Criteria

1. WHEN an image is rotated THEN the system SHALL allow creation of new label rectangles
2. WHEN creating new labels on rotated images THEN the system SHALL save coordinates relative to the current image orientation
3. WHEN saving annotations THEN the system SHALL maintain coordinate accuracy for both old and new labels