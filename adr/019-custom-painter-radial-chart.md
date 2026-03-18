# ADR-019: CustomPainter for radial chart rendering

**Status:** Accepted
**Date:** 2026-02-24
**Deciders:** Mert Ertugrul

## Context

The core visual feature of SliceFocus is a 24-hour radial pie chart where users create, view, and interact with time slices. The rendering needed to support custom gestures (drag-to-create, pinch-to-zoom), hit testing on arc segments, responsive sizing, and smooth animations.

## Decision

Use Flutter's native CustomPainter for the radial chart. The DayCanvasPainter handles arc rendering, time markers, active slice highlighting, grid patterns for unplanned slices, and current-time indicators. Hit testing converts pointer angles to time positions for drag-to-create and context menu interactions.

## Alternatives Considered

- **Third-party chart packages (fl_chart, syncfusion)**: Could not support the required custom gesture handling (drag-to-create slices, context menus on arc segments).
- **WebView-based canvas**: Rejected for performance and platform integration reasons.

## Consequences

- Full control over rendering, gestures, and animations
- More code to maintain (custom hit testing, angle-to-time math, midnight-wrap handling) but no dependency constraints
- Enabled later features like pinch-to-zoom, current time indicator, and Pomodoro subdivision animation without package limitations

*Related: MER-23, MER-22, MER-211, MER-196*
