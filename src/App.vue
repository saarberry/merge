<template>
    <div class="wrapper">
        <canvas ref="canvas" />
        <div class="buttons">
            <button @click="setCompositeOperation('source-over'); render()">source-over</button>
            <button @click="setCompositeOperation('source-in'); render()">source-in</button>
            <button @click="setCompositeOperation('source-out'); render()">source-out</button>
            <button @click="setCompositeOperation('source-atop'); render()">source-atop</button>
            <button @click="setCompositeOperation('destination-over'); render()">destination-over</button>
            <button @click="setCompositeOperation('destination-in'); render()">destination-in</button>
            <button @click="setCompositeOperation('destination-out'); render()">destination-out</button>
            <button @click="setCompositeOperation('destination-atop'); render()">destination-atop</button>
            <button @click="setCompositeOperation('lighter'); render()">lighter</button>
            <button @click="setCompositeOperation('copy'); render()">copy</button>
            <button @click="setCompositeOperation('xor'); render()">xor</button>
            <button @click="setCompositeOperation('multiply'); render()">multiply</button>
            <button @click="setCompositeOperation('screen'); render()">screen</button>
            <button @click="setCompositeOperation('overlay'); render()">overlay</button>
            <button @click="setCompositeOperation('darken'); render()">darken</button>
            <button @click="setCompositeOperation('lighten'); render()">lighten</button>
            <button @click="setCompositeOperation('color-dodge'); render()">color-dodge</button>
            <button @click="setCompositeOperation('color-burn'); render()">color-burn</button>
            <button @click="setCompositeOperation('hard-light'); render()">hard-light</button>
            <button @click="setCompositeOperation('soft-light'); render()">soft-light</button>
            <button @click="setCompositeOperation('difference'); render()">difference</button>
            <button @click="setCompositeOperation('exclusion'); render()">exclusion</button>
            <button @click="setCompositeOperation('hue'); render()">hue</button>
            <button @click="setCompositeOperation('saturation'); render()">saturation</button>
            <button @click="setCompositeOperation('color'); render()">color</button>
            <button @click="setCompositeOperation('luminosity'); render()">luminosity</button>

            <button @click="more(); render()">More</button>
            <button @click="less(); render()">Less</button>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from '@vue/reactivity';
import { onMounted } from '@vue/runtime-core';

const startX = -100;
const startY = -100;
const endX = -100; // Relative to right hand side of canvas.
const endY = -100; // Relative to bottom of canvas.
const incrementalX = 120;
const incrementalY = 160;
const clusterSize = 50;
const shapeSize = 50;
const shapeSideCount = ref<number>(3);
const more = () => shapeSideCount.value++;
const less = () => shapeSideCount.value--;

const canvas = ref<HTMLCanvasElement | null>(null);
const getContext = (): CanvasRenderingContext2D | null => {
    return canvas.value?.getContext("2d") ?? null;
}
const setCompositeOperation = (operation: string) => {
    const context = getContext();
    if (context) {
        context.globalCompositeOperation = operation;
    }
}
const setAlpha = (transparency: number) => {
    const context = getContext();
    if (context) {
        context.globalAlpha = transparency;
    }
}
const clearCanvas = () => {
    const context = getContext();
    if (context) {
        context.clearRect(0, 0, canvas.value?.width || 0, canvas.value?.height || 0);
    }
}

/**
 * Generator that keeps cycling through the hex codes for red, blue, green.
 * @generator
 * @yields {string} Hex code.
 */
function* colors(): Generator<string> {
    let availableColors: string[] = [
        "#FF0000",
        "#00FF00",
        "#0000FF",
    ];

    while (availableColors.length > 0) {
        yield availableColors[0];
        availableColors.push(availableColors[0]);
        availableColors.shift();
    }
}
const colorIterator = colors();

/**
 * Draw a cluster of shapes at the given coordinates.
 * @param {number} x
 * @param {number} y
 */
const drawShapeCluster = (x: number, y: number) => {
    for (let side = 0; side <= 6; side += 1) {
        drawShape(
            x + clusterSize * Math.cos(side * 2 * Math.PI / 6),
            y + clusterSize * Math.sin(side * 2 * Math.PI / 6)
        );
    }
}

/**
 * Draw a single shape at the given coordinates.
 * @param {number} x
 * @param {number} y
 */
const drawShape = (x: number, y: number) => {
    const context = getContext();

    if (context) {
        context.beginPath();
        context.moveTo(x + shapeSize * Math.cos(0), y + shapeSize * Math.sin(0));

        for (let side = 1; side <= shapeSideCount.value; side += 1) {
            context.lineTo(
                x + shapeSize * Math.cos(side * 2 * Math.PI / shapeSideCount.value),
                y + shapeSize * Math.sin(side * 2 * Math.PI / shapeSideCount.value)
            );
        }

        context.fillStyle = colorIterator.next().value;
        context.fill();
    }
}

const render = () => {
    clearCanvas();

    let x = startX;
    let y = startY;

    while (y < window.innerHeight - endY) {
        while (x < window.innerWidth - endX) {
            drawShapeCluster(x, y);
            x += incrementalX;
        }
        y += incrementalY;
        x = startX;
    }
}

onMounted(() => {
    if (canvas.value) {
        canvas.value.width = window.innerWidth;
        canvas.value.height = window.innerHeight;

        setCompositeOperation("lighter");
        setAlpha(.75);
        render();
    }
});
</script>

<style>
body {
    margin: 0;
}
canvas {
    background-color: black;
}
.buttons {
    position: absolute;
    top: 0;
    left: 0;
    display: flex;
    flex-wrap: wrap;
}
</style>
