<template>
  <div class="cesium-wrapper">
    <div class="cesium-container" ref="container"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  ionToken: {
    type: String,
    default: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJqdGkiOiJkZmRiNDYzOS1iZjZjLTQzZmYtOTI2MS1iNTQzOTE3MWI4YmMiLCJpZCI6MzM3NDkyLCJpYXQiOjE3NTY4MDMxODR9.Zy5CdLX440ruPH6EcGGycTnHzDUF-wGdbCRykFjhM2M'
  }
})

const container = ref(null)
let viewer = null
let Cesium = null
const emit = defineEmits(['mapClick'])

const locations = {
  overview: { lon: 120.07, lat: 32.18, height: 20000, pitch: -45 },
  testfield: { lon: 120.101283, lat: 32.250372, height: 20, pitch: -45 },
  workshop: { lon: 120.095096, lat: 32.245153, height: 20, pitch: -45 },
  warehouse: { lon: 120.095096, lat: 32.245153, height: 20, pitch: -45 }
}

async function loadCesium() {
  return new Promise((resolve) => {
    if (window.Cesium) {
      resolve(window.Cesium)
      return
    }
    const script = document.createElement('script')
    script.src = '/cesium/Cesium.js'
    script.onload = () => resolve(window.Cesium)
    document.head.appendChild(script)
  })
}

async function initCesium() {
  try {
    Cesium = await loadCesium()
    Cesium.Ion.defaultAccessToken = props.ionToken

    viewer = new Cesium.Viewer(container.value, {
      baseLayerPicker: false,
      fullscreenButton: false,
      geocoder: false,
      homeButton: true,
      infoBox: false,
      sceneModePicker: false,
      selectionIndicator: false,
      timeline: false,
      navigationHelpButton: false,
      creditContainer: document.createElement('div'),
      enableMouseEvent: true,
      terrain: Cesium.Terrain.fromWorldTerrain(),
      skyBox: new Cesium.SkyBox({
        show: false
      }),
      skyAtmosphere: new Cesium.SkyAtmosphere()
    })

    const scene = viewer.scene

    // 监听地图点击事件，获取动态经纬度
    const handler = new Cesium.ScreenSpaceEventHandler(scene.canvas)
    handler.setInputAction(function (click) {
      const ray = viewer.camera.getPickRay(click.position)
      const cartesian = scene.globe.pick(ray, scene)
      if (cartesian) {
        const cartographic = Cesium.Cartographic.fromCartesian(cartesian)
        const lon = Cesium.Math.toDegrees(cartographic.longitude).toFixed(6)
        const lat = Cesium.Math.toDegrees(cartographic.latitude).toFixed(6)
        emit('mapClick', { lon, lat })
      }
    }, Cesium.ScreenSpaceEventType.LEFT_CLICK)
    scene.globe.enableLighting = false

    // Make globe surface transparent for lower layers to show through
    // Use Imagery layer instead for terrain visualization
    try {
      scene.globe.baseColor = new Cesium.Color(0.1, 0.2, 0.15, 1.0)
    } catch (e) {
      // ignore if not supported
    }

    // Set a dark/nature style background for when terrain is loading
    scene.backgroundColor = new Cesium.Color(0.02, 0.05, 0.08, 1.0)

    const camera = viewer.camera
    camera.constrainedAxis = undefined

    scene.screenSpaceCameraController.zoomEventTypes = [
      Cesium.CameraEventType.WHEEL,
      Cesium.CameraEventType.PINCH
    ]
    scene.screenSpaceCameraController.rotateEventTypes = [
      Cesium.CameraEventType.LEFT_DRAG
    ]
    scene.screenSpaceCameraController.panEventTypes = [
      Cesium.CameraEventType.RIGHT_DRAG
    ]
    scene.screenSpaceCameraController.tiltEventTypes = [
      Cesium.CameraEventType.MIDDLE_DRAG
    ]

    scene.screenSpaceCameraController.minimumZoomDistance = 100
    scene.screenSpaceCameraController.maximumZoomDistance = 5000000
    scene.screenSpaceCameraController.inertia = 0.9

    // Add a marker at the main location
    viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(120.07, 32.18, 50),
      point: {
        pixelSize: 12,
        color: Cesium.Color.RED,
        outlineColor: Cesium.Color.WHITE,
        outlineWidth: 2
      }
    })

    console.log('✅ Cesium initialized')
  } catch (e) {
    console.error('❌ Cesium init failed:', e)
  }
}

function flyTo(mode) {
  if (!viewer) return

  const loc = locations[mode] || locations.overview

  viewer.camera.flyTo({
    destination: Cesium.Cartesian3.fromDegrees(loc.lon, loc.lat, loc.height),
    orientation: {
      heading: Cesium.Math.toRadians(0),
      pitch: Cesium.Math.toRadians(loc.pitch),
      roll: 0
    },
    duration: 1.5
  })

  // Update entity marker
  viewer.entities.removeAll()
  viewer.entities.add({
    position: Cesium.Cartesian3.fromDegrees(loc.lon, loc.lat, 50),
    point: {
      pixelSize: 12,
      color: Cesium.Color.RED,
      outlineColor: Cesium.Color.WHITE,
      outlineWidth: 2
    }
  })

  // Add mode-specific labels
  if (mode === 'overview') {
    viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(120.07, 32.18, 100),
      label: {
        text: '🌾 叶垛家利 - 20亩智慧农田',
        font: '14px sans-serif',
        fillColor: Cesium.Color.fromCssColorString('#00FF88'),
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 2,
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM
      }
    })
  } else if (mode === 'testfield') {
    viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(120.07, 32.18, 100),
      label: {
        text: '🌱 试验田监测区域',
        font: '14px sans-serif',
        fillColor: Cesium.Color.fromCssColorString('#00FF88'),
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 2,
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM
      }
    })
  } else if (mode === 'workshop') {
    viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(120.08, 32.17, 100),
      label: {
        text: '🏭 烘干车间 A & B',
        font: '14px sans-serif',
        fillColor: Cesium.Color.fromCssColorString('#00FF88'),
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 2,
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM
      }
    })
  } else if (mode === 'warehouse') {
    viewer.entities.add({
      position: Cesium.Cartesian3.fromDegrees(120.08, 32.17, 100),
      label: {
        text: '📦 仓库 A & B',
        font: '14px sans-serif',
        fillColor: Cesium.Color.fromCssColorString('#00FF88'),
        outlineColor: Cesium.Color.BLACK,
        outlineWidth: 2,
        style: Cesium.LabelStyle.FILL_AND_OUTLINE,
        verticalOrigin: Cesium.VerticalOrigin.BOTTOM
      }
    })
  }
}

function getViewer() {
  return viewer
}

defineExpose({ flyTo, getViewer })

onMounted(() => {
  initCesium()
})

onUnmounted(() => {
  if (viewer) {
    viewer.destroy()
    viewer = null
  }
})
</script>

<style scoped>
.cesium-wrapper {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 1;
}

.cesium-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
