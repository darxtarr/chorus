# Bubble Specification v1

**Status:** Draft
**Authors:** Opus (2025-12-06), informed by Chi Session 5 architecture discussions
**Scope:** Portable semantic unit for the Chorus/Chi ecosystem

---

## 1. What Is a Bubble?

A **Bubble** is the fundamental unit of semantic content in the Chorus ecosystem.

It is:
- **Semantically coherent**: A paragraph, code block, image, table, glyph contour — something that makes sense as a unit.
- **Identifiable**: Has a unique, stable ID that persists across moves and transformations.
- **Provenanced**: Knows where it came from (source, author, timestamp).
- **Lineaged**: Knows its ancestors and can track derivatives.

A Bubble is **not**:
- A pixel array (that's a rendering decision)
- A file (bubbles can span or subdivide files)
- A message (messages carry bubbles, but aren't bubbles themselves)

---

## 2. Core Structure

```rust
/// The fundamental semantic unit
struct Bubble {
    /// Unique identifier (UUID v7 recommended for time-ordering)
    id: BubbleId,

    /// What this bubble contains
    content: BubbleContent,

    /// Where this bubble came from
    provenance: Provenance,

    /// Relationship to other bubbles
    lineage: Lineage,

    /// How this bubble should be presented (hints, not mandates)
    display: DisplayHints,

    /// Extensible metadata
    metadata: HashMap<String, Value>,
}

/// 128-bit identifier, globally unique
struct BubbleId(u128);

impl BubbleId {
    /// Generate new ID (UUID v7: timestamp + random)
    fn new() -> Self;

    /// Parse from string representation
    fn from_str(s: &str) -> Result<Self, ParseError>;
}
```

---

## 3. Content Types

```rust
enum BubbleContent {
    /// Semantic text with structure
    Text(TextContent),

    /// Raster image
    Image(ImageContent),

    /// Vector geometry (SVG-like or B-spline curves)
    Geometry(GeometryContent),

    /// Structured data (tables, key-value, etc.)
    Data(DataContent),

    /// Reference to another bubble (symbolic link)
    Reference(BubbleId),

    /// Container holding child bubbles
    Container(Vec<BubbleId>),

    /// Raw bytes with MIME type (escape hatch)
    Raw { mime: String, bytes: Vec<u8> },
}

/// Text with semantic structure preserved
struct TextContent {
    /// The actual text (UTF-8, never decomposed)
    content: String,

    /// Semantic structure
    structure: TextStructure,

    /// Text flow direction
    flow: TextFlow,

    /// Rich text segments (bold, italic, links, etc.)
    segments: Vec<RichSegment>,
}

enum TextStructure {
    Paragraph,
    Heading { level: u8 },
    Code { language: Option<String> },
    Quote,
    ListItem { depth: u8, ordered: bool },
    Table { row: u32, col: u32 },
}

enum TextFlow {
    LeftToRight,
    RightToLeft,
    TopToBottom,
    Bidi,  // Mixed direction
}

/// Geometry from Solidus or similar
struct GeometryContent {
    /// Source format identifier
    format: GeometryFormat,

    /// The geometry data
    data: GeometryData,
}

enum GeometryFormat {
    /// Solidus B-spline output
    SolidusNurbs { version: u32 },

    /// SVG path data
    SvgPath,

    /// Pre-tessellated triangles
    Triangles,
}
```

---

## 4. Provenance

Every bubble knows its origin.

```rust
struct Provenance {
    /// When this bubble was created
    created: Timestamp,

    /// Who/what created it
    author: Author,

    /// How it was created
    source: Source,

    /// If transformed, what operation produced it
    transformation: Option<Transformation>,
}

enum Author {
    /// Human author
    Human { id: String, name: Option<String> },

    /// LLM-generated
    Model {
        model_id: String,      // e.g., "claude-opus-4-5-20251101"
        session_id: Option<String>,
        prompt_hash: Option<String>,  // Hash of generating prompt
    },

    /// System-generated (file import, protocol conversion, etc.)
    System { component: String },

    /// Unknown or anonymous
    Unknown,
}

enum Source {
    /// Created fresh by author
    Original,

    /// Imported from external file
    FileImport {
        path: String,
        offset: Option<Range<u64>>,  // Byte range in file
        hash: Option<String>,        // Content hash for verification
    },

    /// Extracted from another bubble
    Extracted { parent: BubbleId, selector: String },

    /// Transformed from another bubble
    Transformed { parent: BubbleId },

    /// Merged from multiple bubbles
    Merged { parents: Vec<BubbleId> },

    /// Received over network
    Network { node: String, received: Timestamp },
}

struct Transformation {
    /// What operation was applied
    operation: String,  // e.g., "markdown_parse", "degree_elevation", "summarize"

    /// Tool/model that performed it
    agent: String,

    /// Parameters (for reproducibility)
    params: HashMap<String, Value>,
}
```

---

## 5. Lineage

Bubbles can have ancestors and track derivatives.

```rust
struct Lineage {
    /// Direct parent bubble (if derived)
    parent: Option<BubbleId>,

    /// All ancestors (optional, for deep history)
    ancestors: Option<Vec<BubbleId>>,

    /// Known children (bubbles derived from this one)
    /// Note: This may be incomplete; children track parents, not vice versa
    children: Vec<BubbleId>,

    /// Semantic version within a lineage
    version: Option<Version>,
}

struct Version {
    major: u32,
    minor: u32,
    /// Optional label (e.g., "draft", "reviewed", "final")
    label: Option<String>,
}
```

---

## 6. Display Hints

Suggestions for rendering, not mandates. The compositor (Spirit) decides.

```rust
struct DisplayHints {
    /// Suggested position (semantic, not pixels)
    position: Option<Position>,

    /// Suggested size constraints
    size: Option<SizeHint>,

    /// Visual style hints
    style: Option<StyleHint>,

    /// Priority/urgency for attention
    priority: Priority,
}

enum Position {
    /// Grid-based (terminal-like)
    Grid { row: u32, col: u32 },

    /// Relative to another bubble
    RelativeTo { anchor: BubbleId, offset: [f32; 3] },

    /// Absolute in world coordinates (for 3D/VR)
    World { coords: [f64; 3] },

    /// Flow position (let layout engine decide)
    Flow { order: u32 },
}

enum Priority {
    Background,  // Can be hidden/collapsed
    Normal,
    Elevated,    // Should be visible
    Urgent,      // Demands immediate attention
}
```

---

## 7. Wire Format

For transmission between processes and nodes.

### 7.1 Compact Binary (for high-frequency paths)

```
[magic: u32 = 0x42554242 ("BUBB")]
[version: u8]
[flags: u8]
[id: u128]
[content_type: u8]
[content_length: u32]
[content: bytes]
[provenance_length: u16]
[provenance: bytes (MessagePack or CBOR)]
[lineage_length: u16]
[lineage: bytes (MessagePack or CBOR)]
[checksum: u32 (CRC32)]
```

### 7.2 JSON (for debugging, interop, LLM consumption)

```json
{
  "id": "01936f8a-7c4a-7def-8f3c-abc123def456",
  "content": {
    "type": "text",
    "content": "Hello, world!",
    "structure": "paragraph",
    "flow": "ltr"
  },
  "provenance": {
    "created": "2025-12-06T14:30:00Z",
    "author": { "type": "human", "id": "ulli" },
    "source": { "type": "original" }
  },
  "lineage": {
    "parent": null,
    "version": { "major": 1, "minor": 0 }
  },
  "display": {
    "priority": "normal"
  }
}
```

---

## 8. Operations

### 8.1 Core Operations

```rust
trait BubbleStore {
    /// Create a new bubble
    fn create(&mut self, content: BubbleContent, author: Author) -> BubbleId;

    /// Retrieve by ID
    fn get(&self, id: BubbleId) -> Option<&Bubble>;

    /// Derive a new bubble from an existing one
    fn derive(&mut self, parent: BubbleId, content: BubbleContent, transform: Transformation) -> BubbleId;

    /// Query by provenance
    fn query_by_author(&self, author: &Author) -> Vec<BubbleId>;
    fn query_by_source(&self, path: &str) -> Vec<BubbleId>;

    /// Query by lineage
    fn ancestors(&self, id: BubbleId) -> Vec<BubbleId>;
    fn descendants(&self, id: BubbleId) -> Vec<BubbleId>;
}
```

### 8.2 Cross-Node Operations

```rust
trait BubbleTransport {
    /// Send bubble to another node
    fn send(&self, id: BubbleId, destination: NodeId) -> Result<(), TransportError>;

    /// Receive bubble from network
    fn receive(&mut self) -> Result<Bubble, TransportError>;

    /// Sync lineage information across nodes
    fn sync_lineage(&mut self, id: BubbleId) -> Result<(), SyncError>;
}
```

---

## 9. Integration Points

### 9.1 Chi (Compositor)

Chi receives bubbles and renders them:
- TextContent → rustybuzz shaping → GPU geometry
- GeometryContent (SolidusNurbs) → tessellation → GPU triangles
- ImageContent → texture upload → GPU quad

Chi maintains a scene graph of bubbles with their display state.

### 9.2 Solidus (Geometry)

Solidus produces bubbles:
- Input: Font file
- Output: GeometryContent bubbles (one per glyph contour)
- Provenance: `Source::FileImport` with font path
- Lineage: Glyph bubbles are children of a Font bubble

### 9.3 Reflex (Telemetry)

Reflexes emit bubbles as observations:
- Content: DataContent with metrics
- Author: `Author::System { component: "reflex_cpu" }`
- Priority: Escalates from Background → Normal → Elevated → Urgent based on conditions

### 9.4 Gemma (Houndmaster)

Gemma consumes bubble streams:
- Reads reflex bubbles as narratives
- Correlates patterns across bubble provenance
- Emits summary bubbles with `Author::Model { model_id: "gemma-2-9b", ... }`

### 9.5 Spirit (Arbiter)

Spirit orchestrates bubbles:
- Routes bubbles between nodes
- Manages attention (which bubbles get display priority)
- Maintains long-horizon context by tracking lineage chains

---

## 10. Design Principles

1. **Identity is permanent.** Once a bubble has an ID, that ID never changes. Modifications create new bubbles with lineage links.

2. **Provenance is immutable.** You cannot rewrite history. A bubble's origin is fixed at creation.

3. **Lineage is append-only.** Children can be added, but parent links are permanent.

4. **Content is the truth.** Display hints are suggestions. The compositor decides presentation.

5. **Everything is a bubble.** If it has semantic meaning, it should be a bubble. Logs, metrics, errors, annotations — all bubbles.

6. **Bubbles are portable.** Any bubble can move between processes, nodes, and models without losing identity or history.

---

## 11. Open Questions

- **Garbage collection:** When can bubbles be deleted? Never (append-only)? After all references are gone? After N days?

- **Conflict resolution:** If two nodes modify the "same" bubble, how do we handle it? CRDTs? Last-write-wins? Branches?

- **Privacy/access control:** Can bubbles have visibility restrictions? "Only author can see provenance"?

- **Size limits:** What's the maximum bubble size? Should large content be chunked into multiple bubbles?

- **Compression:** Should wire format support compressed content? Per-bubble or stream-level?

---

## 12. Version History

- **v1 (2025-12-06):** Initial draft. Core structure, provenance, lineage, wire format, integration points.
