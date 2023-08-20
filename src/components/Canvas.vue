<script setup lang="ts">
	import { ref, onMounted } from 'vue'
	import type { Ref } from 'vue';

	const height: Ref<number> = ref(0);
	const width: Ref<number> = ref(0);
	let canvas: HTMLCanvasElement | null;
	let svg: SVGElement | null;
	let context: CanvasRenderingContext2D | null | undefined
	let maxDist: number;
	let curXandY: number[] = [0, 0];
	let drawPoints: number[][] = [];
	let radius: number = 1;

	onMounted(() => {
		canvas = document.querySelector("div canvas");
		svg = document.querySelector("div svg");
		context = canvas?.getContext("2d");
		setSize();
		
		window.addEventListener("resize", (e) => {
			setSize();
		})
		
		document.addEventListener("keydown", (e) => {
			if (e.key.toLowerCase() === "z" && e.ctrlKey){
				undoStroke();
			} 
		})
		
		maxDist = width.value * height.value / 100000;
	})

	const setSize = () => {
		height.value = Math.ceil(window.innerHeight / 100 * 80);
		width.value = window.innerWidth;
		canvas?.setAttribute("width", width.value.toString());
		canvas?.setAttribute("height", height.value.toString());
		svg?.setAttribute("width", width.value.toString());
		svg?.setAttribute("height", height.value.toString());
	}

	const beginDraw = (e: any) => {
		curXandY[0] = e.offsetX;
		curXandY[1] = e.offsetY;
		drawPoints.push(curXandY);
		canvas?.addEventListener("mousemove", grabPoints);
	}

	const grabPoints = (e: any) => {
		let distanceX: number = Math.abs(curXandY[0] - e.offsetX);
		let distanceY: number = Math.abs(curXandY[1] - e.offsetY);

		if (Math.sqrt(distanceX * distanceX + distanceY * distanceY) >= maxDist) {
			let newcurXandY: number[] = [e.offsetX, e.offsetY];
			context?.clearRect(0, 0, width.value, height.value);
			drawPoints.push(newcurXandY);
			curXandY = newcurXandY;

			
			for (let point: number = 0; point < drawPoints.length; point++) {
		 		context?.beginPath();
		 		context?.arc(drawPoints[point][0], drawPoints[point][1], radius, Math.PI * 2, 0);
		 		context?.stroke();
			 }
		}
	}

	const endDraw = (e: any) => {
		canvas?.removeEventListener("mousemove", grabPoints);

		const markers: number[][] = createMarkers(drawPoints);
		drawVector(markers);
		drawPoints = [];
	}

	const createMarkers = (pointList: number[][]) => {
		let isPositive: boolean = true;
		let slope: number;
		let markers: number[][] = [];

		markers.push([pointList[0][0], pointList[0][1], 0]);

		for (let adjPoint: number = 1; adjPoint < pointList.length; adjPoint += 2) {
			markers.push([pointList[adjPoint][0], pointList[adjPoint][1], adjPoint]);
		}

		markers.push([pointList[pointList.length-1][0], pointList[pointList.length-1][1], pointList.length - 1]);

		return markers;
	}

	const drawVector = (markerList: number[][]) => {
		const new_group = document.createElement("g");
		for (let marker: number = 1; marker < markerList.length; marker++) {
			const svgStringStart: string = `M ${markerList[marker - 1][0]} ${markerList[marker - 1][1]} `;

			const middleLoc: number[] = [(markerList[marker - 1][0] + markerList[marker][0]) / 2, (markerList[marker - 1][1] + markerList[marker][1]) / 2];

			const medianIdx: number = Math.floor((markerList[marker][2] - markerList[marker - 1][2]) / 2 + markerList[marker - 1][2]);
			const medianPoint: number[] = drawPoints[medianIdx];

			const distanceVec: number[] = [(middleLoc[0] - medianPoint[0]), (middleLoc[1] - medianPoint[1])];

			const svgCurveStr: string = svgStringStart + `Q ${distanceVec[0] * -1 + medianPoint[0]} ${distanceVec[1] * -1 + medianPoint[1]} ${markerList[marker][0]} ${markerList[marker][1]}`;

			const curve = document.createElement("path");

			curve.setAttribute("d", svgCurveStr);
			curve.setAttribute("stroke", "black");
			curve.setAttribute("fill", "transparent");
			curve.setAttribute("stroke-linejoin", "round");
			curve.setAttribute("stroke-linecap", "round");
			curve.setAttribute("stroke-width", "1");
			new_group.appendChild(curve);
		}
		
		svg?.appendChild(new_group);
		if (svg !== null) {
			svg.innerHTML += "";
		}
	}

	const undoStroke = () => {
		svg?.removeChild(svg?.childNodes[svg?.childNodes.length - 1]);
		context?.clearRect(0, 0, width.value, height.value);
	}

	const exportSVG = () => {
		let svgString: string = "";
		if (svg != null){
			svgString = String.prototype.concat("<svg>", svg.innerHTML, "</svg>");
		}

		const svgBlob: Blob = new Blob([svgString], {type: "image/svg"});
		const downloadLocation: string = URL.createObjectURL(svgBlob);
		
		const link: HTMLAnchorElement = document.createElement('a');
		link.setAttribute("href", downloadLocation);
		link.setAttribute("download", "out.svg");

		document.body.appendChild(link);
		link.click();
		document.body.removeChild(link);
	}

</script>

<template>
	<div>
		<canvas @mousedown="beginDraw" @mouseup="endDraw">
		</canvas>
		<p>{{ width }}x{{ height }}</p>
		<svg>
		</svg>
		<div class="control-buttons">
			<button id="export" @click="exportSVG">
				Export to SVG
			</button>
			<button id="undo" @click="undoStroke">
				Undo (Ctrl+Z)
			</button>
		</div>
	</div>
</template>

<style scoped>
	div canvas {
		box-sizing: border-box;
		background: transparent;
	}

	div svg {
		top: 10vh;
		left: 0;
		position: absolute;
		z-index: -1;
	}

	p {
		position: absolute;
		top: 10vh;
		left: 0;
		z-index: 2;
		color: black;
	}

	.control-buttons > *{
		position: absolute;
		border-radius: 5%;
		height: 20px;
		font-size: 10px;
		background-color: white;
	}

	#export {
		left: 0;
		bottom: 10vh;
	}

	#undo {
		left: 0;
		bottom: calc(10vh + 20px);
	}
</style>