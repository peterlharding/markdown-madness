# Google Maps API Marker Migration Guide

## ⚠️ Deprecation Notice
As of **February 21st, 2024**, `google.maps.Marker` is deprecated. Use `google.maps.marker.AdvancedMarkerElement` instead.

## Import Structure

### Browser/Client-side JavaScript

#### Modern Import (Recommended)
```javascript
import { Loader } from '@googlemaps/js-api-loader';

const loader = new Loader({
  apiKey: "YOUR_API_KEY",
  version: "weekly",
  libraries: ["marker"] // Important: include the marker library
});

loader.load().then(() => {
  // Use google.maps.marker.AdvancedMarkerElement here
});
```

#### HTML Script Tag
```html
<script async defer
  src="https://maps.googleapis.com/maps/api/js?key=YOUR_API_KEY&libraries=marker&callback=initMap">
</script>
```

### Package Dependencies
```json
{
  "dependencies": {
    "@googlemaps/js-api-loader": "^1.16.2",
    "@googlemaps/google-maps-services-js": "^3.3.42"
  }
}
```

## Migration Examples

### Basic Migration

#### Old Way (Deprecated)
```javascript
const oldMarker = new google.maps.Marker({
  position: { lat: -34.397, lng: 150.644 },
  map: map,
  title: "Hello World!",
  icon: 'path/to/icon.png'
});
```

#### New Way (AdvancedMarkerElement)
```javascript
const newMarker = new google.maps.marker.AdvancedMarkerElement({
  position: { lat: -34.397, lng: 150.644 },
  map: map,
  title: "Hello World!",
  content: buildContent(), // Custom HTML content
  gmpClickable: true
});
```

### Complete Implementation

```javascript
async function initMap() {
  // Request needed libraries
  const { Map } = await google.maps.importLibrary("maps");
  const { AdvancedMarkerElement, PinElement } = await google.maps.importLibrary("marker");
  
  const map = new Map(document.getElementById("map"), {
    zoom: 4,
    center: { lat: -25.344, lng: 131.031 },
    mapId: "DEMO_MAP_ID", // Required for AdvancedMarkerElement
  });

  // Simple pin marker
  const pinMarker = new AdvancedMarkerElement({
    map: map,
    position: { lat: -25.344, lng: 131.031 },
    title: "Pin Marker"
  });

  // Custom styled pin
  const pin = new PinElement({
    background: "#FBBC04",
    borderColor: "#137333",
    glyphColor: "white",
  });

  const styledMarker = new AdvancedMarkerElement({
    map: map,
    position: { lat: -25.444, lng: 131.131 },
    content: pin.element,
    title: "Styled Pin Marker"
  });

  // Custom HTML content marker
  const customMarker = new AdvancedMarkerElement({
    map: map,
    position: { lat: -25.244, lng: 131.231 },
    content: buildCustomContent(),
    title: "Custom HTML Marker"
  });

  // Add click listener
  customMarker.addListener("click", () => {
    console.log("Advanced marker clicked!");
  });
}

function buildCustomContent() {
  const content = document.createElement("div");
  content.classList.add("property");
  content.innerHTML = `
    <div class="icon">
      <i aria-hidden="true" class="fa fa-icon" title="marker"></i>
    </div>
    <div class="details">
      <div class="price">Custom Marker</div>
      <div class="address">123 Main St</div>
    </div>
  `;
  return content;
}
```

## Key Differences

| Old Marker | AdvancedMarkerElement |
|------------|----------------------|
| `new google.maps.Marker()` | `new google.maps.marker.AdvancedMarkerElement()` |
| `icon` property for images | `content` property for HTML/PinElement |
| No map ID required | **Requires `mapId` in map config** |
| Limited customization | Full HTML customization support |
| Basic click events | Enhanced interaction capabilities |

## Important Requirements

### ✅ Must Have
1. **Map ID Required**: AdvancedMarkerElement requires a `mapId` in your map configuration
2. **Marker Library**: Include `marker` in libraries array when loading Google Maps API
3. **Dynamic Import**: Use `google.maps.importLibrary("marker")` for modern loading
4. **Content vs Icon**: Replace `icon` property with `content` property

### 🎨 Custom Marker CSS
```css
.property {
  align-items: center;
  background-color: #FFFFFF;
  border-radius: 50px;
  color: #263238;
  display: flex;
  font-size: 14px;
  gap: 15px;
  padding: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.3);
}

.property::after {
  border-left: 9px solid transparent;
  border-right: 9px solid transparent;
  border-top: 9px solid #FFFFFF;
  content: "";
  height: 0;
  left: 50%;
  position: absolute;
  top: 95%;
  transform: translate(-50%, 0);
  width: 0;
  z-index: 1;
}
```

## Migration Checklist

- [ ] Update Google Maps API loader to include `marker` library
- [ ] Add `mapId` to map configuration
- [ ] Replace `google.maps.Marker` with `google.maps.marker.AdvancedMarkerElement`
- [ ] Update `icon` property to `content` property
- [ ] Use `google.maps.importLibrary("marker")` for dynamic imports
- [ ] Test marker click events and interactions
- [ ] Update any marker-related styling to use HTML/CSS instead of icon images

## Benefits of Migration

✨ **Enhanced Features:**
- Full HTML customization support
- Better performance and rendering
- More flexible styling options
- Improved accessibility
- Future-proof API compatibility
- Advanced interaction capabilities

## Resources

- [Google Maps AdvancedMarkerElement Documentation](https://developers.google.com/maps/documentation/javascript/advanced-markers)
- [Migration Guide](https://developers.google.com/maps/documentation/javascript/advanced-markers/migration)
- [PinElement Documentation](https://developers.google.com/maps/documentation/javascript/reference/advanced-markers#PinElement)