[![](https://img.shields.io/static/v1?label=Live%20Demo&message=✨&labelColor=2f363d&color=blue&style=flat&logo=github&logoColor=959da5)](https://nickmccarty.me/360-pano-viewer)

# 📜 Panorama Viewer Comparison Tool

This web app prototype enables interactive comparison between one or two 360° panoramic images using the [Pannellum](https://pannellum.org) viewer library. It provides a slider-based split view for visual comparison and synchronizes the camera angles between two viewers.

---

## 🔧 Elements and Variables

```js
const fileInput = document.getElementById('fileInput');
const slider = document.getElementById('slider');
const panoAContainer = document.getElementById('panoA');
const panoBContainer = document.getElementById('panoB');
const loading = document.getElementById('loading');
```

- **fileInput**: File input element for selecting one or two image files.
- **slider**: Range input controlling the percentage width of the second panorama (B).
- **panoAContainer / panoBContainer**: Containers for the left and right Pannellum viewers.
- **loading**: Element shown while panoramas are loading.

```js
let panoA, panoB, urls = [];
```

- **panoA / panoB**: Instances of the Pannellum viewers.
- **urls**: Blob URLs of the selected image files.

---

## 🌀 Function: `showLoading(state)`

Toggles visibility of the loading indicator.

```js
function showLoading(state) {
  loading.style.display = state ? 'block' : 'none';
}
```

---

## ✂️ Function: `setClip(val)`

Adjusts the visible width of the second (right) panorama viewer, based on slider value.

```js
function setClip(val) {
  panoBContainer.style.width = `${val}%`;
}
```

---

## 🔄 Function: `syncViewers()`

Synchronizes view parameters (yaw, pitch, zoom) between both viewers to keep them in sync while navigating.

```js
function syncViewers() {
  let syncing = false;
  const sync = (source, target) => {
    if (syncing) return;
    syncing = true;
    target.setYaw(source.getYaw());
    target.setPitch(source.getPitch());
    target.setHfov(source.getHfov());
    setTimeout(() => syncing = false, 10);
  };

  [ [panoA, panoB], [panoB, panoA] ].forEach(([a, b]) => {
    ['load', 'zoomchange', 'mousemove', 'touchmove', 'mousedown'].forEach(evt =>
      a.on(evt, () => sync(a, b))
    );
  });
}
```

---

## 🖼️ Function: `loadPanoramas([img1, img2])`

Initializes two synchronized Pannellum viewers with the provided image URLs. Waits for both to load, then applies initial sync and slider clipping.

```js
function loadPanoramas([img1, img2]) {
  if (panoA) panoA.destroy();
  if (panoB) panoB.destroy();

  showLoading(true);
  const view = { yaw: 0, pitch: 0, hfov: 100 };

  panoA = pannellum.viewer('panoA', { ... });
  panoB = pannellum.viewer('panoB', { ... });

  let loaded = 0;
  const onLoaded = () => {
    if (++loaded === 2) {
      panoB.setYaw(panoA.getYaw());
      panoB.setPitch(panoA.getPitch());
      panoB.setHfov(panoA.getHfov());
      setClip(slider.value);
      syncViewers();
      showLoading(false);
    }
  };

  panoA.on('load', onLoaded);
  panoB.on('load', onLoaded);
}
```

---

## 📂 File Upload Handling

Handles user file input and determines whether to show one or two panoramas.

```js
fileInput.addEventListener('change', (e) => {
  const files = Array.from(e.target.files).slice(0, 2); // Max 2 files
  if (files.length < 1) return;

  urls.forEach(u => URL.revokeObjectURL(u)); // Clean up old blobs
  urls = files.map(f => URL.createObjectURL(f));

  if (urls.length === 1) {
    // Single panorama mode
    panoA = pannellum.viewer('panoA', {
      type: 'equirectangular',
      panorama: urls[0],
      autoLoad: true,
      showControls: true
    });
    panoBContainer.style.display = 'none';
    document.getElementById('sliderContainer').style.display = 'none';
  } else {
    // Dual panorama comparison mode
    panoBContainer.style.display = 'block';
    document.getElementById('sliderContainer').style.display = 'block';
    loadPanoramas(urls);
  }
});
```

---

## 🎚️ Slider Change Event

Updates the width of the second panorama as the user moves the slider.

```js
slider.addEventListener('input', e => setClip(e.target.value));
```

---

## 🧹 Cleanup on Page Unload

Releases memory by revoking blob URLs when the user leaves the page.

```js
window.addEventListener('beforeunload', () => urls.forEach(u => URL.revokeObjectURL(u)));
```

---

## ✅ Summary

- Supports both **single and dual** 360° panorama viewing.
- Dual mode enables a **synchronized comparison** with a draggable slider to reveal one view over the other.
- Utilizes **Pannellum** for rendering equirectangular panoramas.
- Features include: loading indicator, memory cleanup, real-time viewer sync, and responsive interaction.
