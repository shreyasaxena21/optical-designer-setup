<template>
  <div class="designer-layout">
    <aside class="component-palette">
      <h2>Optical Components</h2>
      <div class="component-list">
        <div class="palette-item" draggable="true" @dragstart="handleDragStart($event, 'Source', laserSourceImage)">
          <img :src="laserSourceImage" alt="Laser Source" />
          <span>Laser Source</span>
        </div>
        
        <div class="palette-item" draggable="true" @dragstart="handleDragStart($event, 'Mirror', planarMirrorImage)">
          <img :src="planarMirrorImage" alt="Planar Mirror" />
          <span>Planar Mirror</span>
        </div>

        <div class="palette-item" draggable="true" @dragstart="handleDragStart($event, 'Lens', convexLensImage)">
          <img :src="convexLensImage" alt="Convex Lens" />
          <span>Convex Lens</span>
        </div>
        
        <div class="palette-item" draggable="true" @dragstart="handleDragStart($event, 'Beam Splitter', beamSplitterImage)">
          <img :src="beamSplitterImage" alt="Beam Splitter" />
          <span>Beam Splitter</span>
        </div>

        <div class="palette-item" draggable="true" @dragstart="handleDragStart($event, 'Photo Detector', photoDetectorImage)">
          <img :src="photoDetectorImage" alt="Photo Detector" />
          <span>Photo Detector</span>
        </div>
      </div>
    </aside>

    <main 
      class="grid-area" 
      @dragover.prevent="handleDragOver" 
      @drop="handleDrop"
    >
      <div v-for="comp in components" :key="comp.id" 
        class="grid-component" 
        :class="{ 'selected': selectedComponentId === comp.id }" 
        :data-id="comp.id" 
        :style="{ 
          left: `${comp.x}px`, 
          top: `${comp.y}px`, 
          transform: `rotate(${comp.angle}deg)`,
          zIndex: selectedComponentId === comp.id ? 10 : 9 
        }"
        @mousedown="startDragComponent(comp.id, $event)"
        @click.stop="selectComponent(comp.id)"
      >
        <img v-if="comp.imageUrl" :src="comp.imageUrl" :alt="comp.type" class="component-image" />
        <div v-else class="component-placeholder">{{ comp.type.charAt(0) }}</div>
        
        <!-- Interaction count badge -->
        <div v-if="getInteractionCount(comp.id) > 0" class="interaction-badge">
          {{ getInteractionCount(comp.id) }}
        </div>
        
        <div class="rotation-handle" @mousedown.stop="startRotateComponent(comp.id, $event)"></div>
      </div>

      <svg class="ray-visualization" v-if="simulationResults && simulationResults.results && simulationResults.results.rays_out">
        <line 
          v-for="(ray, index) in simulationResults.results.rays_out" 
          :key="index"
          :x1="ray.x1" 
          :y1="ray.y1" 
          :x2="ray.x2" 
          :y2="ray.y2" 
          :stroke="ray.color || 'rgba(255, 255, 100, 0.8)'" 
          stroke-width="2" 
          stroke-linecap="round"
          :opacity="ray.int > 0.1 ? 1 : 0.5"
        />
      </svg>

      <!-- Info overlay -->
      <div class="info-overlay" v-if="simulationResults && simulationResults.results">
        <div class="info-item">
          <strong>Ray Segments:</strong> {{ simulationResults.results.statistics?.total_ray_segments || 0 }}
        </div>
        <div class="info-item">
          <strong>Reflections:</strong> {{ simulationResults.results.statistics?.total_reflections || 0 }}
        </div>
        <div class="info-item">
          <strong>Refractions:</strong> {{ simulationResults.results.statistics?.total_refractions || 0 }}
        </div>
      </div>
    </main>

    <aside class="controls-panel">
      <h2>Properties & Controls</h2>
      
      <div v-if="selectedComponent">
        <h3>{{ selectedComponent.type }} Properties</h3>
        <label>ID: {{ selectedComponent.id }}</label><br/>
        <label>X: <input type="number" v-model.number="selectedComponent.x" /></label><br/>
        <label>Y: <input type="number" v-model.number="selectedComponent.y" /></label><br/>
        <label>Angle: <input type="number" v-model.number="selectedComponent.angle" step="0.1" /></label><br/>
        
        <div v-if="selectedComponent.type === 'Source'">
          <label>Wavelength (nm): <input type="number" v-model.number="selectedComponent.properties.wavelength" /></label>
          <label>Intensity: <input type="number" step="0.1" min="0" max="10" v-model.number="selectedComponent.properties.intensity" /></label>
          <label>Spread (deg): <input type="number" v-model.number="selectedComponent.properties.spread_deg" /></label>
        </div>
        <div v-if="selectedComponent.type === 'Mirror'">
          <label>Reflectivity: <input type="number" step="0.01" min="0" max="1" v-model.number="selectedComponent.properties.reflectivity" /></label>
          <label>Diameter: <input type="number" v-model.number="selectedComponent.properties.diameter" /></label>
        </div>
        <div v-if="selectedComponent.type === 'Lens'">
          <label>Focal Length: <input type="number" v-model.number="selectedComponent.properties.focalLength" /></label>
          <label>Diameter: <input type="number" v-model.number="selectedComponent.properties.diameter" /></label>
          <label>Transmission: <input type="number" step="0.01" min="0" max="1" v-model.number="selectedComponent.properties.transmission" /></label>
        </div>
        <div v-if="selectedComponent.type === 'Beam Splitter'">
          <label>Split Ratio: <input type="number" step="0.01" min="0" max="1" v-model.number="selectedComponent.properties.splitRatio" /></label>
          <label>Diameter: <input type="number" v-model.number="selectedComponent.properties.diameter" /></label>
        </div>
        <div v-if="selectedComponent.type === 'Photo Detector'">
          <label>Sensitivity: <input type="number" step="0.01" min="0" max="1" v-model.number="selectedComponent.properties.sensitivity" /></label>
          <label>Diameter: <input type="number" v-model.number="selectedComponent.properties.diameter" /></label>
        </div>

        <button @click="deleteSelectedComponent" class="delete-btn">Delete Component</button>
        <hr />
      </div>

      <h3>Global Controls</h3>
      <button @click="handleDownloadJSON" class="action-btn">Download JSON</button>
      <button @click="handleRunSimulation" class="action-btn simulate-btn">Run Simulation</button>
      
      <div v-if="simulationResults && simulationResults.results" class="simulation-results">
        <h3>📊 Simulation Results</h3>
        
        <div class="result-section">
          <h4>Statistics</h4>
          <div class="stat-grid">
            <div class="stat-item">
              <span class="stat-label">Path Length:</span>
              <span class="stat-value">{{ simulationResults.results.statistics?.total_path_length_mm.toFixed(1) || 0 }} px</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">Ray Segments:</span>
              <span class="stat-value">{{ simulationResults.results.statistics?.total_ray_segments || 0 }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">Reflections:</span>
              <span class="stat-value">{{ simulationResults.results.statistics?.total_reflections || 0 }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">Refractions:</span>
              <span class="stat-value">{{ simulationResults.results.statistics?.total_refractions || 0 }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">Beam Splits:</span>
              <span class="stat-value">{{ simulationResults.results.statistics?.total_splits || 0 }}</span>
            </div>
          </div>
        </div>

        <div class="result-section" v-if="Object.keys(simulationResults.results.detectors || {}).length > 0">
          <h4>🔴 Detectors</h4>
          <div v-for="(value, key) in simulationResults.results.detectors" :key="key" class="detector-reading">
            <span class="detector-label">{{ key.split('-')[0] }} {{ key.split('-')[1] }}:</span>
            <div class="detector-bar-container">
              <div class="detector-bar" :style="{ width: `${Math.min(value * 100, 100)}%` }"></div>
              <span class="detector-value">{{ value.toFixed(3) }}</span>
            </div>
          </div>
        </div>

        <div class="result-section" v-if="simulationResults.results.statistics?.component_interactions">
          <h4>🎯 Component Interactions</h4>
          <div v-for="(data, compId) in simulationResults.results.statistics.component_interactions" :key="compId" class="interaction-item">
            <span class="interaction-label">{{ data.type }}:</span>
            <span class="interaction-count">{{ data.interactions }}x</span>
          </div>
        </div>
      </div>
        
    </aside>
  </div>
</template>

<script setup>
import { ref, computed, nextTick, watch, onBeforeUnmount } from 'vue';

// --- IMAGE IMPORTS ---
import laserSourceImage from '@/assets/laser-source.png';
import planarMirrorImage from '@/assets/planar-mirror.png';
import convexLensImage from '@/assets/convex-lens.png';
import beamSplitterImage from '@/assets/beam-splitter.png';
import photoDetectorImage from '@/assets/photo-detector.png';

// --- REF & STATE INITIALIZATION ---
const components = ref([]);
const selectedComponentId = ref(null);
const simulationResults = ref(null); 

// Drag/Rotate State
let isDragging = false;
let isRotating = false;
let draggingComponentId = null;
let startAngle = 0;
const gridAreaOffset = ref({ left: 0, top: 0 });

// Constants
const BACKEND_URL = "http://localhost:5000/simulate";
const GRID_SIZE = 20;

// --- COMPUTED PROPERTIES ---
const selectedComponent = computed(() => 
  components.value.find(comp => comp.id === selectedComponentId.value)
);

const simulationInputData = computed(() => {
    const dataToSimulate = components.value.map(({ id, x, y, angle, properties }) => ({
        id, x, y, angle, properties: JSON.stringify(properties)
    }));
    return JSON.stringify(dataToSimulate);
});

// --- HELPER FUNCTIONS ---
const getInteractionCount = (compId) => {
  if (!simulationResults.value?.results?.statistics?.component_interactions) return 0;
  return simulationResults.value.results.statistics.component_interactions[compId]?.interactions || 0;
};

// --- DRAG-AND-DROP FROM PALETTE TO GRID ---
const generateUniqueId = (type) => `${type.toLowerCase().replace(/\s+/g, '-')}-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;

const handleDragStart = (event, type, imageUrlVariable) => {
    event.dataTransfer.setData('componentType', type);
    event.dataTransfer.setData('componentImageUrl', imageUrlVariable); 
    event.dataTransfer.effectAllowed = 'copy';
};

const handleDragOver = (event) => {
    event.preventDefault();
    event.dataTransfer.dropEffect = 'copy';
};

const handleDrop = (event) => {
    event.preventDefault();
    const type = event.dataTransfer.getData('componentType');
    const imageUrl = event.dataTransfer.getData('componentImageUrl');

    if (!type) return;

    const gridArea = event.currentTarget.getBoundingClientRect();
    let x = event.clientX - gridArea.left;
    let y = event.clientY - gridArea.top;

    x = Math.round(x / GRID_SIZE) * GRID_SIZE;
    y = Math.round(y / GRID_SIZE) * GRID_SIZE;

    const newComponent = {
        id: generateUniqueId(type),
        type: type,
        x: x,
        y: y,
        angle: 0,
        imageUrl: imageUrl,
        properties: {}
    };

    // Set default properties
    if (type === 'Source') {
        newComponent.properties = { wavelength: 632, intensity: 1.0, spread_deg: 0 };
    } else if (type === 'Mirror') {
        newComponent.properties = { reflectivity: 0.95, diameter: 50 };
    } else if (type === 'Lens') {
        newComponent.properties = { focalLength: 100, diameter: 50, transmission: 0.98 };
    } else if (type === 'Beam Splitter') {
        newComponent.properties = { splitRatio: 0.5, diameter: 50 };
    } else if (type === 'Photo Detector') {
        newComponent.properties = { sensitivity: 1.0, diameter: 50 };
    }
    
    components.value.push(newComponent);
    selectedComponentId.value = newComponent.id;
    handleRunSimulation();
};

// --- DRAGGING LOGIC ---
const startDragComponent = (id, event) => {
    isDragging = true;
    draggingComponentId = id;
    selectedComponentId.value = id;
    const comp = components.value.find(c => c.id === id);
    if (!comp) return;

    const gridArea = event.currentTarget.parentElement.getBoundingClientRect();
    gridAreaOffset.value.left = gridArea.left;
    gridAreaOffset.value.top = gridArea.top;
    
    const compRect = event.currentTarget.getBoundingClientRect();
    comp.offsetX = event.clientX - compRect.left;
    comp.offsetY = event.clientY - compRect.top;

    window.addEventListener('mousemove', dragComponentMove);
    window.addEventListener('mouseup', stopDragComponent);
};

const dragComponentMove = (event) => {
    if (!isDragging || !draggingComponentId) return;
    const comp = components.value.find(c => c.id === draggingComponentId);
    if (!comp) return;

    let newX = event.clientX - gridAreaOffset.value.left - comp.offsetX; 
    let newY = event.clientY - gridAreaOffset.value.top - comp.offsetY;

    newX = Math.round(newX / GRID_SIZE) * GRID_SIZE;
    newY = Math.round(newY / GRID_SIZE) * GRID_SIZE;

    comp.x = newX;
    comp.y = newY;
};

const stopDragComponent = () => {
    isDragging = false;
    draggingComponentId = null;
    window.removeEventListener('mousemove', dragComponentMove);
    window.removeEventListener('mouseup', stopDragComponent);
    handleRunSimulation(); 
};

// --- ROTATION LOGIC ---
const startRotateComponent = (id, event) => {
    isRotating = true;
    draggingComponentId = id;
    selectedComponentId.value = id;
    const comp = components.value.find(c => c.id === id);
    if (!comp) return;

    const compRect = event.currentTarget.closest('.grid-component').getBoundingClientRect();
    const centerX = compRect.left + compRect.width / 2;
    const centerY = compRect.top + compRect.height / 2;

    const dx = event.clientX - centerX;
    const dy = event.clientY - centerY;
    startAngle = Math.atan2(dy, dx) * (180 / Math.PI);

    window.addEventListener('mousemove', rotateComponentMove);
    window.addEventListener('mouseup', stopRotateComponent);
};

const rotateComponentMove = (event) => {
    if (!isRotating || !draggingComponentId) return;
    const comp = components.value.find(c => c.id === draggingComponentId);
    if (!comp) return;

    const compElement = document.querySelector(`.grid-component[data-id="${comp.id}"]`); 
    if (!compElement) return;

    const compRect = compElement.getBoundingClientRect();
    const centerX = compRect.left + compRect.width / 2;
    const centerY = compRect.top + compRect.height / 2;

    const dx = event.clientX - centerX;
    const dy = event.clientY - centerY;
    const currentAngle = Math.atan2(dy, dx) * (180 / Math.PI);

    comp.angle = (comp.angle + (currentAngle - startAngle));
    comp.angle = (comp.angle % 360 + 360) % 360;
    startAngle = currentAngle;
};

const stopRotateComponent = () => {
    isRotating = false;
    draggingComponentId = null;
    window.removeEventListener('mousemove', rotateComponentMove);
    window.removeEventListener('mouseup', stopRotateComponent);
    handleRunSimulation();
};

// --- UI LOGIC ---
const selectComponent = (id) => {
    selectedComponentId.value = id;
};

const deleteSelectedComponent = () => {
    if (selectedComponentId.value) {
        components.value = components.value.filter(comp => comp.id !== selectedComponentId.value);
        selectedComponentId.value = null;
        handleRunSimulation();
    }
};

// --- SIMULATION ---
const handleRunSimulation = async () => {
    if (components.value.length === 0) {
        simulationResults.value = null;
        return;
    }

    try {
        const dataToSend = components.value.map(({ id, type, x, y, angle, properties }) => ({
            id, type, x, y, angle, properties
        }));

        const response = await fetch(BACKEND_URL, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(dataToSend),
        });

        if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
        }

        const data = await response.json();
        simulationResults.value = data; 
        
    } catch (error) {
        console.error("Error running simulation:", error);
        simulationResults.value = null;
    }
};

// Auto-run simulation on property changes
watch(simulationInputData, (newVal, oldVal) => {
    if (newVal !== oldVal) {
        if (window.simulationTimeout) {
            clearTimeout(window.simulationTimeout);
        }
        window.simulationTimeout = setTimeout(() => {
            if (!isDragging && !isRotating) {
               handleRunSimulation();
            }
        }, 300); 
    }
}, { deep: false });

// --- UTILITY ---
const handleDownloadJSON = () => {
    const dataToExport = {
        version: '1.0',
        timestamp: new Date().toISOString(),
        components: components.value.map(({ id, type, x, y, angle, properties }) => ({
            id, type, x, y, angle, properties
        })),
        simulationResults: simulationResults.value
    };
    
    const dataStr = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(dataToExport, null, 2));
    const downloadAnchorNode = document.createElement('a');
    
    const now = new Date().toISOString().replace(/[:.]/g, '-');
    downloadAnchorNode.setAttribute("href", dataStr);
    downloadAnchorNode.setAttribute("download", `optical-setup_${now}.json`);
    
    document.body.appendChild(downloadAnchorNode);
    nextTick(() => { 
        downloadAnchorNode.click();
        downloadAnchorNode.remove();
    });
};

onBeforeUnmount(() => {
    window.removeEventListener('mousemove', dragComponentMove);
    window.removeEventListener('mouseup', stopDragComponent);
    window.removeEventListener('mousemove', rotateComponentMove);
    window.removeEventListener('mouseup', stopRotateComponent);
    if (window.simulationTimeout) {
         clearTimeout(window.simulationTimeout);
    }
});

nextTick(() => {
    if (components.value.length > 0) {
        handleRunSimulation();
    }
});
</script>

<style scoped>

.designer-layout {
  display: flex;
  height: 100vh;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background-color: #141414;
}

.component-palette {
  width: 200px;
  background-color: #030303;
  border-right: 1px solid #444;
  padding: 15px;
  overflow-y: auto;
  color: white;
}

.component-palette h2 {
  color: #fff;
  border-bottom: 1px solid #444;
  padding-bottom: 10px;
  margin-bottom: 15px;
  font-size: 1.1em;
}

.palette-item {
  display: flex;
  align-items: center;
  padding: 10px;
  border: 1px solid #333;
  border-radius: 8px;
  background-color: #222;
  cursor: grab;
  color: #fff;
  transition: background-color 0.2s, transform 0.1s;
  margin-bottom: 10px;
}

.palette-item:hover {
  background-color: #35495e;
  transform: translateX(3px);
}

.palette-item:active {
  cursor: grabbing;
}

.palette-item img {
  width: 32px;
  height: 32px;
  margin-right: 10px;
  object-fit: contain;
}

.grid-area {
  flex-grow: 1;
  position: relative;
  background-color: #0a0a0a; 
  background-image: 
    linear-gradient(to right, #333 1px, transparent 1px),
    linear-gradient(to bottom, #333 1px, transparent 1px);
  background-size: 20px 20px;
  cursor: default;
  overflow: hidden;
}

.grid-component {
  position: absolute;
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  border: 2px solid transparent; 
  border-radius: 8px;
  cursor: grab;
  user-select: none;
  transition: border-color 0.1s, transform 0.05s;
}

.grid-component:active {
  cursor: grabbing;
}

.grid-component.selected {
  border-color: #1890ff;
  box-shadow: 0 0 15px rgba(24, 144, 255, 0.6);
}

.component-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  pointer-events: none;
}

.component-placeholder {
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #444;
  border-radius: 5px;
  color: white;
  font-size: 1.5em;
  font-weight: bold;
}

.interaction-badge {
  position: absolute;
  top: -8px;
  right: -8px;
  background-color: #ff4757;
  color: white;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.7em;
  font-weight: bold;
  border: 2px solid #0a0a0a;
  z-index: 102;
}

.rotation-handle {
  position: absolute;
  width: 15px;
  height: 15px;
  background-color: #1890ff;
  border-radius: 50%;
  border: 1px solid white;
  bottom: -5px; 
  right: -5px;
  cursor: ew-resize;
  z-index: 101;
  transition: background-color 0.2s, transform 0.2s;
}

.rotation-handle:hover {
  background-color: #40a9ff;
  transform: scale(1.2);
}

.ray-visualization {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 5;
}

.info-overlay {
  position: absolute;
  top: 10px;
  left: 10px;
  background-color: rgba(0, 0, 0, 0.8);
  border: 1px solid #444;
  border-radius: 8px;
  padding: 10px 15px;
  color: white;
  font-size: 0.85em;
  pointer-events: none;
  z-index: 100;
}

.info-item {
  margin: 3px 0;
}

.controls-panel {
  width: 300px;
  background-color: #0d0d0d;
  border-left: 1px solid #444;
  padding: 15px;
  overflow-y: auto;
  color: white;
}

.controls-panel h2, .controls-panel h3 {
  color: #fff;
  margin-top: 0;
}

.controls-panel h3 {
  margin-top: 15px;
  margin-bottom: 10px;
  font-size: 1em;
  border-bottom: 1px solid #444;
  padding-bottom: 5px;
}

.controls-panel h4 {
  color: #1890ff;
  margin-top: 10px;
  margin-bottom: 8px;
  font-size: 0.9em;
}

.controls-panel label {
  display: block;
  margin-bottom: 8px;
  font-size: 0.9em;
  color: #ccc;
}

.controls-panel input[type="number"] {
  width: calc(100% - 10px);
  padding: 6px;
  border: 1px solid #555;
  background-color: #222;
  color: white;
  border-radius: 4px;
  margin-top: 4px;
  font-size: 0.9em;
}

.controls-panel input[type="number"]:focus {
  outline: none;
  border-color: #1890ff;
}

.controls-panel button {
  width: 100%;
  padding: 10px 15px;
  margin-bottom: 10px;
  background-color: #1890ff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.2s, transform 0.1s;
  font-weight: 500;
}

.controls-panel button:hover {
  background-color: #40a9ff;
  transform: translateY(-1px);
}

.controls-panel button:active {
  transform: translateY(0);
}

.delete-btn {
  background-color: #ff4757 !important;
}

.delete-btn:hover {
  background-color: #ff6b81 !important;
}

.action-btn {
  background-color: #2ecc71;
}

.action-btn:hover {
  background-color: #27ae60;
}

.simulate-btn {
  background-color: #9b59b6;
}

.simulate-btn:hover {
  background-color: #8e44ad;
}

.simulation-results {
  background-color: #1a1a1a;
  border: 1px solid #444;
  border-radius: 8px;
  padding: 12px;
  margin-top: 20px;
}

.result-section {
  margin-bottom: 15px;
}

.stat-grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 8px;
  margin-top: 8px;
}

.stat-item {
  display: flex;
  justify-content: space-between;
  padding: 6px 8px;
  background-color: #222;
  border-radius: 4px;
  font-size: 0.85em;
}

.stat-label {
  color: #aaa;
}

.stat-value {
  color: #1890ff;
  font-weight: bold;
}

.detector-reading {
  margin: 8px 0;
}

.detector-label {
  display: block;
  font-size: 0.85em;
  color: #aaa;
  margin-bottom: 4px;
}

.detector-bar-container {
  position: relative;
  height: 24px;
  background-color: #222;
  border-radius: 4px;
  overflow: hidden;
}

.detector-bar {
  height: 100%;
  background: linear-gradient(90deg, #e74c3c, #f39c12);
  transition: width 0.3s ease;
}

.detector-value {
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
  font-size: 0.8em;
  color: white;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.8);
}

.interaction-item {
  display: flex;
  justify-content: space-between;
  padding: 6px 8px;
  background-color: #222;
  border-radius: 4px;
  font-size: 0.85em;
  margin: 4px 0;
}

.interaction-label {
  color: #aaa;
}

.interaction-count {
  color: #2ecc71;
  font-weight: bold;
}

/* Scrollbar styling */
.component-palette::-webkit-scrollbar,
.controls-panel::-webkit-scrollbar {
  width: 8px;
}

.component-palette::-webkit-scrollbar-track,
.controls-panel::-webkit-scrollbar-track {
  background: #1a1a1a;
}

.component-palette::-webkit-scrollbar-thumb,
.controls-panel::-webkit-scrollbar-thumb {
  background: #444;
  border-radius: 4px;
}

.component-palette::-webkit-scrollbar-thumb:hover,
.controls-panel::-webkit-scrollbar-thumb:hover {
  background: #555;
}

hr {
  border: none;
  border-top: 1px solid #444;
  margin: 15px 0;
}

</style>