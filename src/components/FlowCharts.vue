<template>
  <input
    type="color"
    style="display: none"
    ref="colorInput"
    @change="onColorChange"
  />
  <div
    style="
      display: flex;
      justify-content: center;
      position: relative;
    "
  >
    <!-- Barra laterale -->
    <div
      style="
        width: 60px;
        background: #f0f0f0;
        border-right: 1px solid #ccc;
        display: flex;
        flex-direction: column;
        align-items: center;
      "
    >
      <div
        @click="setMode('color')"
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{ background: mode == 'color' ? 'gray' : '#f0f0f0' }"
      >
        <fa-i icon="fa-solid fa-palette" style="font-size: 24px" />
      </div>
      <div
        @click="
          setMode('add');
          addShapeType = 'rect';
        "
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{
          background:
            mode == 'add' && addShapeType == 'rect' ? 'gray' : '#f0f0f0',
        }"
      >
        <fa-i icon="fa-regular fa-square" style="font-size: 24px" />
      </div>
      <div
        @click="
          setMode('add');
          addShapeType = 'diamond';
        "
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{
          background:
            mode == 'add' && addShapeType == 'diamond' ? 'gray' : '#f0f0f0',
        }"
      >
        <fa-i
          icon="fa-regular fa-square"
          style="font-size: 24px; transform: rotate(45deg)"
        />
      </div>
      <div
        @click="
          setMode('add');
          addShapeType = 'circle';
        "
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{
          background:
            mode == 'add' && addShapeType == 'circle' ? 'gray' : '#f0f0f0',
        }"
      >
        <fa-i icon="fa-regular fa-circle" style="font-size: 24px" />
      </div>
      <div
        @click="setMode('link')"
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{ background: mode == 'link' ? 'gray' : '#f0f0f0' }"
      >
        <fa-i icon="fa-solid fa-pencil" style="font-size: 24px" />
      </div>
      <div
        @click="setMode('move')"
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{ background: mode == 'move' ? 'gray' : '#f0f0f0' }"
      >
        <fa-i icon="fa-solid fa-up-down-left-right" style="font-size: 24px" />
      </div>
      <div
        @click="setMode('resize')"
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{ background: mode == 'resize' ? 'gray' : '#f0f0f0' }"
      >
        <fa-i icon="fa-solid fa-expand" style="font-size: 24px" />
      </div>
      <div
        @click="setMode('delete')"
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{ background: mode == 'delete' ? 'gray' : '#f0f0f0' }"
      >
        <fa-i icon="fa-regular fa-trash-can" style="font-size: 24px" />
      </div>
      <div
        @click="setMode('text')"
        style="cursor: pointer; padding: 10px; width: 100%"
        :style="{ background: mode == 'text' ? 'gray' : '#f0f0f0' }"
      >
        <fa-i icon="fa-solid fa-font" style="font-size: 24px" />
      </div>
      <!-- Download -->
      <div
        @click="download"
        style="cursor: pointer; padding: 10px; width: 100%; background: #f0f0f0"
      >
        <fa-i icon="fa-solid fa-download" style="font-size: 24px" />
      </div>

      <!-- Print -->
      <div
        @click="printSVG"
        style="cursor: pointer; padding: 10px; width: 100%; background: #f0f0f0"
      >
        <fa-i icon="fa-solid fa-print" style="font-size: 24px" />
      </div>
      <div
        @click="rotateGrid()"
        style="cursor: pointer; padding: 10px; width: 100%; background: #f0f0f0"
      >
        <fa-i icon="fa-solid fa-arrows-rotate" style="font-size: 24px" />
      </div>
    </div>

    <!-- Griglia -->
    <svg
      ref="canvas"
      @contextmenu.prevent="onContextMenu"
      @mousedown="onMouseDown"
      @mousemove.passive="onMouseMove"
      @mouseup="onMouseUp"
      :width="canvasWidth"
      :height="canvasHeight"
      :style="{ border: '1px solid #ccc', cursor: currentCursor }"
    >
      <defs>
        <template v-for="(link, i) in links" :key="'marker-' + i">
          <marker
            :id="'arrow-' + i"
            viewBox="0 0 10 10"
            refX="5"
            refY="5"
            markerWidth="6"
            markerHeight="6"
            orient="auto"
          >
            <path d="M 0 0 L 10 5 L 0 10 z" :fill="link.color || 'black'" />
          </marker>
        </template>
      </defs>

      <template v-for="(row, y) in grid" :key="'bg-' + y">
        <template v-for="(_, x) in row" :key="`bg-${x},${y}`">
          <rect
            :x="x * cellSize"
            :y="y * cellSize"
            :width="cellSize"
            :height="cellSize"
            fill="#f9f9f9"
            stroke="#ddd"
          />
        </template>
      </template>

      <!-- Blocchi -->
      <template v-for="block in blocks" :key="'block-' + block.id">
        <rect
          v-if="block.type === 'rect' || !block.type"
          :x="block.x * cellSize"
          :y="block.y * cellSize"
          :width="block.w * cellSize"
          :height="block.h * cellSize"
          :fill="block.color"
          stroke="#555"
          stroke-width="2"
        />

        <polygon
          v-else-if="block.type === 'diamond'"
          :points="
            [
              [block.x + block.w / 2, block.y],
              [block.x + block.w, block.y + block.h / 2],
              [block.x + block.w / 2, block.y + block.h],
              [block.x, block.y + block.h / 2],
            ]
              .map(([x, y]) => [x * cellSize, y * cellSize].join(','))
              .join(' ')
          "
          :fill="block.color"
          stroke="#555"
          stroke-width="2"
        />

        <ellipse
          v-else-if="block.type === 'circle'"
          :cx="(block.x + block.w / 2) * cellSize"
          :cy="(block.y + block.h / 2) * cellSize"
          :rx="(block.w * cellSize) / 2"
          :ry="(block.h * cellSize) / 2"
          :fill="block.color"
          stroke="#555"
          stroke-width="2"
        />
        <template v-if="mode === 'resize'">
          <circle
            v-for="(pos, index) in [
              [block.x, block.y],
              [block.x + block.w, block.y],
              [block.x, block.y + block.h],
              [block.x + block.w, block.y + block.h],
            ]"
            :key="index"
            :cx="pos[0] * cellSize"
            :cy="pos[1] * cellSize"
            r="4"
            fill="black"
          />
        </template>
      </template>

      <!-- Preview nuovo blocco -->
      <template v-if="mode === 'add' && previewBlock">
        <rect
          v-if="previewBlock.type === 'rect'"
          :x="previewBlock.x * cellSize"
          :y="previewBlock.y * cellSize"
          :width="previewBlock.w * cellSize"
          :height="previewBlock.h * cellSize"
          fill="lightblue"
          stroke="#999"
          stroke-dasharray="4"
        />
        <polygon
          v-else-if="previewBlock.type === 'diamond'"
          :points="
            [
              [previewBlock.x + previewBlock.w / 2, previewBlock.y],
              [
                previewBlock.x + previewBlock.w,
                previewBlock.y + previewBlock.h / 2,
              ],
              [
                previewBlock.x + previewBlock.w / 2,
                previewBlock.y + previewBlock.h,
              ],
              [previewBlock.x, previewBlock.y + previewBlock.h / 2],
            ]
              .map(([x, y]) => [x * cellSize, y * cellSize].join(','))
              .join(' ')
          "
          fill="lightblue"
          stroke="#999"
          stroke-dasharray="4"
        />

        <ellipse
          v-else-if="previewBlock.type === 'circle'"
          :cx="(previewBlock.x + previewBlock.w / 2) * cellSize"
          :cy="(previewBlock.y + previewBlock.h / 2) * cellSize"
          :rx="(previewBlock.w * cellSize) / 2"
          :ry="(previewBlock.h * cellSize) / 2"
          fill="lightblue"
          stroke="#999"
          stroke-dasharray="4"
        />
      </template>

      <!-- testo blocchi -->
      <template v-for="block in blocks" :key="'text-' + block.id">
        <text
          v-if="block.content && (!editingText || editingText.id !== block.id)"
          :clip-path="'url(#clip-' + block.id + ')'"
          style="cursor: pointer"
          :x="getTextX(block)"
          :y="
            block.y * cellSize +
            (block.h * cellSize) / 2 -
            (getTruncatedLines(block).length - 1) * 8
          "
          font-family="monospace"
          font-size="14"
          fill="black"
          :text-anchor="block.align"
        >
          <tspan
            v-for="(line, index) in getTruncatedLines(block)"
            :key="index"
            :x="getTextX(block)"
            :dy="index === 0 ? 0 : 16"
          >
            {{ line }}
          </tspan>
        </text>
      </template>

      <!-- Collegamenti -->
      <template v-for="(link, i) in links" :key="'link-' + i">
        <template
          v-for="(segment, j) in getLinkSegments(link)"
          :key="'seg-' + j"
        >
          <line
            :x1="segment.x1 * cellSize"
            :y1="segment.y1 * cellSize"
            :x2="segment.x2 * cellSize"
            :y2="segment.y2 * cellSize"
            :stroke="link.color"
            stroke-width="3"
            :marker-end="
              j === getLinkSegments(link).length - 1
                ? 'url(#arrow-' + i + ')'
                : ''
            "
          />
        </template>
      </template>

      <!-- Preview collegamento -->
      <template v-if="linkStart && tempLineEnd">
        <template v-for="(segment, i) in previewSegments" :key="'preview-' + i">
          <line
            :x1="segment.x1 * cellSize"
            :y1="segment.y1 * cellSize"
            :x2="segment.x2 * cellSize"
            :y2="segment.y2 * cellSize"
            stroke="black"
            stroke-width="3"
            stroke-dasharray="4"
            :marker-end="i === previewSegments.length - 1 ? 'url(#arrow)' : ''"
          />
        </template>
      </template>
    </svg>
  </div>
  <Dialog v-model="dialog" @update:modelValue="confirmTextEdit">
    <!-- Editor di testo HTML sopra SVG -->
    <div
      v-if="editingText"
      ref="textEditor"
      contenteditable
      @mousedown.stop
      @input="editingInput = $event.target.innerText"
      @keydown.esc="cancelTextEdit"
      :style="editingTextStyle"
    />
  </Dialog>
</template>

<script>
import Dialog from "@/components/VDialog.vue";
export default {
  components: { Dialog },
  data() {
    const rows = 45;
    const cols = 80;
    const cellSize = 15;
    return {
      selectedElement: null,
      dialog: false,
      addShapeType: "rect",
      charWidth: 8,
      editingText: null,
      editingInput: "",
      previewSegments: [],
      resizeCorner: null,
      hoveredResizeCorner: null,
      reverseState: 0,
      movingBlock: null,
      dragOffset: { x: 0, y: 0 },
      dragging: false,
      resizing: false,
      linking: false,
      rightClickInProgress: false,
      currentCursor: "default",
      rows,
      cols,
      cellSize,
      canvasWidth: cols * cellSize,
      canvasHeight: rows * cellSize,
      grid: Array.from({ length: rows }, () => Array(cols).fill(0)),
      blocks: [
        {
          id: 1,
          x: 5,
          y: 5,
          w: 6,
          h: 4,
          type: "rect",
          align: "start",
          color: "#ADD8E6",
        },
        {
          id: 2,
          x: 20,
          y: 10,
          w: 6,
          h: 6,
          type: "diamond",
          align: "middle",
          color: "#ADD8E6",
        },
        {
          id: 3,
          x: 30,
          y: 20,
          w: 6,
          h: 6,
          type: "circle",
          align: "middle",
          color: "#ADD8E6",
        },
      ],
      links: [],
      movingLink: null,
      linkDragOffset: { x: 0, y: 0 },
      mode: "link",
      linkStart: null,
      tempLineEnd: null,
      previewBlock: null,
    };
  },

  computed: {
    editingTextStyle() {
      if (!this.editingText) return {};

      const block = this.editingText;
      return {
        width: block.w * this.cellSize + "px",
        height: block.h * this.cellSize + "px",
        fontFamily: "monospace",
        fontSize: "14px",
        color: "black",
        background: "white",
        outline: "none",
        border: "none",
        padding: "4px",
        display: "block",
        overflow: "hidden",
        whiteSpace: "pre-wrap",
        pointerEvents: "auto",
        textAlign:
          block.align === "start"
            ? "left"
            : block.align === "end"
            ? "right"
            : "center",
      };
    },
  },
  methods: {
    setMode(mode) {
      this.mode = mode;
    },
    getLinkSegments(link) {
      const { from, to, reverse } = link;
      const segments = [];

      const dx = Math.abs(to.x - from.x);
      const dy = Math.abs(to.y - from.y);
      const threshold = 20 / this.cellSize;

      const isHorizontal = from.y === to.y;
      const isVertical = from.x === to.x;

      if (reverse === 1) {
        segments.push({ x1: from.x, y1: from.y, x2: from.x, y2: to.y });
        segments.push({ x1: from.x, y1: to.y, x2: to.x, y2: to.y });
      } else if (reverse === 2) {
        segments.push({ x1: from.x, y1: from.y, x2: to.x, y2: from.y });
        segments.push({ x1: to.x, y1: from.y, x2: to.x, y2: to.y });
      } else {
        if ((dx > dy && dy < threshold) || isHorizontal) {
          segments.push({ x1: from.x, y1: from.y, x2: to.x, y2: from.y });
          if (from.y !== to.y) {
            segments.push({ x1: to.x, y1: from.y, x2: to.x, y2: to.y });
          }
        } else if ((dy > dx && dx < threshold) || isVertical) {
          segments.push({ x1: from.x, y1: from.y, x2: from.x, y2: to.y });
          if (from.x !== to.x) {
            segments.push({ x1: from.x, y1: to.y, x2: to.x, y2: to.y });
          }
        } else {
          segments.push({ x1: from.x, y1: from.y, x2: to.x, y2: to.y });
        }
      }

      return segments;
    },

    getTruncatedLines(block) {
      const heightLimit = block.h * this.cellSize;
      const lineHeight = 16;
      const maxLines = Math.floor((heightLimit - 4) / lineHeight);
      const wrapped = this.getWrappedLines(block);
      if (wrapped.length <= maxLines) return wrapped;

      const truncated = wrapped.slice(0, maxLines);
      // Sostituisci l'ultima riga con "..." quando il contenuto non sta dentro il blocco
      truncated[maxLines - 1] =
        truncated[maxLines - 1].replace(/.{3}$/, "") + "...";
      return truncated;
    },

    download() {
      const svgEl = this.$refs.canvas;

      // Aggiungi margine di 1 cella su ogni lato
      const originalWidth = this.canvasWidth;
      const originalHeight = this.canvasHeight;

      const canvas = document.createElement("canvas");
      const margin = this.cellSize;
      canvas.width = originalWidth + margin * 2;
      canvas.height = originalHeight + margin * 2;

      const ctx = canvas.getContext("2d");
      const img = new Image();
      const svgData = new XMLSerializer().serializeToString(svgEl);
      const svgBlob = new Blob([svgData], {
        type: "image/svg+xml;charset=utf-8",
      });
      const url = URL.createObjectURL(svgBlob);

      img.onload = () => {
        ctx.drawImage(img, margin, margin);
        URL.revokeObjectURL(url);

        const dataUrl = canvas.toDataURL("image/png");

        const a = document.createElement("a");
        a.href = dataUrl;
        a.download = "flowchart.png";
        document.body.appendChild(a);
        a.click();
        document.body.removeChild(a);
      };

      img.src = url;
    },
    printSVG() {
      const svg = this.$refs.canvas;
      const serializer = new XMLSerializer();
      const source = serializer.serializeToString(svg);

      // Calcola orientamento
      const isLandscape = this.canvasWidth > this.canvasHeight;
      const pageOrientation = isLandscape ? "landscape" : "portrait";

      const html = `
    <html>
      <head>
        <title>Stampa Flowchart</title>
        <style>
          @page {
            size: A4 ${pageOrientation};
            margin: 0;
          }
          html, body {
            margin: 0;
            padding: 0;
            height: 100%;
            width: 100%;
          }
          body {
            display: flex;
            align-items: center;
            justify-content: center;
            background: white;
          }
          .svg-wrapper {
            display: flex;
            align-items: center;
            justify-content: center;
            max-width: 100%;
            max-height: 100%;
          }
          svg {
            display: block;
            max-width: 100%;
            max-height: 100%;
            height: auto;
            width: auto;
            background: white;
          }
        </style>
      </head>
      <body onload="window.print(); window.close();">
        <div class="svg-wrapper">
          ${source}
        </div>
      </body>
    </html>
  `;

      const printWindow = window.open("", "_blank");
      if (!printWindow) return;
      printWindow.document.open();
      printWindow.document.write(html);
      printWindow.document.close();
    },
    rotateGrid() {
      // Scambia righe e colonne
      const newRows = this.cols;
      const newCols = this.rows;

      // Aggiorna griglia
      this.grid = Array.from({ length: newRows }, () => Array(newCols).fill(0));

      // Ruota i blocchi: (x, y) diventa (y, x), (w, h) diventa (h, w)
      this.blocks = this.blocks.map((b) => ({
        ...b,
        x: b.y,
        y: b.x,
        w: b.h,
        h: b.w,
      }));

      // Ruota i collegamenti
      this.links = this.links.map((l) => ({
        ...l,
        from: { x: l.from.y, y: l.from.x },
        to: { x: l.to.y, y: l.to.x },
      }));

      // Ruota il blocco di anteprima se presente
      if (this.previewBlock) {
        this.previewBlock = {
          x: this.previewBlock.y,
          y: this.previewBlock.x,
          w: this.previewBlock.h,
          h: this.previewBlock.w,
        };
      }

      // Scambia rows e cols
      [this.rows, this.cols] = [newRows, newCols];

      // Aggiorna dimensioni canvas
      this.canvasWidth = newCols * this.cellSize;
      this.canvasHeight = newRows * this.cellSize;
    },

    confirmTextEdit() {
      if (this.dialog) {
        return;
      }
      if (this.editingText) {
        this.editingText.content = this.editingInput;
        this.editingText = null;
        this.editingInput = "";
      }
    },

    onColorChange(event) {
      const newColor = event.target.value;

      if (!this.selectedElement) return;

      if ("from" in this.selectedElement && "to" in this.selectedElement) {
        // È una freccia (link)
        const index = this.links.findIndex((l) => l === this.selectedElement);
        if (index !== -1) {
          this.links[index].color = newColor;
        }
      } else {
        // È un blocco
        const index = this.blocks.findIndex((b) => b === this.selectedElement);
        if (index !== -1) {
          this.blocks[index].color = newColor;
        }
      }

      this.selectedElement = null;

      // Rimuovi l'input dinamico (importante se lo lasci visibile)
      const existing = document.getElementById("dynamic-color-input");
      if (existing) existing.remove();
    },
    cancelTextEdit() {
      if (this.editingText) {
        this.editingText = null;
        this.editingInput = "";
      }
      this.dialog = false;
    },
    distanceToSegment(px, py, x1, y1, x2, y2) {
      const dx = x2 - x1;
      const dy = y2 - y1;
      if (dx === 0 && dy === 0) {
        // Il segmento è un punto
        return Math.hypot(px - x1, py - y1);
      }

      const t = ((px - x1) * dx + (py - y1) * dy) / (dx * dx + dy * dy);
      const clampedT = Math.max(0, Math.min(1, t));
      const closestX = x1 + clampedT * dx;
      const closestY = y1 + clampedT * dy;

      return Math.hypot(px - closestX, py - closestY);
    },
    onMouseDown(e) {
      const x = e.offsetX / this.cellSize;
      const y = e.offsetY / this.cellSize;
      const gridX = Math.floor(x);
      const gridY = Math.floor(y);
      e.preventDefault();

      // click destro per scegliere come disegnare i segmenti(obliquo o solo orizzontale/verticale)
      if (e.button === 2 && this.linking) {
        this.reverseState = (this.reverseState + 1) % 3;
        this.rightClickInProgress = true;
        return;
      }

      let block = this.blocks.find(
        (b) =>
          gridX >= b.x && gridX < b.x + b.w && gridY >= b.y && gridY < b.y + b.h
      );
      const clickedLink = this.links.find((link) =>
        this.getLinkSegments(link).some((seg) => {
          const x1 = seg.x1 * this.cellSize;
          const y1 = seg.y1 * this.cellSize;
          const x2 = seg.x2 * this.cellSize;
          const y2 = seg.y2 * this.cellSize;
          const dist = this.distanceToSegment(
            e.offsetX,
            e.offsetY,
            x1,
            y1,
            x2,
            y2
          );
          const withinMarker = Math.hypot(e.offsetX - x2, e.offsetY - y2) < 4;
          return dist < 6 || withinMarker;
        })
      );

      if (this.mode === "text") {
        console.log(e.button);

        if (block && e.button === 2 && block.type == "rect") {
          // Tasto destro in modalità testo: cambia allineamento
          if (!block.align) block.align = "start";
          block.align =
            block.align === "start"
              ? "middle"
              : block.align === "middle"
              ? "end"
              : "start";
        } else if (block && e.button == 0) {
          this.editingText = block;
          this.editingInput = block.content || "";

          this.dialog = true;

          this.$nextTick(() => {
            const el = this.$refs.textEditor;
            if (el) {
              el.innerText = this.editingInput;
              el.focus();
              const range = document.createRange();
              range.selectNodeContents(el);
              range.collapse(false); // cursore alla fine
              const sel = window.getSelection();
              sel.removeAllRanges();
              sel.addRange(range);
            }
          });
        }

        return;
      }

      if (this.mode === "move" && e.button == 0) {
        if (block) {
          this.movingBlock = block;
          this.dragging = true;
          this.currentCursor = "grabbing";
          this.dragOffset = { x: gridX - block.x, y: gridY - block.y };
          return;
        }
        if (clickedLink) {
          this.movingLink = clickedLink;
          this.dragging = true;
          this.currentCursor = "grabbing";
          this.linkDragOffset = { x, y };
          return;
        }
      }

      if (this.mode === "delete" && e.button == 0) {
        if (block) {
          this.blocks = this.blocks.filter((b) => b !== block);
          return;
        }
        if (clickedLink) {
          this.links = this.links.filter((l) => l !== clickedLink);
          return;
        }
      }

      if (this.mode === "color" && e.button === 0) {
        if (block || clickedLink) {
          this.selectedElement = block || clickedLink;

          // Rimuovi eventuale input precedente
          const existing = document.getElementById("dynamic-color-input");
          if (existing) existing.remove();

          // Crea nuovo input color
          const input = document.createElement("input");
          input.type = "color";
          input.id = "dynamic-color-input";
          input.style.position = "absolute";
          input.style.left = e.pageX + "px";
          input.style.top = e.pageY + "px";
          input.style.zIndex = 1000;
          input.style.opacity = 0; // invisibile
          input.style.pointerEvents = "none"; // non interferisce con eventi
          input.value = this.selectedElement.color || "#000000";

          // Cambia colore solo quando l'utente conferma (rilascia il click)
          input.addEventListener("change", (event) => {
            const newColor = event.target.value;

            if (
              "from" in this.selectedElement &&
              "to" in this.selectedElement
            ) {
              const index = this.links.findIndex(
                (l) => l === this.selectedElement
              );
              if (index !== -1) this.links[index].color = newColor;
            } else {
              const index = this.blocks.findIndex(
                (b) => b === this.selectedElement
              );
              if (index !== -1) this.blocks[index].color = newColor;
            }

            this.selectedElement = null;
          });

          // Rimuovi l’input solo quando perde il focus, NON subito al change
          input.addEventListener("blur", () => {
            input.remove();
          });

          // Aggiungi al DOM e simula click
          document.body.appendChild(input);
          requestAnimationFrame(() => {
            input.click();
          });

          return;
        }
      }

      if (this.mode === "add" && e.button == 0) {
        const id = this.blocks.length + 1;
        const newType = this.addShapeType || "rect";

        let w = 6;
        let h = 4;
        if (newType !== "rect") {
          w = 6;
          h = w;
        }
        this.blocks.push({
          id,
          x: gridX,
          y: gridY,
          w,
          h,
          type: newType,
          color: "#ADD8E6",
        });

        this.previewBlock = null;
        return;
      }

      if (this.mode === "resize" && e.button == 0) {
        const radius = 0.8;
        for (const b of this.blocks) {
          const corners = [
            { x: b.x, y: b.y, corner: "tl" },
            { x: b.x + b.w, y: b.y, corner: "tr" },
            { x: b.x, y: b.y + b.h, corner: "bl" },
            { x: b.x + b.w, y: b.y + b.h, corner: "br" },
          ];
          for (const c of corners) {
            const dx = x - c.x;
            const dy = y - c.y;
            if (dx * dx + dy * dy < radius * radius) {
              this.movingBlock = b;
              this.resizing = true;
              this.resizeCorner = c.corner;
              this.hoveredResizeCorner = c.corner;
              return;
            }
          }
        }
      }

      if (this.mode === "link" && e.button == 0) {
        this.linkStart = { x, y };
        this.tempLineEnd = { x, y };
        this.linking = true;
      }
    },

    getTextX(block) {
      const padding = 4;
      const left = block.x * this.cellSize + padding;
      const center = block.x * this.cellSize + (block.w * this.cellSize) / 2;
      const right = block.x * this.cellSize + block.w * this.cellSize - padding;

      if (block.align === "start") return left;
      if (block.align === "end") return right;
      return center;
    },

    onMouseMove(e) {
      const x = e.offsetX / this.cellSize;
      const y = e.offsetY / this.cellSize;
      const gridX = Math.floor(x);
      const gridY = Math.floor(y);
      this.currentCursor = "default";

      const hoveringBlock = this.blocks.find(
        (b) =>
          gridX >= b.x && gridX < b.x + b.w && gridY >= b.y && gridY < b.y + b.h
      );
      const overLink = this.links.find((link) =>
        this.getLinkSegments(link).some((seg) => {
          const x1 = seg.x1 * this.cellSize;
          const y1 = seg.y1 * this.cellSize;
          const x2 = seg.x2 * this.cellSize;
          const y2 = seg.y2 * this.cellSize;
          const dist = this.distanceToSegment(
            e.offsetX,
            e.offsetY,
            x1,
            y1,
            x2,
            y2
          );
          const withinMarker = Math.hypot(e.offsetX - x2, e.offsetY - y2) < 4;
          return dist < 6 || withinMarker;
        })
      );

      if (this.mode === "color" && (hoveringBlock || overLink)) {
        this.currentCursor = "pointer";
        return;
      }
      if (this.mode === "delete" && (hoveringBlock || overLink)) {
        this.currentCursor = "pointer";
        return;
      }
      if (this.mode === "text" && hoveringBlock) {
        this.currentCursor = "pointer";
        return;
      }
      if (this.mode === "move") {
        if (this.dragging && this.movingBlock) {
          this.currentCursor = "grabbing";
          const dx = gridX - this.dragOffset.x;
          const dy = gridY - this.dragOffset.y;
          const block = this.movingBlock;
          block.x = Math.max(0, Math.min(dx, this.cols - block.w));
          block.y = Math.max(0, Math.min(dy, this.rows - block.h));
          return;
        }
        if (this.dragging && this.movingLink) {
          this.currentCursor = "grabbing";
          const dx = x - this.linkDragOffset.x;
          const dy = y - this.linkDragOffset.y;
          this.movingLink.from.x += dx;
          this.movingLink.from.y += dy;
          this.movingLink.to.x += dx;
          this.movingLink.to.y += dy;
          this.linkDragOffset = { x, y };
          return;
        }
        if (overLink) {
          this.currentCursor = "grab";
          return;
        }
        if (hoveringBlock) {
          this.currentCursor = "grab";
        }
      }

      if (this.mode === "resize") {
        const radius = 0.8;
        for (const b of this.blocks) {
          const corners = [
            { x: b.x, y: b.y, corner: "tl" },
            { x: b.x + b.w, y: b.y, corner: "tr" },
            { x: b.x, y: b.y + b.h, corner: "bl" },
            { x: b.x + b.w, y: b.y + b.h, corner: "br" },
          ];
          for (const c of corners) {
            const dx = x - c.x;
            const dy = y - c.y;
            if (dx * dx + dy * dy < radius * radius) {
              this.hoveredResizeCorner = c.corner;
              this.currentCursor = "nwse-resize";
              break;
            }
          }
        }
      }

      if (this.resizing && this.movingBlock && this.hoveredResizeCorner) {
        const b = this.movingBlock;
        const mx = gridX;
        const my = gridY;

        let newX = b.x;
        let newY = b.y;
        let newW = b.w;
        let newH = b.h;

        switch (this.hoveredResizeCorner) {
          case "tl":
            newW = b.w + (b.x - mx);
            newH = b.h + (b.y - my);
            if (newW >= 1) newX = mx;
            if (newH >= 1) newY = my;
            break;
          case "tr":
            newW = mx - b.x;
            newH = b.h + (b.y - my);
            if (newH >= 1) newY = my;
            break;
          case "bl":
            newW = b.w + (b.x - mx);
            newH = my - b.y;
            if (newW >= 1) newX = mx;
            break;
          case "br":
            newW = mx - b.x;
            newH = my - b.y;
            break;
        }

        // Imposta dimensione minima
        const minSize = 1;

        // Se la forma è circle o diamond, mantieni proporzioni quadrate
        const isFixedRatio = ["circle", "diamond"].includes(b.type);
        if (isFixedRatio) {
          // Calcola il lato comune
          let size = Math.max(minSize, Math.max(newW, newH));
          switch (this.hoveredResizeCorner) {
            case "tl":
              if (b.x + b.w - size >= 0 && b.y + b.h - size >= 0) {
                b.x = b.x + b.w - size;
                b.y = b.y + b.h - size;
                b.w = b.h = size;
              }
              break;
            case "tr":
              if (b.x + size <= this.cols && b.y + b.h - size >= 0) {
                b.y = b.y + b.h - size;
                b.w = b.h = size;
              }
              break;
            case "bl":
              if (b.x + b.w - size >= 0 && b.y + size <= this.rows) {
                b.x = b.x + b.w - size;
                b.w = b.h = size;
              }
              break;
            case "br":
              if (b.x + size <= this.cols && b.y + size <= this.rows) {
                b.w = b.h = size;
              }
              break;
          }
        } else {
          if (newW >= minSize) {
            b.x = newX;
            b.w = newW;
          }
          if (newH >= minSize) {
            b.y = newY;
            b.h = newH;
          }
        }
      }

      if (this.linking) {
        const dx = x - this.linkStart.x;
        const dy = y - this.linkStart.y;
        const threshold = 20 / this.cellSize;
        if (Math.abs(dx) > Math.abs(dy) && Math.abs(dy) < threshold) {
          this.tempLineEnd = { x, y: this.linkStart.y };
        } else if (Math.abs(dy) > Math.abs(dx) && Math.abs(dx) < threshold) {
          this.tempLineEnd = { x: this.linkStart.x, y };
        } else {
          this.tempLineEnd = { x, y };
        }
        this.previewSegments = this.getLinkSegments({
          from: this.linkStart,
          to: this.tempLineEnd,
          reverse: this.reverseState,
        });
      }

      if (this.mode === "add") {
        let w = 6;
        let h = 4;
        if (this.addShapeType !== "rect") {
          w = 6;
          h = w;
        }
        this.previewBlock = {
          x: gridX,
          y: gridY,
          w,
          h,
          type: this.addShapeType,
        };
      }
    },
    onMouseUp(e) {
      const x = e.offsetX / this.cellSize;
      const y = e.offsetY / this.cellSize;

      if (e.button === 2) {
        this.rightClickInProgress = false;
        return;
      }

      if (this.resizing || this.dragging) {
        this.resizing = false;
        this.dragging = false;
        this.resizeCorner = null;
        this.hoveredResizeCorner = null;
        this.movingBlock = null;
        this.movingLink = null;
      }

      if (this.linking) {
        const dx = Math.abs(x - this.linkStart.x);
        const dy = Math.abs(y - this.linkStart.y);
        const threshold = 20 / this.cellSize;

        let snappedTo = { x, y };
        if (dx > dy && dy < threshold) {
          snappedTo.y = this.linkStart.y;
        } else if (dy > dx && dx < threshold) {
          snappedTo.x = this.linkStart.x;
        }
        // SALTA se troppo corta
        const minLength = 1;
        const length = Math.sqrt(
          (snappedTo.x - this.linkStart.x) ** 2 +
            (snappedTo.y - this.linkStart.y) ** 2
        );
        if (length < minLength) {
          this.linking = false;
          this.linkStart = null;
          this.tempLineEnd = null;
          this.reverseState = 0;
          return; // non creare il link
        }
        this.links.push({
          from: this.linkStart,
          to: snappedTo,
          reverse: this.reverseState,
          color: "#000000",
        });
        this.linking = false;
        this.linkStart = null;
        this.tempLineEnd = null;
        this.reverseState = 0;
      }
    },
    onKeyDown(e) {
      if (e.key === "Escape") {
        this.linking = false;
        this.linkStart = null;
        this.tempLineEnd = null;
        this.previewBlock = null;
        this.reverseState = 0;
      }
    },
    getWrappedLines(block) {
      const width = block.w * this.cellSize;
      const maxChars = Math.floor((width - 8) / this.charWidth); // -8 per padding
      const lines = [];

      for (let paragraph of (block.content || "").split("\n")) {
        const words = paragraph.split(" ");
        let line = "";
        for (let word of words) {
          while (word.length > maxChars) {
            if (line) {
              lines.push(line);
              line = "";
            }
            lines.push(word.slice(0, maxChars));
            word = word.slice(maxChars);
          }

          if ((line + (line ? " " : "") + word).length <= maxChars) {
            line += (line ? " " : "") + word;
          } else {
            if (line) lines.push(line);
            line = word;
          }
        }

        if (line) lines.push(line);
      }

      return lines;
    },
  },
  mounted() {
    window.addEventListener("keydown", this.onKeyDown);
  },
  beforeUnmount() {
    window.removeEventListener("keydown", this.onKeyDown);
  },
};
</script>
<style scoped>
div[contenteditable] {
  line-height: 16px;
  margin: 0;
}
div[contenteditable] * {
  margin: 0 !important;
}
</style>