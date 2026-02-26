<template>
    <div class="quick-btn" id='quick-btn-id' @ondragover="dragover">
        <div id="dragQPanel" class="panel-drag">
            <!-- <DragSvgDots /> -->
            <div class="drag-dot-svg"></div>
        </div>
        <div>
            <button
                type="button"
                class="quick-btn-save-online"
                title="Save Online"
                @click="saveOnline"
            ></button>
        </div>
        <div>
            <button
                type="button"
                class="quick-btn-save"
                title="Save Offline"
                @click="saveOffline"
            ></button>
        </div>
        <div>
            <button
                type="button"
                class="quick-btn-delete"
                title="Delete Selected"
                @click="deleteSelectedItem"
            ></button>
        </div>
        <div>
            <button
                type="button"
                class="quick-btn-download"
                title="Download as Image"
                @click="createSaveAsImgPrompt"
            ></button>
        </div>
        <div>
            <button
                type="button"
                class="quick-btn-zoom-fit"
                title="Fit to Screen"
                @click="zoomToFit"
            ></button>
        </div>
        <div>
            <button
                type="button"
                class="quick-btn-undo"
                title="Undo"
                @click="undoit"
            ></button>
        </div>
        <div>
            <button
                type="button"
                class="quick-btn-redo"
                title="Redo"
                @click="redoit"
            ></button>
        </div>
        <div>
            <button
                type="button"
                class="quick-btn-view"
                title="Preview Circuit"
                @click="view"
            >
                <i style="color: #ddd" class="fas fa-expand-arrows-alt"></i>
            </button>
        </div>
        <!-- Zoom slider: 1-100% mapped to internal zoom scale via setZoomFromSlider API -->
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
    <div id="exitView"></div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue'
import { saveOnline, saveOffline, deleteSelectedItem, createSaveAsImgPrompt, zoomToFit, undoit, redoit, view } from './QuickButton'
// @ts-ignore - simulator functions are not typed
import { setZoomFromSlider, getZoomSliderValue } from '../../../simulator/src/listeners'

const DISPLAY_ZOOM_MIN = 1
const DISPLAY_ZOOM_MAX = 100
const DISPLAY_ZOOM_DEFAULT = 50
const DISPLAY_ZOOM_STEP = 5

const INTERNAL_ZOOM_MIN = 0
const INTERNAL_ZOOM_MAX = 200

const displayZoomPercent = ref(100)

const clampZoomPercent = (value: number): number => {
    return Math.max(DISPLAY_ZOOM_MIN, Math.min(DISPLAY_ZOOM_MAX, value))
}

const convertInternalToDisplayZoom = (internalValue: number): number => {
    return Math.round((internalValue / INTERNAL_ZOOM_MAX) * 100)
}

const convertDisplayToInternalZoom = (displayPercent: number): number => {
    return (displayPercent / 100) * INTERNAL_ZOOM_MAX
}

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

const updateSimulatorZoom = () => {
    try {
        const internalZoomValue = convertDisplayToInternalZoom(displayZoomPercent.value)
        setZoomFromSlider(internalZoomValue, INTERNAL_ZOOM_MIN, INTERNAL_ZOOM_MAX)
    } catch (error) {
        console.warn('Could not apply zoom:', error)
    }
}

const incrementZoom = () => {
    displayZoomPercent.value = clampZoomPercent(displayZoomPercent.value + DISPLAY_ZOOM_STEP)
    updateSimulatorZoom()
}

const decrementZoom = () => {
    displayZoomPercent.value = clampZoomPercent(displayZoomPercent.value - DISPLAY_ZOOM_STEP)
    updateSimulatorZoom()
}

const onZoomSliderInput = () => {
    updateSimulatorZoom()
}

onMounted(initializeZoomSlider)

function dragover(): void {
    const quickBtn: HTMLElement | null = document.querySelector('.quick-btn')
    if (quickBtn) {
        quickBtn.style.boxShadow = 'none'
        quickBtn.style.background = 'var(--bg-navbar)'
    }
}
</script>

<style>
/* @import url('./QuickButton.css'); */

.panel-drag {
    cursor: move;
    opacity: 0.6;
    /* display: grid;
  grid-template-columns: 1fr 1fr;
  grid-template-rows: 1fr 1fr 1fr 1fr 1fr 1fr;
  padding: 6px 0;
  width: 1px; */
}
.quick-btn {
    background: var(--bg-navbar);
    border-top: 1.5px solid var(--qp-br-tl);
    border-left: 1.5px solid var(--qp-br-tl);
    border-right: 1.5px solid var(--qp-br-rd);
    border-bottom: 1.5px solid var(--qp-br-rd);
}

.quick-btn {
    display: flex;
    position: absolute;
    width: 400px;
    height: 33px;
    top: 90px;
    right: 280px;
    border-radius: 7px;
    z-index: 100;
}

.quick-btn > div {
    margin: auto;
}

.quick-btn > div > button {
    margin: auto;
    border: none;
}

.drag-dot-svg {
    width: 12px;
    height: 20px;
    background: url(../../../styles/css/assets/shorcuts/dragDots.svg)
        center/contain;
    display: block;
    margin-left: 4px;
}

.quick-btn-save-online {
    background: url(../../../styles/css/assets/shorcuts/save-online.svg)
        center/cover;
    width: 21.43px;
    height: 15.2px;
    display: block;
}

.quick-btn-save {
    background: url(../../../styles/css/assets/shorcuts/save.svg) center/cover;
    width: 15.2px;
    height: 15.2px;
    display: block;
}

.quick-btn-delete {
    background: url(../../../styles/css/assets/shorcuts/delete.svg) center/cover;
    width: 20px;
    height: 15.2px;
    display: block;
}

.quick-btn-download {
    background: url(../../../styles/css/assets/shorcuts/download.svg)
        center/cover;
    width: 15.2px;
    height: 15.2px;
    display: block;
}

.quick-btn-zoom-fit {
    background: url(../../../styles/css/assets/shorcuts/fit.svg) center/cover;
    width: 15.2px;
    height: 15.2px;
    display: block;
}

.quick-btn-undo {
    background: url(../../../styles/css/assets/shorcuts/undo.svg) center/cover;
    width: 15.2px;
    height: 16.2px;
    display: block;
}

.quick-btn-redo {
    background: url(../../../styles/css/assets/shorcuts/redo.svg) center/cover;
    width: 15.2px;
    height: 16.2px;
    display: block;
}

.quick-btn-view {
    color: white
}

/* Zoom controls with slider (1-100% display) */
.zoom-controls {
    display: flex;
    gap: 8px;
    align-items: center;
}

.zoom-button-decrement,
.zoom-button-increment {
    background: transparent;
    color: white;
    border: none;
    cursor: pointer;
    font-size: 20px;
    font-weight: bold;
    padding: 0 6px;
    line-height: 1;
    min-width: 24px;
}

.zoom-button-decrement:hover,
.zoom-button-increment:hover {
    opacity: 0.7;
}

.zoom-slider {
    width: 100px;
    height: 5px;
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
    width: 14px;
    height: 14px;
    background: white;
    border-radius: 50%;
    cursor: pointer;
    transition: background 0.15s ease-in-out;
}

.zoom-slider::-webkit-slider-thumb:hover {
    background: #ddd;
}

.zoom-slider::-moz-range-thumb {
    width: 14px;
    height: 14px;
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
    font-size: 12px;
    font-weight: 500;
    min-width: 40px;
    text-align: right;
}

.zoom-button-decrement:hover,
.zoom-button-increment:hover {
    opacity: 0.7;
}

@media (max-width: 991px) {
    .quick-btn-save-online {
        background: url(../../../styles/css/assets/shorcuts/save-online.svg) center/cover;
        width: 25.45px;
        height: 17.85px;
        display: block;
    }

    .quick-btn-save {
        background: url(../../../styles/css/assets/shorcuts/save.svg) center/cover;
        width: 19px;
        height: 19px;
        display: block;
    }

    .quick-btn-delete {
        background: url(../../../styles/css/assets/shorcuts/delete.svg) center/cover;
        width: 25px;
        height: 19px;
        display: block;
    }

    .quick-btn-download {
        background: url(../../../styles/css/assets/shorcuts/download.svg) center/cover;
        width: 19px;
        height: 19px;
        display: block;
    }

    .quick-btn-zoom-fit {
        background: url(../../../styles/css/assets/shorcuts/fit.svg) center/cover;
        width: 19px;
        height: 19px;
        display: block;
    }

    .quick-btn-undo {
        background: url(../../../styles/css/assets/shorcuts/undo.svg) center/cover;
        width: 19px;
        height: 20.25px;
        display: block;
    }

    .quick-btn-redo {
        background: url(../../../styles/css/assets/shorcuts/redo.svg) center/cover;
        width: 19px;
        height: 20.25px;
        display: block;
    }

    .quick-btn-timing {
        font-size: 1.2rem;
    }

    .quick-btn-view {
        font-size: 1.25rem;
    }
}
</style>
