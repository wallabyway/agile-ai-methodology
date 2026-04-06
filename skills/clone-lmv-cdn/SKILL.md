---
name: clone-lmv-cdn
description: Extract original LMV (Autodesk Viewer) source files from the CDN source map into references/clone-lmv-cdn/. Use when an agent needs the unminified R71-build source for Viewer3DImpl, LMVRenderer, RenderContext, MaterialManager, WebGLRenderer, UnifiedCamera, FragmentList, RenderScene, or Viewer3D.
---

# Clone LMV Source from CDN Source Map

The Autodesk Viewer CDN serves `viewer3D.min.js` with a `sourceMappingURL` that
resolves when an `Origin` header is present (as browsers always send). The source
map contains 971 original unminified source files with full content.

## Quick Check

```bash
ls references/clone-lmv-cdn/application/Viewer3DImpl.js 2>/dev/null && echo "EXISTS" || echo "MISSING"
```

If EXISTS, skip extraction.

## Extraction Steps

### 1. Download the source map

```bash
curl -sL --compressed \
  -H "Origin: http://localhost:3000" \
  "https://developer.api.autodesk.com/modelderivative/v2/viewers/7.*/viewer3D.min.js.map" \
  -o /tmp/viewer3D.min.js.map
```

Verify it downloaded (~13 MB):

```bash
wc -c /tmp/viewer3D.min.js.map
```

### 2. Extract the target files

Run this Python script to extract the 9 core files:

```bash
python3 -c "
import json, os

with open('/tmp/viewer3D.min.js.map', 'r') as f:
    sm = json.load(f)

version_line = ''
try:
    import subprocess
    header = subprocess.run(
        ['curl', '-sL', '--compressed',
         'https://developer.api.autodesk.com/modelderivative/v2/viewers/7.*/viewer3D.min.js'],
        capture_output=True, text=True, timeout=30
    ).stdout[:200]
    for line in header.split('\n'):
        if 'LMV v' in line:
            version_line = line.strip().lstrip('/*! ').rstrip(' */').strip()
            break
except Exception:
    pass

sources = sm.get('sources', [])
contents = sm.get('sourcesContent', [])
base = 'references/clone-lmv-cdn'

# Files to extract: (output_path_under_base, match_pattern)
targets = [
    ('application/Viewer3DImpl.js',      'application/Viewer3DImpl.js'),
    ('application/Viewer3D.js',          'application/Viewer3D.js'),
    ('wgs/render/LMVRenderer.js',        'wgs/render/LMVRenderer.js'),
    ('wgs/render/MaterialManager.js',    'wgs/render/MaterialManager.js'),
    ('wgs/render/RenderContext.js',      'wgs/render/RenderContext.js'),
    ('wgs/render/WebGLRenderer.js',      'wgs/render/WebGLRenderer.js'),
    ('wgs/scene/FragmentList.js',        'wgs/scene/FragmentList.js'),
    ('wgs/scene/RenderScene.js',         'wgs/scene/RenderScene.js'),
    ('tools/UnifiedCamera.js',           'tools/UnifiedCamera.js'),
]

extracted = []
for out_path, pattern in targets:
    for i, src in enumerate(sources):
        clean = src.replace('webpack://LMV/./src/', '')
        if clean == pattern and i < len(contents) and contents[i]:
            full = os.path.join(base, out_path)
            os.makedirs(os.path.dirname(full), exist_ok=True)
            with open(full, 'w') as f:
                f.write(contents[i])
            lines = len(contents[i].split('\n'))
            extracted.append((out_path, lines))
            break

with open(os.path.join(base, 'VERSION'), 'w') as f:
    f.write(version_line + '\n' if version_line else 'unknown\n')
    f.write('Source: viewer3D.min.js.map via CDN source map\n')
    f.write(f'Files extracted: {len(extracted)}\n')
    for path, lines in extracted:
        f.write(f'  {path} ({lines} lines)\n')

print(f'Extracted {len(extracted)} files to {base}/')
print(f'Version: {version_line or \"unknown\"}')
for path, lines in extracted:
    print(f'  {path} ({lines} lines)')
"
```

### 3. Verify

```bash
find references/clone-lmv-cdn -name '*.js' | sort
cat references/clone-lmv-cdn/VERSION
```

## File Mapping

| CDN source path | Purpose |
|---|---|
| application/Viewer3DImpl.js | Core viewer implementation, `glrenderer` injection, render loop, `skipCameraUpdate`, `updateCameraMatrices` |
| application/Viewer3D.js | Public API surface wrapping Viewer3DImpl |
| wgs/render/LMVRenderer.js | WebGLRenderer subclass with draw-call hooks |
| wgs/render/MaterialManager.js | Material creation, `updatePixelScale` for 2D zoom-dependent rendering |
| wgs/render/RenderContext.js | FBO management, compositing, `presentBuffer`, `enter2DMode` / `exit2DMode` |
| wgs/render/WebGLRenderer.js | Modified three.js R71 WebGLRenderer, `resetGLState` |
| wgs/scene/FragmentList.js | Per-fragment transforms, visibility, materials |
| wgs/scene/RenderScene.js | Scene/model queue for progressive rendering |
| tools/UnifiedCamera.js | Perspective/ortho unified camera, `pixelsPerUnitAtDistance` |

## Notes

- The CDN build is the **R71 variant** — preprocessor conditionals have been resolved, so only the R71 code path remains.
- The source map is only served when an `Origin` header is present (browsers send this automatically, which is why Chrome DevTools Sources panel shows the files).
- These files are read-only reference material; never modify them.
