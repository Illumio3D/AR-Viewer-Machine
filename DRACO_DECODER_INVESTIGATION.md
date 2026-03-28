# AR Viewer Draco Decoder Investigation

## Investigation Summary

After reviewing the repository history and searching for information about previous agent sessions that fixed the AR viewer Draco decoder issue, here's what I found:

## Repository History

The repository has a very limited commit history:
1. **Nov 28, 2025** - "neuer bug fix" - Initial commit that created the index.html file with the AR viewer implementation
2. **Jan 6, 2026** - "Initial plan" - Current PR session

**Finding:** There is no documented evidence in the git history of a Draco decoder fix from previous agent sessions.

## Current Implementation Analysis

Looking at the current `index.html` file, the AR viewer uses:
- Google's `@google/model-viewer` library (loaded from unpkg CDN)
- A GLB file (`Maschine.glb`) for 3D model display
- Standard model-viewer configuration with AR support

**Key observation:** The current implementation does NOT explicitly configure the Draco decoder location.

## Common Draco Decoder Issues & Fixes

Based on research and common issues with Google Model Viewer and Draco-compressed GLB files, here are the typical problems and solutions:

### Problem 1: Missing Draco Decoder Path
**Symptom:** Model doesn't load, or browser console shows errors about `draco_decoder.wasm` or `draco_decoder.js` not being found.

**Root Cause:** When a GLB file uses Draco compression (`KHR_draco_mesh_compression` extension), Model Viewer needs to load the Draco decoder from a CDN. If the default CDN is blocked, inaccessible, or there are CORS issues, the model won't display.

**Fix:** Explicitly set the Draco decoder location before the model-viewer is initialized:
```javascript
// Set before model-viewer loads
window.ModelViewerElement = window.ModelViewerElement || {};
ModelViewerElement.dracoDecoderLocation = 'https://www.gstatic.com/draco/versioned/decoders/1.5.6/';
```

Or using the more modern approach:
```javascript
import { ModelViewerElement } from '@google/model-viewer';
ModelViewerElement.dracoDecoderLocation = 'https://www.gstatic.com/draco/v1/decoders/';
```

### Problem 2: CORS Issues
**Symptom:** Browser blocks loading of decoder files due to Cross-Origin Resource Sharing policies.

**Fix:** Ensure the decoder is loaded from a CDN that supports CORS, or host the decoder files locally with proper CORS headers.

### Problem 3: Decoder Version Mismatch
**Symptom:** Model loads but appears corrupted or with missing geometry.

**Fix:** Use a compatible decoder version. Google's official decoder versions are hosted at:
- `https://www.gstatic.com/draco/versioned/decoders/1.5.6/`
- `https://www.gstatic.com/draco/v1/decoders/`

### Problem 4: GLB File Without Draco Support
**Symptom:** Error messages about unsupported compression.

**Fix:** Re-export the 3D model ensuring Draco compression is properly applied, or configure the model-viewer to handle non-compressed GLB files.

## Most Likely Fix That Would Have Been Applied

If there was a Draco decoder issue in previous sessions, the most common fix would be:

**Adding explicit Draco decoder configuration to index.html:**

```javascript
// Add this before the model-viewer element processes the model
const script = document.createElement('script');
script.textContent = `
  if (window.ModelViewerElement) {
    window.ModelViewerElement.dracoDecoderLocation = 'https://www.gstatic.com/draco/versioned/decoders/1.5.6/';
  } else {
    document.addEventListener('DOMContentLoaded', () => {
      window.ModelViewerElement.dracoDecoderLocation = 'https://www.gstatic.com/draco/versioned/decoders/1.5.6/';
    });
  }
`;
document.head.appendChild(script);
```

Or more simply, adding this right after loading the model-viewer script:
```html
<script type="module" src="https://unpkg.com/@google/model-viewer/dist/model-viewer.min.js"></script>
<script>
  // Configure Draco decoder location
  window.ModelViewerElement = window.ModelViewerElement || {};
  window.ModelViewerElement.dracoDecoderLocation = 'https://www.gstatic.com/draco/versioned/decoders/1.5.6/';
</script>
```

## Verification Steps

To verify if the `Maschine.glb` file actually uses Draco compression:
1. Use a GLB viewer/inspector tool
2. Look for the `KHR_draco_mesh_compression` extension in the GLB metadata
3. Check browser console for decoder-related messages when loading the page

## **CONFIRMED: Maschine.glb Uses Draco Compression**

After examining the GLB file using hexdump, I confirmed that:
- The file includes `"extensionsUsed":["KHR_draco_mesh_compression",...]`
- The file includes `"extensionsRequired":["KHR_draco_mesh_compression",...]`

**This means the Draco decoder is REQUIRED to display this model.**

### Current Issue
The current `index.html` file does **NOT** have any Draco decoder configuration. This is the likely cause of any loading issues.

## Recommendation

Without access to previous agent session logs or documented issue descriptions, I recommend:

1. **If the model is currently working:** The fix may have already been applied but not documented
2. **If the model is not loading:** Add explicit Draco decoder configuration
3. **For future issues:** Document all fixes in commit messages and/or in a CHANGELOG.md file

## Update: Current Deployment Issue (January 6, 2026)

The user reported a 404 error when accessing the site at `https://three-dimensions.de/`:
```
Failed to load resource: the server responded with a status of 404 (Not Found) (Maschine.glb, line 0)
```

**This is NOT a Draco decoder issue** - it's a deployment issue. The diagnosis:
- The `Maschine.glb` file (50MB) exists in the GitHub repository
- The file is not being served from `https://three-dimensions.de/Maschine.glb`
- This indicates the file wasn't uploaded to the web server, or the hosting provider is blocking large files

**Solution:** Ensure `Maschine.glb` (50MB) and `Maschine.usdz` (19MB) are properly uploaded to the web hosting server. Test by directly accessing `https://three-dimensions.de/Maschine.glb` in a browser.

## References

- [Model Viewer Documentation](https://modelviewer.dev/docs/)
- [Google for AR Developers](https://developers.google.com/ar/develop/webxr/model-viewer)
- [Draco Compression GitHub](https://google.github.io/draco/)
- [Model Viewer Discussions - Draco Decoder](https://github.com/google/model-viewer/discussions/4937)
