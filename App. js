(() => {

  const $ = id => document.getElementById(id);

  const canvas = $("canvas");
  const ctx = canvas.getContext("2d");

  const vectorLayer = $("vectorLayer");


  const state = {

    tool: "brush",

    color: "#20212a",

    size: 6,

    opacity: 1,


    layers: [
      {
        id: 1,
        name: "Layer 1",
        visible: true,
        opacity: 1,
        kind: "Raster",
        data: null
      }
    ],


    active: 1,

    nextId: 2,

    drawing: false,

    start: null,

    last: null,

    undo: [],

    redo: [],

    strokes: [],

    vectors: []

  };


  /* -------------------------
     SNAPSHOT / HISTORY
  ------------------------- */

  function snapshot() {

    state.undo.push(
      JSON.stringify({
        layers: state.layers,
        active: state.active,
        strokes: state.strokes,
        vectors: state.vectors
      })
    );

    if (state.undo.length > 40) {
      state.undo.shift();
    }

    state.redo = [];
  }


  function restore(s) {

    const x = JSON.parse(s);

    state.layers = x.layers;

    state.active = x.active;

    state.strokes = x.strokes;

    state.vectors = x.vectors;

    redraw();

    renderLayers();
  }


  function current() {

    return state.layers.find(
      x => x.id === state.active
    );
  }


  function setStatus(t) {

    $("status").textContent = t;
  }


  /* -------------------------
     LAYERS
  ------------------------- */

  function renderLayers() {

    $("layers").innerHTML = "";


    [...state.layers]
      .reverse()
      .forEach(layer => {

        const row = document.createElement("div");

        row.className =
          "layer-row" +
          (layer.id === state.active
            ? " selected"
            : "");


        row.innerHTML = `
          <button
            class="layer-eye"
            title="Toggle visibility"
          >
            ${layer.visible ? "◉" : "○"}
          </button>

          <span class="layer-name"></span>

          <span class="layer-kind">
            ${layer.kind}
          </span>
        `;


        row.querySelector(".layer-name")
          .textContent = layer.name;


        row.addEventListener("click", e => {

          if (e.target.closest(".layer-eye")) {

            layer.visible = !layer.visible;

            redraw();

            renderLayers();

            return;
          }


          state.active = layer.id;

          syncProps();

          renderLayers();

        });


        $("layers").appendChild(row);

      });


    syncProps();
  }


  function syncProps() {

    const l = current();

    if (!l) return;


    $("layerName").value = l.name;

    $("layerOpacity").value =
      Math.round(l.opacity * 100);

    $("toggleLayer").textContent =
      l.visible ? "Hide" : "Show";
  }


  function addLayer() {

    snapshot();


    const l = {

      id: state.nextId++,

      name: "Layer " + state.nextId,

      visible: true,

      opacity: 1,

      kind: "Raster",

      data: null

    };


    state.layers.push(l);

    state.active = l.id;


    renderLayers();

    setStatus("Layer added");
  }


  /* -------------------------
     REDRAW
  ------------------------- */

  function redraw() {

    ctx.clearRect(
      0,
      0,
      canvas.width,
      canvas.height
    );


    ctx.fillStyle = "#ffffff";

    ctx.fillRect(
      0,
      0,
      canvas.width,
      canvas.height
    );


    state.layers.forEach(layer => {

      if (!layer.visible) return;


      ctx.save();

      ctx.globalAlpha = layer.opacity;


      state.strokes
        .filter(
          s => s.layer === layer.id
        )
        .forEach(s => {

          drawStroke(s);

        });


      ctx.restore();

    });


    /* VECTOR LAYER */

    vectorLayer.innerHTML = "";


    state.vectors.forEach(v => {

      const wrap =
        document.createElement("div");

      wrap.innerHTML = v.svg;


      const svg =
        wrap.querySelector("svg");


      if (svg) {

        svg.style.position = "absolute";

        svg.style.inset = "0";

        svg.style.width = "100%";

        svg.style.height = "100%";

        vectorLayer.appendChild(svg);

      }

    });

  }


  /* -------------------------
     DRAW STROKE
  ------------------------- */

  function drawStroke(s) {

    ctx.save();


    ctx.globalAlpha = s.opacity;

    ctx.strokeStyle = s.color;

    ctx.fillStyle = s.color;

    ctx.lineWidth = s.size;

    ctx.lineCap = "round";

    ctx.lineJoin = "round";


    if (s.tool === "eraser") {

      ctx.globalCompositeOperation =
        "destination-out";

    }


    if (
      s.tool === "brush" ||
      s.tool === "eraser"
    ) {

      ctx.beginPath();


      s.points.forEach((p, i) => {

        if (i) {

          ctx.lineTo(p.x, p.y);

        } else {

          ctx.moveTo(p.x, p.y);

        }

      });


      if (s.points.length === 1) {

        ctx.arc(
          s.points[0].x,
          s.points[0].y,
          s.size / 2,
          0,
          Math.PI * 2
        );

        ctx.fill();

      } else {

        ctx.stroke();

      }

    }

    else {

      const a =
        s.points[0];

      const b =
        s.points[s.points.length - 1];


      ctx.beginPath();


      if (s.tool === "line") {

        ctx.moveTo(a.x, a.y);

        ctx.lineTo(b.x, b.y);

        ctx.stroke();

      }


      if (s.tool === "rect") {

        ctx.strokeRect(
          a.x,
          a.y,
          b.x - a.x,
          b.y - a.y
        );

      }


      if (s.tool === "ellipse") {

        ctx.ellipse(

          (a.x + b.x) / 2,

          (a.y + b.y) / 2,

          Math.abs(b.x - a.x) / 2,

          Math.abs(b.y - a.y) / 2,

          0,

          0,

          Math.PI * 2

        );

        ctx.stroke();

      }

    }


    ctx.restore();

  }


  /* -------------------------
     CANVAS POSITION
  ------------------------- */

  function pos(e) {

    const r =
      canvas.getBoundingClientRect();


    return {

      x:
        (e.clientX - r.left)
        * canvas.width
        / r.width,

      y:
        (e.clientY - r.top)
        * canvas.height
        / r.height

    };

  }


  /* -------------------------
     DRAWING
  ------------------------- */

  canvas.addEventListener(
    "pointerdown",
    e => {

      if (state.tool === "select") {
        return;
      }


      snapshot();


      state.drawing = true;


      canvas.setPointerCapture(
        e.pointerId
      );


      const p = pos(e);


      state.start = p;

      state.last = p;


      state.live = {

        layer: state.active,

        tool: state.tool,

        color: state.color,

        size: state.size,

        opacity: state.opacity,

        points: [p]

      };


      if (
        state.tool === "brush" ||
        state.tool === "eraser"
      ) {

        ctx.beginPath();

        ctx.moveTo(
          p.x,
          p.y
        );

      }

    }
  );


  canvas.addEventListener(
    "pointermove",
    e => {

      if (!state.drawing) {
        return;
      }


      const p = pos(e);


      state.live.points.push(p);


      redraw();


      drawStroke(
        state.live
      );


      state.last = p;

    }
  );


  function finish() {

    if (!state.drawing) {
      return;
    }


    state.drawing = false;


    state.strokes.push(
      state.live
    );


    state.live = null;


    redraw();


    setStatus("Stroke added");

  }


  canvas.addEventListener(
    "pointerup",
    finish
  );


  canvas.addEventListener(
    "pointercancel",
    finish
  );


  /* -------------------------
     TOOLS
  ------------------------- */

  document
    .querySelectorAll("[data-tool]")
    .forEach(button => {

      button.addEventListener(
        "click",
        () => {

          state.tool =
            button.dataset.tool;


          document
            .querySelectorAll("[data-tool]")
            .forEach(x =>
              x.classList.toggle(
                "active",
                x === button
              )
            );


          setStatus(
            "Tool: " +
            state.tool
          );

        }
      );

    });


  /* -------------------------
     COLOR / SIZE / OPACITY
  ------------------------- */

  $("color").addEventListener(
    "input",
    e => {

      state.color =
        e.target.value;

    }
  );


  $("size").addEventListener(
    "input",
    e => {

      state.size =
        +e.target.value;


      $("sizeOut").value =
        state.size + " px";

    }
  );


  $("opacity").addEventListener(
    "input",
    e => {

      state.opacity =
        +e.target.value / 100;


      $("opacityOut").value =
        e.target.value + "%";

    }
  );


  /* -------------------------
     ADD LAYER
  ------------------------- */

  $("addLayer").onclick =
    addLayer;


  /* -------------------------
     LAYER NAME
  ------------------------- */

  $("layerName").addEventListener(
    "change",
    e => {

      const l = current();


      if (l) {

        l.name =
          e.target.value;


        renderLayers();

      }

    }
  );


  /* -------------------------
     LAYER OPACITY
  ------------------------- */

  $("layerOpacity").addEventListener(
    "input",
    e => {

      const l = current();


      if (l) {

        l.opacity =
          +e.target.value / 100;


        redraw();

      }

    }
  );


  /* -------------------------
     HIDE / SHOW
  ------------------------- */

  $("toggleLayer").onclick =
    () => {

      const l = current();


      if (l) {

        l.visible =
          !l.visible;


        redraw();

        renderLayers();

      }

    };


  /* -------------------------
     DELETE LAYER
  ------------------------- */

  $("deleteLayer").onclick =
    () => {

      if (state.layers.length < 2) {

        setStatus(
          "Keep at least one layer"
        );

        return;

      }


      snapshot();


      state.layers =
        state.layers.filter(
          l => l.id !== state.active
        );


      state.active =
        state.layers[
          state.layers.length - 1
        ].id;


      state.strokes =
        state.strokes.filter(
          s =>
            state.layers.some(
              l => l.id === s.layer
            )
        );


      renderLayers();

      redraw();

    };


  /* -------------------------
     UNDO
  ------------------------- */

  $("undo").onclick =
    () => {

      if (!state.undo.length) {
        return;
      }


      state.redo.push(
        JSON.stringify({
          layers: state.layers,
          active: state.active,
          strokes: state.strokes,
          vectors: state.vectors
        })
      );


      restore(
        state.undo.pop()
      );

    };


  /* -------------------------
     REDO
  ------------------------- */

  $("redo").onclick =
    () => {

      if (!state.redo.length) {
        return;
      }


      state.undo.push(
        JSON.stringify({
          layers: state.layers,
          active: state.active,
          strokes: state.strokes,
          vectors: state.vectors
        })
      );


      restore(
        state.redo.pop()
      );

    };


  /* -------------------------
     KEYBOARD
  ------------------------- */

  document.addEventListener(
    "keydown",
    e => {

      if (
        (e.ctrlKey || e.metaKey) &&
        e.key.toLowerCase() === "z"
      ) {

        e.preventDefault();


        if (e.shiftKey) {

          $("redo").click();

        } else {

          $("undo").click();

        }

      }


      if (
        e.target.matches("input")
      ) {

        return;

      }


      if (
        e.key.toLowerCase() === "b"
      ) {

        selectTool("brush");

      }


      if (
        e.key.toLowerCase() === "e"
      ) {

        selectTool("eraser");

      }

    }
  );


  function selectTool(t) {

    const b =
      document.querySelector(
        `[data-tool="${t}"]`
      );


    if (b) {
      b.click();
    }

  }


  /* -------------------------
     SVG IMPORT
  ------------------------- */

  $("importSvg").onclick =
    () => $("svgFile").click();


  $("svgFile").onchange =
    async e => {

      const f =
        e.target.files[0];


      if (!f) {
        return;
      }


      const svg =
        await f.text();


      if (
        !/<svg[\s>]/i.test(svg)
      ) {

        setStatus(
          "Invalid SVG file"
        );

        return;

      }


      snapshot();


      state.vectors.push({
        svg
      });


      redraw();


      setStatus(
        "SVG imported (preserved as vector markup)"
      );


      e.target.value = "";

    };


  /* -------------------------
     IMAGE IMPORT
  ------------------------- */

  $("importImage").onclick =
    () => $("imageFile").click();


  $("imageFile").onchange =
    e => {

      const f =
        e.target.files[0];


      if (!f) {
        return;
      }


      const url =
        URL.createObjectURL(f);


      const img =
        new Image();


      img.onload =
        () => {

          snapshot();


          ctx.drawImage(
            img,
            0,
            0,
            canvas.width,
            canvas.height
          );


          state.strokes.push({

            layer: state.active,

            tool: "brush",

            color: "#000",

            size: 1,

            opacity: 1,

            points: []

          });


          setStatus(
            "Image placed on canvas"
          );


          URL.revokeObjectURL(
            url
          );

        };


      img.src = url;


      e.target.value = "";

    };


  /* -------------------------
     NEW PROJECT
  ------------------------- */

  $("newProject").onclick =
    () => {

      if (
        !confirm(
          "Start a new project? Unsaved work will be lost."
        )
      ) {

        return;

      }


      state.layers = [

        {
          id: 1,

          name: "Layer 1",

          visible: true,

          opacity: 1,

          kind: "Raster"

        }

      ];


      state.active = 1;

      state.nextId = 2;

      state.strokes = [];

      state.vectors = [];

      state.undo = [];

      state.redo = [];


      redraw();

      renderLayers();

    };


  /* -------------------------
     SAVE PROJECT
  ------------------------- */

  $("saveProject").onclick =
    () => {

      const data = {

        version: 1,

        width: canvas.width,

        height: canvas.height,

        layers: state.layers,

        active: state.active,

        nextId: state.nextId,

        strokes: state.strokes,

        vectors: state.vectors

      };


      download(

        new Blob(
          [
            JSON.stringify(data)
          ],
          {
            type:
              "application/json"
          }
        ),

        "canvas-studio-project.json"

      );

    };


  /* -------------------------
     OPEN PROJECT
  ------------------------- */

  $("openProject").onclick =
    () => $("projectFile").click();


  $("projectFile").onchange =
    async e => {

      const f =
        e.target.files[0];


      if (!f) {
        return;
      }


      try {

        const d =
          JSON.parse(
            await f.text()
          );


        if (
          !d.layers ||
          !d.strokes
        ) {

          throw Error();

        }


        snapshot();


        state.layers =
          d.layers;


        state.active =
          d.active ||
          d.layers[0].id;


        state.nextId =
          d.nextId ||
          d.layers.length + 1;


        state.strokes =
          d.strokes;


        state.vectors =
          d.vectors || [];


        redraw();

        renderLayers();


        setStatus(
          "Project opened"
        );

      }

      catch {

        setStatus(
          "Could not open this project file"
        );

      }


      e.target.value = "";

    };


  /* -------------------------
     EXPORT PNG
  ------------------------- */

  $("exportPng").onclick =
    () => {

      const out =
        document.createElement(
          "canvas"
        );


      out.width =
        canvas.width;


      out.height =
        canvas.height;


      out
        .getContext("2d")
        .drawImage(
          canvas,
          0,
          0
        );


      download(

        dataURLBlob(
          out.toDataURL(
            "image/png"
          )
        ),

        "canvas-studio.png"

      );

    };


  /* -------------------------
     EXPORT SVG
  ------------------------- */

  $("exportSvg").onclick =
    () => {

      const vector =
        state.vectors
          .map(v => v.svg)
          .join("\n");


      const paths =
        state.strokes
          .map(s => {

            if (
              !s.points.length
            ) {

              return "";

            }


            const d =
              s.points
                .map(
                  (p, i) =>
                    (i ? "L" : "M") +
                    p.x +
                    " " +
                    p.y
                )
                .join(" ");


            return `
              <path
                d="${d}"
                fill="none"
                stroke="${s.color}"
                stroke-width="${s.size}"
                stroke-linecap="round"
                opacity="${s.opacity}"
              />
            `;

          })
          .join("\n");


      const svg = `

        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="${canvas.width}"
          height="${canvas.height}"
          viewBox="0 0 ${canvas.width} ${canvas.height}"
        >

          <rect
            width="100%"
            height="100%"
            fill="white"
          />

          ${paths}

          ${vector}

        </svg>

      `;


      download(

        new Blob(
          [svg],
          {
            type:
              "image/svg+xml"
          }
        ),

        "canvas-studio.svg"

      );

    };


  /* -------------------------
     DOWNLOAD HELPERS
  ------------------------- */

  function dataURLBlob(url) {

    const [
      head,
      data
    ] = url.split(",");


    const bytes =
      atob(data);


    const arr =
      new Uint8Array(
        bytes.length
      );


    for (
      let i = 0;
      i < bytes.length;
      i++
    ) {

      arr[i] =
        bytes.charCodeAt(i);

    }


    return new Blob(
      [arr],
      {
        type: "image/png"
      }
    );

  }


  function download(
    blob,
    name
  ) {

    const a =
      document.createElement("a");


    a.href =
      URL.createObjectURL(
        blob
      );


    a.download = name;


    a.click();


    setTimeout(
      () =>
        URL.revokeObjectURL(
          a.href
        ),
      1000
    );

  }


  /* -------------------------
     INITIALIZE
  ------------------------- */

  renderLayers();

  redraw();

})();
