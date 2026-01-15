# Implementation Plan

- [x] 1. Change default zoom mode to fit-window


  - Modify the MainWindow.__init__() method to set zoom_mode to FIT_WINDOW instead of MANUAL_ZOOM
  - Set the fit window action as checked by default to reflect the active state
  - _Requirements: 2.1, 2.3_



- [ ] 2. Add automatic scaling after image loading
  - Modify the load_file() method to apply fit-to-window scaling after successful image loading
  - Add calls to toggle_actions(True), adjust_scale(), and paint_canvas() after canvas.setFocus(True)



  - Ensure canvas is properly enabled for user interaction
  - _Requirements: 1.1, 1.2, 2.2_

- [ ] 3. Test and verify zoom functionality
  - Create test cases to verify images automatically fit within window boundaries
  - Test that manual zoom controls still work correctly after the changes
  - Verify window resizing maintains fit-to-window scaling when active
  - _Requirements: 1.3, 3.1, 3.2, 3.3_