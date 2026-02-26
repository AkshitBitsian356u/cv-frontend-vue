<template>
  <div class="quick-mobile">
    <div class="quick-btn-mobile">
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-save-online"
              title="Save Online"
              @click="saveOnline"
          ></button>
      </div>
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-save"
              title="Save Offline"
              @click="saveOffline"
          ></button>
      </div>
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-delete"
              title="Delete Selected"
              @click="deleteSelectedItem"
          ></button>
      </div>
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-download"
              title="Download as Image"
              @click="createSaveAsImgPrompt"
          ></button>
      </div>
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-zoom-fit"
              title="Fit to Screen"
              @click="zoomToFit"
          ></button>
      </div>
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-undo"
              title="Undo"
              @click="undoit"
          ></button>
      </div>
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-redo"
              title="Redo"
              @click="redoit"
          ></button>
      </div>
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-view"
              title="Preview Circuit"
              @click="view"
          >
          <i :style="{ color: '#ddd', transform: simulatorMobileStore.showMobileView ? 'scale(1)' : 'scale(1.25)' }" class="fas fa-expand-arrows-alt"></i>
          </button>
      </div>
      <div class="btn-container">
          <button
              type="button"
              class="quick-btn-timing"
              @mousedown="simulatorMobileStore.showTimingDiagram = !simulatorMobileStore.showTimingDiagram"
          >
          <i :style="{ transform: simulatorMobileStore.showMobileView ? 'scale(1)' : 'scale(1.25)' }" class="fa-solid fa-timeline"></i>
          </button>
    </div>
    <nav class="navbar mobile-nav navbar-expand-lg navbar-dark">
      <Hamburger2 v-if="simulatorMobileStore.showMobileView" :navbar-data="navbarData" />
    </nav>
    </div>
    <!-- Zoom slider: 1-100% mapped to internal zoom scale via setZoomFromSlider API -->
    <div class="slider-container">
      <div class="zoom-controls">
          <button class="zoom-button-decrement" @click="decrementZoom" title="Zoom Out">−</button>
          <input 
            type="range" 
            v-model.number="displayZoomPercent"
            @input="onZoomSliderInput"
            min="1" 
            max="100" 
            step="1"
            class="zoom-slider"
            title="Zoom Level"
          />
          <button class="zoom-button-increment" @click="incrementZoom" title="Zoom In">+</button>
          <span class="zoom-label">{{ displayZoomPercent }}%</span>
      </div>
  </div>
  </div>
  <div id="exitView"></div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'
import Hamburger2 from '../Hamburger/Hamburger2.vue'
import navbarData from '../../../assets/constants/Navbar/NAVBAR_DATA.json'
import { useSimulatorMobileStore } from '../../../store/simulatorMobileStore'
import { saveOnline, saveOffline, deleteSelectedItem, createSaveAsImgPrompt, zoomToFit, undoit, redoit, view } from './QuickButton'
// @ts-ignore - simulator functions are not typed
import { setZoomFromSlider, getZoomSliderValue } from '../../../simulator/src/listeners'

const simulatorMobileStore = useSimulatorMobileStore()

// Display zoom constants (user-facing percentage values)
const DISPLAY_ZOOM_MIN = 1
const DISPLAY_ZOOM_MAX = 100
const DISPLAY_ZOOM_DEFAULT = 50
const DISPLAY_ZOOM_STEP = 5

// Internal zoom constants (simulator scale range)
const INTERNAL_ZOOM_MIN = 0
const INTERNAL_ZOOM_MAX = 200

// Current zoom level displayed to user (1-100%)
const displayZoomPercent = ref(DISPLAY_ZOOM_DEFAULT)

/**
 * Clamp zoom percentage to valid display range
 */
const clampZoomPercent = (value: number): number => {
  return Math.max(DISPLAY_ZOOM_MIN, Math.min(DISPLAY_ZOOM_MAX, value))
}

/**
 * Convert internal zoom value (0-200) to display percentage (1-100%)
 */
const convertInternalToDisplayZoom = (internalValue: number): number => {
  return Math.round((internalValue / INTERNAL_ZOOM_MAX) * 100)
}

/**
 * Convert display percentage (1-100%) to internal zoom value (0-200)
 */
const convertDisplayToInternalZoom = (displayPercent: number): number => {
  return (displayPercent / 100) * INTERNAL_ZOOM_MAX
}

/**
 * Initialize zoom slider with current simulator zoom level
 */
const initializeZoomSlider = () => {
  try {
    const currentInternalZoom = getZoomSliderValue(INTERNAL_ZOOM_MIN, INTERNAL_ZOOM_MAX)
    const currentDisplayZoom = convertInternalToDisplayZoom(currentInternalZoom)
    displayZoomPercent.value = clampZoomPercent(currentDisplayZoom)
  } catch (error) {
    console.warn('Could not initialize zoom slider:', error)
    displayZoomPercent.value = DISPLAY_ZOOM_DEFAULT
  }
}

/**
 * Update simulator zoom based on current display percentage
 */
const updateSimulatorZoom = () => {
  try {
    const internalZoomValue = convertDisplayToInternalZoom(displayZoomPercent.value)
    setZoomFromSlider(internalZoomValue, INTERNAL_ZOOM_MIN, INTERNAL_ZOOM_MAX)
  } catch (error) {
    console.warn('Could not apply zoom:', error)
  }
}

/**
 * Increase zoom level by one step
 */
const incrementZoom = () => {
  displayZoomPercent.value = clampZoomPercent(displayZoomPercent.value + DISPLAY_ZOOM_STEP)
  updateSimulatorZoom()
}

/**
 * Decrease zoom level by one step
 */
const decrementZoom = () => {
  displayZoomPercent.value = clampZoomPercent(displayZoomPercent.value - DISPLAY_ZOOM_STEP)
  updateSimulatorZoom()
}

/**
 * Handle slider input event - updates zoom in real-time as user drags
 */
const onZoomSliderInput = () => {
  updateSimulatorZoom()
}

onMounted(initializeZoomSlider)
</script>

<style scoped>
/* @import url('./QuickButton.css'); */

.mobile-nav {
  padding: 0 0 !important;
}

.slider-container {
  display: flex;
  justify-content: center;
}

.quick-btn-mobile {
  background: var(--bg-navbar);
  border-top: 1.5px solid var(--qp-br-tl);
  border-left: 1.5px solid var(--qp-br-tl);
  border-right: 1.5px solid var(--qp-br-rd);
  border-bottom: 1.5px solid var(--qp-br-rd);
}

.quick-btn-mobile {
  display: flex;
  align-items: center;
  z-index: 99;
  justify-content: space-between;
  padding: 0 1.5rem;
}

.quick-btn-mobile > div > button {
  border: none;
}

.quick-mobile {
  display: flex;
  flex-direction: column;
  background: var(--bg-navbar);
}

.btn-container {
  padding: 0.75rem 0;
}

.quick-btn-view, .quick-btn-timing {
  color: white;
}

/* Zoom controls with slider (1-100% display) */
.zoom-controls {
  display: flex;
  gap: 1rem;
  justify-content: center;
  align-items: center;
  padding: 0.5rem 0;
}

.zoom-button-decrement,
.zoom-button-increment {
  background: transparent;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 24px;
  font-weight: bold;
  padding: 0 8px;
  line-height: 1;
  min-width: 32px;
}

.zoom-button-decrement:hover,
.zoom-button-increment:hover {
  opacity: 0.7;
}

.zoom-slider {
  width: 120px;
  height: 6px;
  -webkit-appearance: none;
  appearance: none;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 3px;
  outline: none;
  cursor: pointer;
}

.zoom-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 16px;
  height: 16px;
  background: white;
  border-radius: 50%;
  cursor: pointer;
  transition: background 0.15s ease-in-out;
}

.zoom-slider::-webkit-slider-thumb:hover {
  background: #ddd;
}

.zoom-slider::-moz-range-thumb {
  width: 16px;
  height: 16px;
  background: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  transition: background 0.15s ease-in-out;
}

.zoom-slider::-moz-range-thumb:hover {
  background: #ddd;
}

.zoom-label {
  color: white;
  font-size: 14px;
  font-weight: 500;
  min-width: 45px;
  text-align: right;
}

@media (max-width: 768px) {
  .quick-btn-mobile {
    padding: 0 0.75rem;
  }
}
</style>
