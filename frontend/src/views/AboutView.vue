<template>
  <div class="about-container">
    <header class="about-header">
      <h1>🔬 About the Optical Setup Designer</h1>
      <p>A simple, interactive tool for visualizing the principles of geometric optics.</p>
    </header>

    <section class="about-section features">
      <h2>✨ Key Features</h2>
      <ul>
        <li><b>Interactive Design Canvas:</b> Drag-and-drop optical components onto a grid.</li>
        <li><b>Real-Time Simulation:</b> Automatically runs the ray tracing algorithm when components are moved or properties are changed.</li>
        <li><b>Geometric Optics:</b> Accurate simulation of fundamental light behaviors:
          <ul>
            <li><b>Reflection</b> (Planar Mirrors)</li>
            <li><b>Refraction</b> (Thin Lens Approximation)</li>
            <li><b>Splitting </b> (Beam Splitters)</li>
          </ul>
        </li>
        <li><b>Intensity Tracking: </b> Calculates light intensity loss and measures total intensity at Photo Detectors.</li>
      </ul>
    </section>

    <hr>

    <section class="about-section technology">
      <h2>💻 Technology Stack</h2>
      <p>This application is built using a modern, decoupled architecture:</p>
      <div class="tech-list">
        <div class="tech-item">
          <strong>Frontend (Interface):</strong> Vue 3 (Composition API) + HTML/CSS
        </div>
        <div class="tech-item">
          <strong>Backend (Simulation Logic):</strong> Flask (Python)
        </div>
        <div class="tech-item">
          <strong>Communication:</strong> RESTful API calls for transmitting component data and receiving ray segments.
        </div>
      </div>
    </section>

    <hr>

  </div>
</template>

<style scoped>
/* Scoped styles ensure these styles only apply to this component */
.about-container {
  max-width: 900px;
  margin: 40px auto;
  padding: 30px;
  background-color: #1a1a1a; /* Dark background */
  color: #f0f0f0; /* Light text */
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.5);
  font-family: Arial, sans-serif;
  min-height: calc(100vh - 80px); /* Fill most of the viewport */
}

.about-header {
  text-align: center;
  margin-bottom: 40px;
}

.about-header h1 {
  color: #1890ff; /* Bright blue accent */
  font-size: 2.5em;
  margin-bottom: 10px;
}

.about-header p {
  font-size: 1.1em;
  color: #ccc;
}

.about-section {
  margin-bottom: 30px;
  padding: 20px;
  border-left: 5px solid #333;
}

.about-section h2 {
  color: #fff;
  font-size: 1.8em;
  margin-bottom: 15px;
}

.about-section ul {
  list-style: none;
  padding-left: 0;
}

.about-section ul li {
  margin-bottom: 12px;
  line-height: 1.6;
  font-size: 1em;
  color: #ddd;
}

.about-section ul li strong {
  color: #1890ff;
}

hr {
  border: 0;
  height: 1px;
  background-color: #333;
  margin: 30px 0;
}

.tech-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.tech-item {
  background-color: #222;
  padding: 10px 15px;
  border-radius: 6px;
  border: 1px solid #333;
}

.about-footer {
  text-align: center;
  margin-top: 40px;
  padding-top: 20px;
  border-top: 1px solid #333;
}

.about-footer p {
  font-size: 0.9em;
  color: #aaa;
}
</style>