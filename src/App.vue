<script setup>
import { ref, onMounted, computed } from "vue";
import ProteinNetwork from "vue-protein-network-visualizer";

const nodes = ref(null);
const edges = ref(null);
const loading = ref(true);
const error = ref(null);

// Responsive dimensions
const containerWidth = ref(800);
const containerHeight = ref(600);

// Sample data URLs
const nodesUrl =
  "https://ys-portfolio-site.s3.eu-west-1.amazonaws.com/sample_residue_data.json";
const edgesUrl =
  "https://ys-portfolio-site.s3.eu-west-1.amazonaws.com/sample_protein_data.json";

const repoUrl = "https://github.com/ygs07/protein-d3-representation";

// Responsive network dimensions
const networkDimensions = computed(() => {
  const isMobile = window.innerWidth < 768;
  const isTablet = window.innerWidth < 1024;

  if (isMobile) {
    return { width: window.innerWidth - 40, height: 400 };
  } else if (isTablet) {
    return { width: 700, height: 500 };
  } else {
    return { width: 1000, height: 600 };
  }
});

async function loadData() {
  try {
    const [nRes, eRes] = await Promise.all([fetch(nodesUrl), fetch(edgesUrl)]);

    if (!nRes.ok) throw new Error(`Nodes fetch failed: ${nRes.status}`);
    if (!eRes.ok) throw new Error(`Edges fetch failed: ${eRes.status}`);

    nodes.value = await nRes.json();
    edges.value = await eRes.json();
  } catch (err) {
    error.value = err && err.message ? err.message : String(err);
  } finally {
    loading.value = false;
  }
}

// Download functions
function downloadJson(data, filename) {
  const blob = new Blob([JSON.stringify(data, null, 2)], {
    type: "application/json",
  });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

function downloadNodes() {
  if (nodes.value) {
    downloadJson(nodes.value, "sample_residue_data.json");
  }
}

function downloadEdges() {
  if (edges.value) {
    downloadJson(edges.value, "sample_protein_data.json");
  }
}

function downloadBoth() {
  if (nodes.value && edges.value) {
    const combinedData = {
      nodes: nodes.value,
      edges: edges.value,
      description:
        "Sample protein network data for vue-protein-network-visualizer",
      timestamp: new Date().toISOString(),
    };
    downloadJson(combinedData, "protein_network_sample_data.json");
  }
}

// Handle window resize
function handleResize() {
  containerWidth.value = networkDimensions.value.width;
  containerHeight.value = networkDimensions.value.height;
}

onMounted(() => {
  loadData();
  handleResize();
  window.addEventListener("resize", handleResize);
});
</script>

<template>
  <div class="app-container">
    <header class="app-header">
      <div class="header-content">
        <div class="title-section">
          <h1>Protein Network Visualizer</h1>
          <p class="subtitle">
            Interactive visualization of protein residue interaction networks
          </p>
        </div>

        <div class="header-actions">
          <a
            :href="repoUrl"
            target="_blank"
            rel="noopener noreferrer"
            class="repo-link"
          >
            <svg width="16" height="16" viewBox="0 0 16 16" fill="currentColor">
              <path
                d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"
              />
            </svg>
            <span class="repo-text">GitHub</span>
          </a>
        </div>
      </div>
    </header>

    <main class="app-main">
      <section class="visualization-section">
        <div class="section-header">
          <h2>Network Visualization</h2>
          <div class="data-info" v-if="nodes && edges">
            <span class="data-stats">
              {{ nodes.length }} residues • {{ edges.length }} connections
            </span>
          </div>
        </div>

        <div class="visualization-container">
          <div v-if="loading" class="loading-state">
            <div class="spinner"></div>
            <p>Loading protein network data…</p>
          </div>

          <div v-else-if="error" class="error-state">
            <div class="error-icon">⚠️</div>
            <h3>Error loading data</h3>
            <p>{{ error }}</p>
            <button @click="loadData" class="retry-button">Try Again</button>
          </div>

          <div v-else class="network-wrapper">
            <div class="network-container">
              <ProteinNetwork
                :nodeData="nodes"
                :edgeData="edges"
                :width="networkDimensions.width"
                :height="networkDimensions.height"
                :distanceThreshold="8"
                background="#ffffff"
              />
            </div>
          </div>
        </div>
      </section>

      <section class="data-section" v-if="nodes && edges">
        <h2>Sample Data</h2>
        <div class="data-actions">
          <div class="download-buttons">
            <button @click="downloadBoth" class="download-btn primary">
              <svg
                width="16"
                height="16"
                viewBox="0 0 16 16"
                fill="currentColor"
              >
                <path
                  d="M.5 9.9a.5.5 0 01.5.5v2.5a1 1 0 001 1h12a1 1 0 001-1v-2.5a.5.5 0 011 0v2.5a2 2 0 01-2 2H2a2 2 0 01-2-2v-2.5a.5.5 0 01.5-.5z"
                />
                <path
                  d="M7.646 11.854a.5.5 0 00.708 0l3-3a.5.5 0 00-.708-.708L8.5 10.293V1.5a.5.5 0 00-1 0v8.793L5.354 8.146a.5.5 0 10-.708.708l3 3z"
                />
              </svg>
              Download All Data
            </button>

            <div class="download-options">
              <button @click="downloadNodes" class="download-btn secondary">
                <svg
                  width="16"
                  height="16"
                  viewBox="0 0 16 16"
                  fill="currentColor"
                >
                  <path
                    d="M.5 9.9a.5.5 0 01.5.5v2.5a1 1 0 001 1h12a1 1 0 001-1v-2.5a.5.5 0 011 0v2.5a2 2 0 01-2 2H2a2 2 0 01-2-2v-2.5a.5.5 0 01.5-.5z"
                  />
                  <path
                    d="M7.646 11.854a.5.5 0 00.708 0l3-3a.5.5 0 00-.708-.708L8.5 10.293V1.5a.5.5 0 00-1 0v8.793L5.354 8.146a.5.5 0 10-.708.708l3 3z"
                  />
                </svg>
                Residue Data
              </button>

              <button @click="downloadEdges" class="download-btn secondary">
                <svg
                  width="16"
                  height="16"
                  viewBox="0 0 16 16"
                  fill="currentColor"
                >
                  <path
                    d="M.5 9.9a.5.5 0 01.5.5v2.5a1 1 0 001 1h12a1 1 0 001-1v-2.5a.5.5 0 011 0v2.5a2 2 0 01-2 2H2a2 2 0 01-2-2v-2.5a.5.5 0 01.5-.5z"
                  />
                  <path
                    d="M7.646 11.854a.5.5 0 00.708 0l3-3a.5.5 0 00-.708-.708L8.5 10.293V1.5a.5.5 0 00-1 0v8.793L5.354 8.146a.5.5 0 10-.708.708l3 3z"
                  />
                </svg>
                Protein Data
              </button>
            </div>
          </div>

          <div class="data-preview">
            <h3>Data Preview</h3>
            <div class="preview-grid">
              <div class="preview-item">
                <h4>Residue Data</h4>
                <div class="preview-content">
                  <pre>{{ JSON.stringify(nodes.slice(0, 2), null, 2) }}</pre>
                  <small>Showing first 2 of {{ nodes.length }} residues</small>
                </div>
              </div>
              <div class="preview-item">
                <h4>Protein Data</h4>
                <div class="preview-content">
                  <pre>{{ JSON.stringify(edges.slice(0, 2), null, 2) }}</pre>
                  <small
                    >Showing first 2 of {{ edges.length }} connections</small
                  >
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>

    <footer class="app-footer">
      <p>
        Built with <a href="https://vuejs.org/" target="_blank">Vue 3</a> •
        Powered by <a href="https://d3js.org/" target="_blank">D3.js</a>
      </p>
    </footer>
  </div>
</template>

<style scoped>
/* Base styles */
.app-container {
  min-height: 100vh;
  color: #1f2937;
  padding: 16px;
  font-family: "Inter", -apple-system, BlinkMacSystemFont, sans-serif;
}

/* Mobile First - Header */
.app-header {
  color: #1f2937;
  margin-bottom: 32px;
}

.header-content {
  display: flex;
  flex-direction: column;
  gap: 16px;
  align-items: flex-start;
}

.title-section h1 {
  font-size: 1.75rem;
  font-weight: 700;
  margin: 0 0 8px 0;
  -webkit-background-clip: text;
  background-clip: text;
  line-height: 1.2;
}

.subtitle {
  font-size: 0.95rem;
  opacity: 0.9;
  margin: 0;
  color: #000;
  line-height: 1.4;
}

.header-actions {
  width: 100%;
}

.repo-link {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 20px;
  background: rgba(255, 255, 255, 0.1);
  color: black;
  text-decoration: none;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: all 0.3s ease;
  backdrop-filter: blur(10px);
  width: 100%;
  justify-content: center;
}

.repo-link:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-1px);
}

/* Main content */
.app-main {
  max-width: 1200px;
  margin: 0 auto;
}

.visualization-section {
  background: white;
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.section-header {
  display: flex;
  flex-direction: column;
  gap: 12px;
  margin-bottom: 20px;
}

.section-header h2 {
  margin: 0;
  color: #1f2937;
  font-size: 1.25rem;
}

.data-stats {
  background: #f3f4f6;
  padding: 8px 12px;
  border-radius: 6px;
  font-size: 0.85rem;
  color: #6b7280;
  font-weight: 500;
  align-self: flex-start;
}

.visualization-container {
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  overflow: hidden;
  min-height: 300px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.network-container {
  width: 100%;
  overflow: auto;
}

.network-wrapper {
  width: 100%;
  display: flex;
  justify-content: center;
}

/* Loading and Error States */
.loading-state {
  text-align: center;
  padding: 40px 20px;
}

.spinner {
  width: 32px;
  height: 32px;
  border: 3px solid #e5e7eb;
  border-top: 3px solid #3b82f6;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto 16px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.error-state {
  text-align: center;
  padding: 40px 20px;
  color: #dc2626;
}

.error-icon {
  font-size: 2rem;
  margin-bottom: 12px;
}

.error-state h3 {
  margin: 0 0 8px 0;
  font-size: 1.1rem;
}

.error-state p {
  margin: 0 0 16px 0;
  font-size: 0.9rem;
}

.retry-button {
  background: #dc2626;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
}

.retry-button:hover {
  background: #b91c1c;
}

/* Data Section */
.data-section {
  background: white;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.data-section h2 {
  margin: 0 0 20px 0;
  color: #1f2937;
  font-size: 1.25rem;
}

.download-buttons {
  display: flex;
  flex-direction: column;
  gap: 16px;
  margin-bottom: 24px;
}

.download-options {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.download-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 16px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
  text-decoration: none;
  font-size: 0.9rem;
  justify-content: center;
  text-align: center;
}

.download-btn.primary {
  background: #3b82f6;
  color: white;
}

.download-btn.primary:hover {
  background: #2563eb;
  transform: translateY(-1px);
}

.download-btn.secondary {
  background: #f3f4f6;
  color: #374151;
  border: 1px solid #d1d5db;
}

.download-btn.secondary:hover {
  background: #e5e7eb;
  transform: translateY(-1px);
}

/* Data Preview */
.data-preview h3 {
  margin: 0 0 16px 0;
  color: #1f2937;
  font-size: 1.1rem;
}

.preview-grid {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.preview-item {
  border: 1px solid #e5e7eb;
  border-radius: 8px;
  padding: 16px;
  background: #fafafa;
}

.preview-item h4 {
  margin: 0 0 12px 0;
  color: #374151;
  font-size: 0.9rem;
}

.preview-content pre {
  background: white;
  padding: 12px;
  border-radius: 6px;
  font-size: 0.75rem;
  overflow: auto;
  max-height: 150px;
  margin: 0 0 8px 0;
  border: 1px solid #e5e7eb;
  font-family: "Monaco", "Menlo", "Ubuntu Mono", monospace;
}

.preview-content small {
  color: #6b7280;
  font-size: 0.7rem;
}

/* Footer */
.app-footer {
  text-align: center;

  margin-top: 32px;
  opacity: 0.8;
  font-size: 0.9rem;
}

.app-footer a {
  color: rgb(35, 208, 12);
  text-decoration: none;
  font-weight: 500;
}

.app-footer a:hover {
  text-decoration: underline;
}

/* Tablet Styles */
@media (min-width: 768px) {
  .app-container {
    padding: 24px;
  }

  .header-content {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }

  .title-section h1 {
    font-size: 2rem;
  }

  .subtitle {
    font-size: 1rem;
  }

  .header-actions {
    width: auto;
  }

  .repo-link {
    width: auto;
  }

  .repo-text {
    display: inline;
  }

  .section-header {
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }

  .download-options {
    flex-direction: row;
  }

  .preview-grid {
    flex-direction: row;
  }

  .preview-item {
    flex: 1;
  }

  .visualization-section,
  .data-section {
    padding: 24px;
  }
}

/* Desktop Styles */
@media (min-width: 1024px) {
  .app-container {
    padding: 32px;
  }

  .title-section h1 {
    font-size: 2.5rem;
  }

  .section-header h2,
  .data-section h2 {
    font-size: 1.5rem;
  }

  .visualization-section {
    padding: 32px;
  }

  .data-section {
    padding: 32px;
  }

  .preview-content pre {
    font-size: 0.8rem;
    max-height: 200px;
  }
}

/* Large Desktop */
@media (min-width: 1280px) {
  .visualization-container {
    min-height: 400px;
  }
}

/* Touch device optimizations */
@media (hover: none) and (pointer: coarse) {
  .download-btn:hover,
  .repo-link:hover {
    transform: none;
  }

  .download-btn:active,
  .repo-link:active {
    transform: scale(0.98);
  }
}

/* High contrast support */
@media (prefers-contrast: high) {
  .visualization-section,
  .data-section {
    border: 2px solid #000;
  }

  .download-btn.secondary {
    border: 2px solid #000;
  }
}

/* Reduced motion support */
@media (prefers-reduced-motion: reduce) {
  .spinner {
    animation-duration: 2s;
  }

  .download-btn,
  .repo-link {
    transition: none;
  }
}
</style>