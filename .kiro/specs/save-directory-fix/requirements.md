# Requirements Document

## Introduction

This feature addresses a critical bug in the LabelImg application where the save functionality incorrectly saves annotation files to the input folder instead of the designated output folder. Additionally, the output folder path gets changed to the input folder path when they should remain separate. This feature also adds persistence for input and output folder selections, allowing users to maintain their preferred directory structure across application sessions.

## Requirements

### Requirement 1

**User Story:** As a user annotating images, I want my annotation files to always save to the designated output folder, so that my files are properly organized and separated from the input images.

#### Acceptance Criteria

1. WHEN a user has set a default save directory (output folder) THEN the system SHALL always save annotation files to that directory
2. WHEN a user saves an annotation file THEN the system SHALL NOT change the output folder path to match the input folder path
3. WHEN no default save directory is set THEN the system SHALL fall back to saving in the same directory as the image file
4. WHEN the save dialog opens THEN the system SHALL default to the output folder path if one is configured

### Requirement 2

**User Story:** As a user with separate input and output directories, I want the output folder path to remain unchanged during save operations, so that I maintain my intended file organization structure.

#### Acceptance Criteria

1. WHEN a user has configured different input and output directories THEN the system SHALL maintain both paths independently
2. WHEN saving annotation files THEN the system SHALL NOT modify the default_save_dir variable to match the input directory
3. WHEN the current_path() method is called THEN it SHALL prioritize the default_save_dir over the image file directory
4. WHEN the save file dialog is opened THEN it SHALL use the output directory as the starting location

### Requirement 3

**User Story:** As a user working with batch annotation, I want consistent save behavior across all images, so that all my annotation files end up in the correct output location without manual intervention.

#### Acceptance Criteria

1. WHEN processing multiple images in sequence THEN the system SHALL maintain the same output directory for all saves
2. WHEN auto-saving is enabled THEN the system SHALL use the configured output directory consistently
3. WHEN switching between images THEN the system SHALL NOT reset the save directory to the input directory
4. WHEN the application starts with a configured save directory THEN it SHALL display the correct output path in the status bar
### 
Requirement 4

**User Story:** As a user who frequently works with the same input and output directories, I want the application to remember my folder selections, so that I don't have to reconfigure them every time I restart the application.

#### Acceptance Criteria

1. WHEN a user selects an input directory (opens a folder) THEN the system SHALL save this path to persistent settings
2. WHEN a user configures an output directory (changes save directory) THEN the system SHALL save this path to persistent settings
3. WHEN the application starts THEN it SHALL automatically load the previously selected input and output directories from settings
4. WHEN the application loads saved directories THEN it SHALL verify the directories still exist before using them
5. WHEN a saved directory no longer exists THEN the system SHALL fall back to default behavior and clear the invalid path from settings

### Requirement 5

**User Story:** As a user returning to work on annotations, I want the application to automatically open my last used input directory and maintain my output directory preference, so that I can continue working seamlessly.

#### Acceptance Criteria

1. WHEN the application starts with saved input directory THEN it SHALL automatically load images from that directory
2. WHEN the application starts with saved output directory THEN it SHALL set that as the default save location
3. WHEN both input and output directories are saved THEN the system SHALL maintain them as separate, independent paths
4. WHEN the user changes either directory during a session THEN the system SHALL update the persistent settings immediately
5. WHEN the application displays status messages THEN it SHALL show both the current input and output directory paths