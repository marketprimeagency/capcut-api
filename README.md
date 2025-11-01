# CapCut API

A TypeScript/Deno library for programmatically generating CapCut video editing projects. This library allows you to create, manipulate, and export CapCut project files without opening the CapCut application.

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Getting Started](#getting-started)
- [Core Concepts](#core-concepts)
- [API Reference](#api-reference)
- [Examples](#examples)
- [Testing](#testing)
- [Configuration](#configuration)
- [License](#license)

## Overview

The CapCut API provides a programmatic interface to create video editing projects compatible with CapCut. It enables developers to:

- Create video editing projects from scratch
- Add and manipulate video, audio, and text materials
- Apply effects, animations, and transitions
- Control timing, positioning, and transformations
- Export projects to CapCut-compatible JSON format

## Installation

This project uses Deno as the runtime. Ensure you have Deno installed on your system.

```bash
# Clone the repository
git clone <repository-url>
cd project

# Run tests to verify installation
deno test
```

### Dependencies

The project uses the following dependencies (managed via `deno.json`):

- `@std/assert` - Assertion library for testing
- `@std/collections` - Collection utilities
- `@std/fs` - File system utilities
- `@std/path` - Path manipulation
- `fluent-ffmpeg` - FFmpeg wrapper for media processing
- `ffmpeg-probe` - Media file metadata extraction
- `imagescript` - Image processing

## Getting Started

### Basic Project Creation

```typescript
import { Content } from "./src/model/project/Content.ts";
import { MetaInfo } from "./src/model/project/MetaInfo.ts";
import { Track } from "./src/model/tracks/Track.ts";
import { Segment } from "./src/model/segments/Segments.ts";
import { VideoMaterial } from "./src/model/materials/VideoMaterial.ts";
import { DraftMaterialFactory } from "./src/model/draft_materials/DraftMaterialFactory.ts";

// Create a new project
const content = new Content();
const meta = new MetaInfo();

// Load a video asset
const videoAsset = await DraftMaterialFactory.create({
  file_Path: "path/to/video.mp4"
});

// Add to project metadata
meta.setDraftMaterials([videoAsset]);

// Create video material and segment
const videoMaterial = new VideoMaterial();
videoMaterial.setDraftMaterial(videoAsset);

const segment = new Segment();
segment.setMaterial(videoMaterial);

// Create a track and add the segment
const track = new Track({ type: "video" });
track.addSegment(segment);
content.track_instances.upsert(track);

// Export project files
const contentJSON = JSON.stringify(content);
const metaJSON = JSON.stringify(meta);
```

## Core Concepts

### Project Structure

A CapCut project consists of two main files:
- `draft_content.json` - Contains the project structure, tracks, segments, and materials
- `draft_meta_info.json` - Contains metadata about imported media files

### Hierarchy

```
Content (Project)
├── Materials (Videos, Audio, Text, Effects)
├── Tracks (Video Track, Audio Track, Text Track)
│   └── Segments (Individual clips on the timeline)
│       ├── Material (Reference to a material)
│       └── Extra Materials (Effects, animations, speed controls)
└── Canvas Config (Resolution, aspect ratio)
```

### Key Components

#### 1. **Content**
The main project container that holds all tracks and materials.

#### 2. **MetaInfo**
Metadata container for imported media files (draft materials).

#### 3. **Tracks**
Timeline tracks that hold segments. Types include:
- Video tracks
- Audio tracks
- Text tracks

#### 4. **Segments**
Individual clips on a track with timing and transformation properties.

#### 5. **Materials**
Media and effect resources:
- **VideoMaterial** - Video clips
- **AudioMaterial** - Audio clips
- **TextMaterial** - Text elements
- **AnimationMaterial** - Animation effects
- **EffectMaterial** - Visual effects
- **SpeedMaterial** - Speed controls
- **BeatMaterial** - Beat detection for audio

#### 6. **DraftMaterial**
Represents imported media files with metadata extracted via FFmpeg.

## API Reference

### Content

Main project container.

```typescript
class Content {
  // Material storage
  material_instances: MaterialInstanceList;

  // Track storage
  track_instances: Repository<Track>;

  // Set all materials at once
  setMaterials(materials: MaterialInstanceList): void;

  // Merge materials into existing collection
  mergeMaterials(materials: Partial<MaterialInstanceList>): void;

  // Set all tracks
  setTracks(tracks: Track[]): void;

  // Split a segment at a specific time
  splitSegment(segment: Segment, time: number): Segment;

  // Apply an effect to a segment
  applyEffect(effect: EffectMaterial, segment: Segment): void;
}
```

### Track

Timeline track container.

```typescript
class Track {
  constructor(init?: Partial<{
    type: "video" | "audio" | "text";
    flag: number;
    attribute: TrackAttribute;
  }>);

  // Add a segment to the track
  addSegment(segment: Segment): void;

  // Set all segments
  setSegments(segments: Segment[]): void;

  // Remove gaps between segments
  removeEmptyTrackSpace(): void;
}

enum TrackAttribute {
  NONE,
  MUTE,
  HIDDEN,
  MUTE_HIDDEN,
  LOCKED,
  MUTE_LOCKED,
  HIDDEN_LOCKED,
  ALL
}
```

### Segment

Individual clip on a track.

```typescript
class Segment {
  // Set the primary material
  setMaterial(material: VideoMaterial | AudioMaterial | TextMaterial): void;

  // Set additional materials (effects, animations, etc.)
  setExtraMaterials(materials: Partial<MaterialInstance>): void;

  // Set timing
  setTargetTimerange(timerange: { start: number; duration: number }): void;
  setSourceTimerange(timerange: { start: number; duration: number }): void;

  // Set transformation
  mergeClip(clip: Partial<{
    scale: { x: number; y: number };
    transform: { x: number; y: number };
    rotation: number;
    alpha: number;
  }>): void;

  // Set playback speed
  setSpeed(speed: number): void;

  // Adjust speed to achieve target duration
  speedToDuration(duration: number): void;

  // Set volume (0-1)
  setVolume(volume: number): void;
}
```

### VideoMaterial

Video clip material.

```typescript
class VideoMaterial {
  // Link to imported video file
  setDraftMaterial(draftMaterial: DraftMaterial): void;

  // Get video duration
  getDuration(): number;
}
```

### AudioMaterial

Audio clip material.

```typescript
class AudioMaterial {
  // Create from predefined audio
  static get(name: string): AudioMaterial;

  // Set draft material
  setDraftMaterial(draftMaterial: DraftMaterial): void;
}
```

### TextMaterial

Text element material.

```typescript
class TextMaterial {
  // Set text content
  setText(value: string): void;

  // Set font
  setFont(font: "ZYFervent"): void;

  // Set font size
  setFontSize(value: number): void;

  // Set text color (normalized RGB: 0-1)
  setColor(color: [number, number, number]): void;

  // Add stroke/outline
  setStroke(stroke: {
    color: [number, number, number];
    width: number;
  }): void;

  // Apply text effect
  setEffect(effect: EffectMaterial): void;

  // Preset for card text
  static cardTextPreset(cardName: string): TextMaterial;

  // Preset for card price
  static cardPricePreset(price: string): TextMaterial;
}
```

### AnimationMaterial

Animation effects for materials.

```typescript
class AnimationMaterial {
  // Link to segment
  setSegment(segment: Segment): void;

  // Add fade in effect
  addFadeIn(): void;

  // Add fade out effect
  addFadeOut(): void;
}
```

### EffectMaterial

Visual effects.

```typescript
class EffectMaterial {
  // Get predefined effect
  static get(name: "FOUNDATION" | string): EffectMaterial;
}
```

### DraftMaterialFactory

Factory for creating imported media materials.

```typescript
class DraftMaterialFactory {
  // Create from file path
  static create(data: {
    file_Path: string;
  }): Promise<DraftMaterial>;

  // Get predefined asset
  static get(name: "blackScreen" | "FDN"): Promise<DraftMaterial>;
}
```

### Timing

All timing values are in **nanoseconds** (1 second = 1,000,000 nanoseconds).

```typescript
interface Timerange {
  start: number;      // Start time in nanoseconds
  duration: number;   // Duration in nanoseconds
}

// Example: 3 seconds
const threeSeconds = 3000000; // 3 * 1,000,000
```

### Positioning

Positioning uses normalized coordinates:
- X-axis: -0.5 to 0.5 (left to right)
- Y-axis: -0.5 to 0.5 (top to bottom)
- Center: (0, 0)

```typescript
// Position at top center
segment.mergeClip({
  transform: { x: 0.0, y: -0.4 }
});
```

### Scaling

Scale values are relative to original size:
- 1.0 = 100% (original size)
- 0.5 = 50%
- 2.0 = 200%

```typescript
// Scale to 80%
segment.mergeClip({
  scale: { x: 0.8, y: 0.8 }
});
```

## Examples

### Example 1: Simple Video with Audio

```typescript
import { Content } from "./src/model/project/Content.ts";
import { MetaInfo } from "./src/model/project/MetaInfo.ts";
import { Track } from "./src/model/tracks/Track.ts";
import { Segment } from "./src/model/segments/Segments.ts";
import { VideoMaterial } from "./src/model/materials/VideoMaterial.ts";
import { AudioMaterial } from "./src/model/materials/AudioMaterial.ts";
import { DraftMaterialFactory } from "./src/model/draft_materials/DraftMaterialFactory.ts";

const content = new Content();
const meta = new MetaInfo();

// Load video
const videoDraft = await DraftMaterialFactory.create({
  file_Path: "video.mp4"
});
meta.setDraftMaterials([videoDraft]);

// Create video track
const videoMaterial = new VideoMaterial();
videoMaterial.setDraftMaterial(videoDraft);

const videoSegment = new Segment();
videoSegment.setMaterial(videoMaterial);

const videoTrack = new Track({ type: "video" });
videoTrack.addSegment(videoSegment);
content.track_instances.upsert(videoTrack);

// Add background music
const audioMaterial = AudioMaterial.get("lofiWarmth");
const audioSegment = new Segment();
audioSegment.setMaterial(audioMaterial);
audioSegment.setTargetTimerange({ start: 0, duration: 10000000 });

const audioTrack = new Track({ type: "audio" });
audioTrack.addSegment(audioSegment);
content.track_instances.upsert(audioTrack);

content.mergeMaterials({
  videos: [videoMaterial],
  audios: [audioMaterial]
});
```

### Example 2: Text with Animation

```typescript
import { Content } from "./src/model/project/Content.ts";
import { Track } from "./src/model/tracks/Track.ts";
import { Segment } from "./src/model/segments/Segments.ts";
import { TextMaterial } from "./src/model/materials/TextMaterial.ts";
import { AnimationMaterial } from "./src/model/materials/AnimationMaterial.ts";

const content = new Content();

// Create animated text
const textMaterial = new TextMaterial();
textMaterial.setText("Hello World");
textMaterial.setFont("ZYFervent");
textMaterial.setFontSize(20);
textMaterial.setColor([1, 1, 1]); // White

const textSegment = new Segment();
const animation = new AnimationMaterial();
animation.setSegment(textSegment);

textSegment.setMaterial(textMaterial);
textSegment.setExtraMaterials({
  material_animations: animation
});

textSegment.setTargetTimerange({
  start: 0,
  duration: 5000000 // 5 seconds
});

textSegment.mergeClip({
  transform: { x: 0.0, y: 0.0 } // Center
});

animation.addFadeIn();
animation.addFadeOut();

const textTrack = new Track({ type: "text" });
textTrack.addSegment(textSegment);
content.track_instances.upsert(textTrack);

content.mergeMaterials({
  texts: [textMaterial],
  material_animations: [animation]
});
```

### Example 3: Multi-Segment Video with Speed Control

```typescript
import { Content } from "./src/model/project/Content.ts";
import { Track } from "./src/model/tracks/Track.ts";
import { Segment } from "./src/model/segments/Segments.ts";
import { VideoMaterial } from "./src/model/materials/VideoMaterial.ts";
import { SpeedMaterial } from "./src/model/materials/SpeedMaterial.ts";
import { DraftMaterialFactory } from "./src/model/draft_materials/DraftMaterialFactory.ts";

const content = new Content();

const videoDraft = await DraftMaterialFactory.create({
  file_Path: "video.mp4"
});

// First segment: normal speed
const videoMaterial1 = new VideoMaterial();
videoMaterial1.setDraftMaterial(videoDraft);

const segment1 = new Segment();
segment1.setMaterial(videoMaterial1);
segment1.setSourceTimerange({ start: 0, duration: 5000000 });

// Second segment: 2x speed
const videoMaterial2 = new VideoMaterial();
videoMaterial2.setDraftMaterial(videoDraft);

const speedMaterial = new SpeedMaterial();
const segment2 = new Segment();
segment2.setMaterial(videoMaterial2);
segment2.setExtraMaterials({ speeds: speedMaterial });
segment2.setSourceTimerange({ start: 5000000, duration: 5000000 });
segment2.setSpeed(2.0); // 2x speed

const videoTrack = new Track({ type: "video" });
videoTrack.addSegment(segment1);
videoTrack.addSegment(segment2);
content.track_instances.upsert(videoTrack);

content.mergeMaterials({
  videos: [videoMaterial1, videoMaterial2],
  speeds: [speedMaterial]
});
```

## Testing

The project includes comprehensive tests for all major components.

```bash
# Run all tests
deno test

# Run tests with coverage
deno test --coverage

# Run specific test file
deno test src/model/tracks/Track.test.ts
```

### Test Coverage

Tests are available for:
- Core models (Root, Repository)
- Draft materials and factories
- All material types
- Segments and tracks
- Metadata extraction

## Configuration

### CapCut Paths Configuration

Before using the library, configure the paths to your CapCut installation in `capcutPaths.ts`:

```typescript
const capcutInstallationFolder = join('C:', 'Users', 'YourName', 'AppData', 'Local', 'CapCut', '2.2.0.491');
const capcutCacheFolder = join('C:', 'Users', 'YourName', 'AppData', 'Local', 'CapCut', 'User Data', 'Cache');
```

These paths are used for:
- System fonts
- Cached effects
- Cached music

### Project Output

Projects are typically exported to the CapCut drafts folder:

```typescript
const projectFolder = join(
  "path/to/capcut/drafts",
  "CapCut Drafts",
  "project-name"
);

await Deno.mkdir(projectFolder, { recursive: true });
await Deno.writeTextFile(
  join(projectFolder, "draft_content.json"),
  JSON.stringify(content)
);
await Deno.writeTextFile(
  join(projectFolder, "draft_meta_info.json"),
  JSON.stringify(meta)
);
```

## Architecture

### Repository Pattern

The library uses an in-memory repository pattern for managing collections:

```typescript
class InMemoryRepository<T extends Root> implements Repository<T> {
  getAll(): T[];
  getById(id: string): T | undefined;
  upsert(item: T): T;
  removeById(id: string): T | undefined;
}
```

### Root Base Class

All models extend the `Root` class which provides:
- Automatic ID generation
- Clone functionality
- JSON serialization
- Type-safe data access

```typescript
class Root<TData, TClass> {
  protected data: WithId<TData>;

  get id(): string;
  clone(): TClass;
  getDataValues(): WithId<TData>;
}
```

### Material System

Materials are organized into categories:
- **Media Materials**: Video, Audio, Image
- **Effect Materials**: Effects, Animations, Transitions
- **Control Materials**: Speed, Volume, Beat
- **Text Materials**: Text, TextTemplate

All materials can be serialized to JSON for export.

## Time Representations

Time in CapCut projects is measured in nanoseconds:

```typescript
// Conversion helpers
const secondsToNs = (seconds: number) => seconds * 1_000_000;
const nsToSeconds = (ns: number) => ns / 1_000_000;

// Examples
const oneSecond = 1_000_000;      // 1 second
const halfSecond = 500_000;       // 0.5 seconds
const threeSeconds = 3_000_000;   // 3 seconds
```

## Color Representations

Colors are represented as normalized RGB arrays (0-1):

```typescript
// Examples
const white = [1, 1, 1];
const black = [0, 0, 0];
const red = [1, 0, 0];
const gray = [0.5, 0.5, 0.5];

// Convert to hex
const convertToHex = (color: number[]) => {
  return `#${color
    .map(n => Math.round(n * 255))
    .map(n => n.toString(16).padStart(2, '0'))
    .join('')}`;
};
```

## Common Patterns

### Adding a Video Clip

```typescript
const draft = await DraftMaterialFactory.create({ file_Path: "video.mp4" });
meta.setDraftMaterials([draft]);

const material = new VideoMaterial();
material.setDraftMaterial(draft);

const segment = new Segment();
segment.setMaterial(material);

const track = new Track({ type: "video" });
track.addSegment(segment);
content.track_instances.upsert(track);
```

### Positioning Elements

```typescript
// Top left
segment.mergeClip({ transform: { x: -0.4, y: -0.4 } });

// Center
segment.mergeClip({ transform: { x: 0.0, y: 0.0 } });

// Bottom right
segment.mergeClip({ transform: { x: 0.4, y: 0.4 } });
```

### Applying Effects

```typescript
const effect = EffectMaterial.get("FOUNDATION");
segment.setExtraMaterials({ effects: effect });
content.applyEffect(effect, segment);
```

## Limitations

- Currently targets CapCut version 2.2.0
- Limited to features available in the CapCut JSON format
- Requires FFmpeg for media metadata extraction
- Text fonts must be available in CapCut installation

## Contributing

This project was created for personal use. If you'd like to contribute:

1. Ensure all tests pass
2. Follow existing code style
3. Add tests for new features
4. Document new functionality

## License

This project is licensed under the GNU General Public License v3.0. See the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Uses FFmpeg for media processing
- Compatible with CapCut video editor
- Built with Deno runtime
