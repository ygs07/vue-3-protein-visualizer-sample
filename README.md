# 🧬 Protein Visualization Test

This is a sample **Vue 3 + Vite** application built to test and showcase the [`vue-protein-network-visualizer`](https://github.com/ygs07/protein-d3-representation) component library.

The goal of this app is to provide an environment where you can load and visualize **protein network data** (nodes and edges) using D3.js through a reusable Vue component.

---

## 🚀 Features

- Uses **Vue 3 (Composition API)** and **Vite** for fast development.
- Demonstrates how to import and use the `vue-protein-network-visualizer` library.
- Supports dynamic data loading (local JSON or remote sources like S3 or Cloudflare).
- Built-in error handling and loading states for clean data fetching.
- Fully customizable visualization powered by **D3.js**.

---

## 🧩 Example Component Usage

Below is a simplified example of how this app imports and uses the component:

```vue
<script setup>
import { ref, onMounted } from "vue";
import ProteinNetwork from "vue-protein-network-visualizer";

const nodes = ref(null);
const edges = ref(null);
const loading = ref(true);
const error = ref(null);

async function loadData() {
  try {
    const [nodesRes, edgesRes] = await Promise.all([
      fetch("/data/nodes.json"),
      fetch("/data/edges.json"),
    ]);
    nodes.value = await nodesRes.json();
    edges.value = await edgesRes.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
}

onMounted(loadData);
</script>

<template>
  <div>
    <ProteinNetwork v-if="!loading && !error" :nodes="nodes" :edges="edges" />
    <p v-else-if="loading">Loading data...</p>
    <p v-else>Error: {{ error }}</p>
  </div>
</template>
```
