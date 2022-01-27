<template>
    <div class="wrapper">
        <canvas ref="canvas" />
        <div class="buttons">
            <button @click="setCompositeOperation('source-over')">source-over</button>
            <button @click="setCompositeOperation('source-in')">source-in</button>
            <button @click="setCompositeOperation('source-out')">source-out</button>
            <button @click="setCompositeOperation('source-atop')">source-atop</button>
            <button @click="setCompositeOperation('destination-over')">destination-over</button>
            <button @click="setCompositeOperation('destination-in')">destination-in</button>
            <button @click="setCompositeOperation('destination-out')">destination-out</button>
            <button @click="setCompositeOperation('destination-atop')">destination-atop</button>
            <button @click="setCompositeOperation('lighter')">lighter</button>
            <button @click="setCompositeOperation('copy')">copy</button>
            <button @click="setCompositeOperation('xor')">xor</button>
            <button @click="setCompositeOperation('multiply')">multiply</button>
            <button @click="setCompositeOperation('screen')">screen</button>
            <button @click="setCompositeOperation('overlay')">overlay</button>
            <button @click="setCompositeOperation('darken')">darken</button>
            <button @click="setCompositeOperation('lighten')">lighten</button>
            <button @click="setCompositeOperation('color-dodge')">color-dodge</button>
            <button @click="setCompositeOperation('color-burn')">color-burn</button>
            <button @click="setCompositeOperation('hard-light')">hard-light</button>
            <button @click="setCompositeOperation('soft-light')">soft-light</button>
            <button @click="setCompositeOperation('difference')">difference</button>
            <button @click="setCompositeOperation('exclusion')">exclusion</button>
            <button @click="setCompositeOperation('hue')">hue</button>
            <button @click="setCompositeOperation('saturation')">saturation</button>
            <button @click="setCompositeOperation('color')">color</button>
            <button @click="setCompositeOperation('luminosity')">luminosity</button>

            <button @click="moreX()">More X</button>
            <button @click="lessX()">Less X</button>
            <button @click="moreY()">More Y</button>
            <button @click="lessY()">Less Y</button>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ref } from '@vue/reactivity';
import { onMounted } from '@vue/runtime-core';

const canvas = ref<HTMLCanvasElement | null>(null);
const getContext = (): CanvasRenderingContext2D | null => {
    return canvas.value?.getContext("2d") ?? null;
}
const setCompositeOperation = (operation: string) => {
    const context = getContext();
    if (context) {
        context.globalCompositeOperation = operation;
        renderFuckingEverything();
    }
}
const clearCanvas = () => {
    const context = getContext();
    if (context) {
        context.clearRect(0, 0, canvas.value?.width || 0, canvas.value?.height || 0);
    }
}

const incrementalX = ref<number>(120);
const incrementalY = ref<number>(160);
const moreX = () => {
    incrementalX.value += 20;
    renderFuckingEverything();
}
const lessX = () => {
    incrementalX.value -= 20;
    renderFuckingEverything();
}
const moreY = () => {
    incrementalY.value += 20;
    renderFuckingEverything();
}
const lessY = () => {
    incrementalY.value -= 20;
    renderFuckingEverything();
}

const drawHexagons = (xStart: number, yStart: number, colors: string[]) => {
    const size = 50;
    let x: number = xStart;
    let y: number = yStart;

    for (let side = 0; side <= 6; side += 1) {
        x = x + size * Math.cos(side * 2 * Math.PI / 6);
        y = y + size * Math.sin(side * 2 * Math.PI / 6);
        drawHexagon(x, y, colors[side % colors.length]);
    }
}

const drawHexagon = (x: number, y: number, color: string) => {
    const size = 50;
    const context = getContext();

    if (context) {
        context.beginPath();
        context.moveTo(x + size * Math.cos(0), y + size * Math.sin(0));

        for (let side = 1; side <= 6; side += 1) {
            context.lineTo(x + size * Math.cos(side * 2 * Math.PI / 6), y + size * Math.sin(side * 2 * Math.PI / 6));
        }

        context.fillStyle = color;
        context.fill();
    }
}

const renderFuckingEverything = () => {
    clearCanvas();
    const context = getContext();
    if (context) {
        context.globalAlpha = .75;

        let colors = [
            "#FF0000",
            "#00FF00",
            "#0000FF",
        ];

        const initialX = -100;
        const initialY = -100;
        let x = initialX;
        let y = initialY;

        while (y < window.innerHeight - initialY) {
            while (x < window.innerWidth - initialX) {
                drawHexagons(x, y, colors);
                colors.push(colors[0]);
                colors.shift();
                x += incrementalX.value;
            }
            y += incrementalY.value;
            x = initialX;
        }
    }
}

onMounted(() => {
    if (canvas.value) {
        canvas.value.width = window.innerWidth;
        canvas.value.height = window.innerHeight;

        const context = getContext();
        if (context) {
            context.globalCompositeOperation = "lighter";
        }
    }

    renderFuckingEverything();
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
