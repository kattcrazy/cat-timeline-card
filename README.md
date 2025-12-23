# Cat Timeline Card

A customizable Home Assistant card that displays a timeline of cat-related events from your Home Assistant entities. This card automatically monitors binary sensors and displays events in a clean, scrollable timeline format.

## Installation

1. Edit `base-cat-timeline-card.js` so that it suits your setup. There are placeholders in place already. Post in discussions if you need help.
2. Copy your finished file to your Home Assistant `www` folder (typically `/config/www/`)
3. Add the resource to your `configuration.yaml` or via the ui

```yaml
lovelace:
  resources:
    - url: /local/base-cat-timeline-card.js
      type: module
```

3. Restart Home Assistant
4. Add the card to your dashboard using the card picker or manually:

```yaml
type: custom:cat-timeline-card
max_events: 10
max_time_ago: 24
primary_icon_colour: "#ff0000"
other_cat_icon_colour: "#00ff00"
```

## Configuration Options

The card can be configured via the UI editor or YAML. Available options:

### `max_events` (optional)
- **Type**: Number
- **Default**: `null` (no limit)
- **Description**: Maximum number of events to display in the timeline. Leave empty to show all events.

### `max_time_ago` (optional)
- **Type**: Number (hours)
- **Default**: `null` (no limit)
- **Description**: Only show events from the last X hours. Leave empty to show all events. Supports decimals (e.g., `0.5` for 30 minutes).

### `primary_icon_colour` (optional)
- **Type**: String (hex code or CSS variable)
- **Default**: `null` (uses `var(--primary-color, #03a9f4)`)
- **Description**: Primary colour for timeline icons. This is also used as the default/fallback colour for the scrollbar and timeline line.
- **Examples**: `#ff0000`, `var(--accent-color)`

### `other_cat_icon_colour` (optional)
- **Type**: String (hex code or CSS variable)
- **Default**: `null` (uses primary icon colour)
- **Description**: Colour for timeline icons of other cat events. Leave empty to use the primary colour.
- **Examples**: `#00ff00`, `var(--secondary-color)`

## Customization Guide

This card is designed to be customized by editing the JavaScript file directly. The code is organized into three main sections that you'll need to modify:

### 1. Events Section

The **Events** section defines which Home Assistant entities to monitor and what state changes trigger timeline events.

#### Location in Code

- `initializePreviousStates()` - Lines ~97-120: Defines which entities to track for state changes
- `loadHistoricalEvents()` - Lines ~122-163: Defines which entities to load from history
- `checkStateChanges()` - Lines ~272-344: Defines the logic for detecting and creating events

#### How to Customize

**Step 1: Update Entity Lists**

In `initializePreviousStates()` and `loadHistoricalEvents()`, replace the placeholder entity IDs with your actual Home Assistant entity IDs:

```javascript
const entitiesToTrack = [
  'binary_sensor.PLACEHOLDER_CAT_FLAP_ENTITY',        // Replace with your cat flap entity
  'binary_sensor.PLACEHOLDER_FOOD_BOWL_ENTITY',        // Replace with your food bowl entity
  'binary_sensor.PLACEHOLDER_CAMERA_1_OCCUPANCY_ENTITY', // Replace with your camera occupancy entity
  // ... add more entities as needed
];
```

**Step 2: Add Event Detection Logic**

In `checkStateChanges()`, customize the event detection. There are two helper methods:

- `checkEntityState(entityId, targetState, message, icon)` - For simple state changes
- `checkEntityStateWithCondition(entityId, targetState, conditionEntityId, conditionValue, messageIfTrue, messageIfFalse, icon)` - For state changes that depend on another entity's value

**Example - Simple Event:**
```javascript
// Track cat flap opening
this.checkEntityState(
  'binary_sensor.my_cat_flap',  // Entity to monitor
  'on',                          // State that triggers the event
  'Cat used the cat flap',       // Message to display
  'mdi:cat'                      // Icon to display
);
```

**Example - Conditional Event:**
```javascript
// Track camera detection with classification
this.checkEntityStateWithCondition(
  'binary_sensor.camera_occupancy',           // Entity to monitor
  'on',                                        // State that triggers
  'sensor.camera_classification',             // Entity to check for condition
  'MyCat',                                     // Value that makes messageIfTrue appear
  'MyCat was seen by the camera',              // Message if condition matches
  'A cat was seen by the camera',             // Message if condition doesn't match
  'mdi:camera'                                 // Icon to display
);
```

**Step 3: Handle Historical Data**

In `processHistoricalData()` (Lines ~165-245), add logic to process historical state changes. This follows the same pattern as `checkStateChanges()` but works with historical data:

```javascript
if (entityId === 'binary_sensor.my_cat_flap' && state === 'on') {
  this.addTimelineEventFromHistory('Cat used the cat flap', 'mdi:cat', timestamp);
}
```

### 2. Messages Section

The **Messages** section defines the text that appears in the timeline for each event.

#### Location in Code

- `checkStateChanges()` - Lines ~272-344: Messages for real-time events
- `processHistoricalData()` - Lines ~165-245: Messages for historical events
- `getIconColorForEvent()` - Lines ~438-448: Logic to determine icon colour based on message content

#### How to Customize

**Replace Placeholder Messages**

Find all instances of `'PLACEHOLDERTEXTPLACEHOLDERTEXT'` and replace them with your custom messages:

```javascript
// Before
this.checkEntityState(
  'binary_sensor.my_cat_flap',
  'on',
  'PLACEHOLDERTEXTPLACEHOLDERTEXT',  // Replace this
  'mdi:cat'
);

// After
this.checkEntityState(
  'binary_sensor.my_cat_flap',
  'on',
  'Fluffy used the cat flap',  // Your custom message
  'mdi:cat'
);
```

**Customize Icon Colour Logic**

In `getIconColorForEvent()` (Lines ~438-448), customize how the card determines which colour to use for each event. By default, it checks if the message contains a specific string:

```javascript
getIconColorForEvent(event) {
  const message = event.message || '';
  const isPrimaryEvent = message.includes('PLACEHOLDERTEXTPLACEHOLDERTEXT'); // Replace with your cat's name or identifier
  
  if (isPrimaryEvent) {
    return this.primaryIconColour || 'var(--primary-color, #03a9f4)';
  } else {
    return this.otherCatIconColour || this.primaryIconColour || 'var(--primary-color, #03a9f4)';
  }
}
```

**Example:**
```javascript
// Use primary colour for events containing "Fluffy", other colour for everything else
const isPrimaryEvent = message.includes('Fluffy');
```

### 3. Icons Section

The **Icons** section defines which Material Design Icons (MDI) are displayed for each event type.

#### Location in Code

- `checkStateChanges()` - Lines ~272-344: Icons for real-time events
- `processHistoricalData()` - Lines ~165-245: Icons for historical events
- `render()` - Lines ~542-576: Icon rendering and special icon handling

#### How to Customize

**Replace Placeholder Icons**

Find all instances where icons are specified and replace them with MDI icon names:

```javascript
// Default placeholders
'mdi:cat'      // Generic cat icon
'mdi:camera'   // Camera icon
'mdi:bell'     // Sound/bell icon

// Replace with your preferred icons
'mdi:camera-outline' // Camera icon variant
'mdi:volume-high'   // Sound icon
'mdi:bowl-mix'      // Food bowl icon
'mdi:home-export'   // Cat flap icon
```

**Available Icon Sources:**
- Material Design Icons: https://materialdesignicons.com/
- Home Assistant Icons: Use any icon available in Home Assistant

**Special Icon Positioning**

If you have an icon that needs special positioning (like the bowl icon in my custom version), you can add a CSS class:

1. In the `render()` method, identify your special icon:
```javascript
const isSpecialIcon = event.icon === 'mdi:bowl-mix';
```

2. Apply the special class:
```javascript
html`<div class="timeline-icon ${isSpecialIcon ? 'timeline-icon-special' : ''}">`
```

3. Adjust the CSS in the `static styles` section (Line ~674):
```css
.timeline-icon-special {
  top: -4px; /* Adjust as needed */
}
```

**Clickable Events**

To make camera events clickable (e.g., to open a media browser), customize the `handleCameraClick()` function in the `render()` method (Lines ~548-555):

```javascript
const handleCameraClick = () => {
  if (!this.hass) return;
  
  // Example: Open Frigate media browser
  const urlPath = 'app%2Cmedia-source%3A%2F%2Ffrigate/video%2Cmedia-source%3A%2F%2Ffrigate%2Ffrigate%2Fevent-search%2Fclips%2F%2F%2F%2F%2F%2F/video%2Cmedia-source%3A%2F%2Ffrigate%2Ffrigate%2Fevent-search%2Fclips%2F.YOUR_CAMERA%2F%2F%2FYOUR_CAMERA%2F%2F';
  const url = this.hass.hassUrl(`/media-browser/browser/${urlPath}`);
  window.open(url, '_blank');
};
```

To make other event types clickable, modify the condition in the render method:
```javascript
const isClickableEvent = event.icon === 'mdi:camera' || event.icon === 'mdi:your-icon';
```

### Advanced Customization

#### Rate Limiting Events

Some events (like food bowl usage) may trigger frequently. The card includes rate limiting logic. To customize:

1. In `addTimelineEvent()` (Lines ~385-407), modify the rate limiting condition:
```javascript
if (message === 'Your specific message') {
  if (!this.shouldAddEvent(now)) {
    return; // Skip if too soon
  }
  this.lastEventTime = now;
}
```

2. Adjust the time threshold in `shouldAddEvent()` (Lines ~376-383):
```javascript
const fiveMinutes = 5 * 60 * 1000; // Change to your desired interval
```

#### Event Deduplication

The card automatically collapses consecutive duplicate events (showing "..." for 3+ duplicates). This happens in `collapseConsecutiveDuplicates()` (Lines ~456-489). The logic is automatic and doesn't require customization.

#### Timestamp Format

To change how timestamps are displayed, modify `formatTimestamp()` (Lines ~409-420):

```javascript
formatTimestamp(date) {
  // Current format: "3:45pm 15/12"
  // Customize as needed
  const hours = date.getHours();
  const minutes = date.getMinutes();
  // ... your custom formatting
}
```

## Code Structure Overview

```
CatTimelineCard
├── Constructor & Properties
│   ├── State management
│   └── Configuration variables
├── Configuration
│   ├── setConfig() - Processes YAML config
│   └── getStubConfig() - Default config values
├── Lifecycle Methods
│   ├── updated() - Handles state updates
│   ├── firstUpdated() - Initial setup
│   └── willUpdate() - Pre-render operations
├── State Tracking
│   ├── initializePreviousStates() - Sets up entity tracking
│   └── checkStateChanges() - Monitors real-time changes
├── Historical Data
│   ├── loadHistoricalEvents() - Loads past events
│   └── processHistoricalData() - Processes history
├── Event Management
│   ├── addTimelineEvent() - Adds new events
│   ├── addTimelineEventFromHistory() - Adds historical events
│   └── shouldAddEvent() - Rate limiting
├── Display Logic
│   ├── getFilteredEvents() - Applies time/event limits
│   ├── collapseConsecutiveDuplicates() - Deduplication
│   ├── getIconColorForEvent() - Icon colour logic
│   └── formatTimestamp() - Time formatting
└── Rendering
    └── render() - Main template
```

## Tips for Customization

1. **Start Small**: Begin by customizing one event type, test it, then add more.

2. **Use Browser Console**: Open your browser's developer console to see any errors. The card logs errors with the prefix "CAT TIMELINE CARD:".

3. **Test Entity IDs**: Make sure your entity IDs are correct. You can find them in Home Assistant's Developer Tools > States.

4. **Icon Names**: Use the exact MDI icon name (case-sensitive). Test icons at https://materialdesignicons.com/

5. **Message Consistency**: Keep message text consistent if you want the deduplication feature to work properly.

6. **Rate Limiting**: If events are too frequent, adjust the rate limiting logic or time threshold.

7. **Historical Data**: The card loads 24 hours of history by default (or `max_time_ago` if set). Make sure your entities have history enabled in Home Assistant.

## Troubleshooting

**No events appearing:**
- Check that your entity IDs are correct
- Verify entities are updating in Home Assistant
- Check browser console for errors
- Ensure entities have history enabled

**Events appearing too frequently:**
- Add or adjust rate limiting in `addTimelineEvent()`
- Increase the time threshold in `shouldAddEvent()`

**Wrong icon colours:**
- Check `getIconColorForEvent()` logic
- Verify your message text matches the condition
- Ensure config colours are set correctly

**Historical events not loading:**
- Verify entities have history enabled
- Check that entity IDs in `loadHistoricalEvents()` match your setup
- Check browser console for API errors

