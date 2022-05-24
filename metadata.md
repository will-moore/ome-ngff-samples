---
title: "Show metadata summary for OME-NGFF data"
---
<script type="application/ld+json">
{
  "@context": "http://schema.org",
  "@type": "Catalog",
  "inLanguage": "en-US",
  "name": "IDR OME-NGFF Samples"
  "publisher": {
    "@type": "Organization",
    "name": "GitHub"
  },
  "copyrightYear": "2022",
  "discussionUrl": "https://github.com/IDR/ome-ngff-samples/issues"
}
</script>

<div id="log">
<pre>Logging...</pre>
</div>

<script type="module">

import { slice, openArray } from "https://cdn.skypack.dev/zarr";

function log(text) {
    const el = document.createElement("pre");
    el.innerHTML = text;
    document.getElementById("log").appendChild(el);
}

function logJson(data) {
    log(JSON.stringify(data, null, 2));
}

async function getJson(url) {
    return await fetch(url).then(rsp => rsp.json());
}

async function loadRegion(path, region) {
    // E.g. region = [1,1,1,"0:100", "0:100"]
    region = region.map(dim => {
        if (typeof dim === "string") {
            let startStop = dim.split(":").map(d => parseInt(d));
            console.log("startStop", startStop, dim.split(":"));
            return startStop.length === 2 ? slice(...startStop) : startStop[0];
        }
        return dim;
    });
    console.log("region", region);
    const z = await openArray({ store: source + path });
    let zarray = await z.get(region);
    console.log(zarray);
    return zarray;
}

function getMinMax(zarray) {
    const shape = zarray.shape;
    const data = zarray.data;
    const height = shape[0];
    const width = shape[1];
    let minVal = Infinity;
    let maxVal = -Infinity;
    for (let y = 0; y < height; y++) {
        for (let x = 0; x < width; x++) {
            let rawValue = data[y][x];
            if (rawValue < minVal) minVal = rawValue;
            if (rawValue > maxVal) maxVal = rawValue;
        }
    }
    return [minVal, maxVal];
}

async function renderRegion(path, region) {
    const zarray = await loadRegion(path, region);
    const shape = zarray.shape;
    const data = zarray.data;
    const height = shape[0];
    const width = shape[1];
    const range = getMinMax(zarray);

    let red = 0;
    let green = 1;
    let blue = 2;
    let alpha = 3;
    console.log("SHAPE", shape, shape[0] * shape[1]);
    console.log("data.length", data.length, data.length * 4)

    const rgba = new Uint8ClampedArray(4 * height * width).fill(0);
    let offset = 0;
    let maxPixel = 0
    let maxValue = 0;
    for (let y = 0; y < height; y++) {
        for (let x = 0; x < width; x++) {
        // for (let c=0; c < raw_chs.length; c++) {
            let rawValue = data[y][x];
            maxPixel = Math.max(maxPixel, rawValue);
            let fraction = ((rawValue - range[0]) / (range[1] - range[0]));
            fraction = fraction * 2;
            let v = (fraction * 256) << 0;
            maxValue = Math.max(maxValue, v);
            rgba[offset + red] = v;
            rgba[offset + green] = v;
            rgba[offset + blue] = v;
            rgba[offset + alpha] = 255; // alpha
            offset += 4;
        }
    }
    console.log("maxValue", maxValue, "maxPixel", maxPixel);
    console.log("rgba", rgba);
    logCanvas(rgba, width, height);
}

function logCanvas(rgba, width, height) {
  // Create image and draw to canvas
  const img = new ImageData(rgba, width, height);
  // add canvas...
  let canvas = document.createElement("canvas");
  canvas.width = width;
  canvas.height = height;
  document.getElementById("log").appendChild(canvas);
//   var canvas = document.getElementById('canvas');
  var ctx = canvas.getContext('2d');
  ctx.putImageData(img, 0, 0);
}


(async function() {
    const searchParams = new URLSearchParams(window.location.search);
    let source = searchParams.get('source');
    if (!source) {
        log("Use e.g. ?source=https://uk1s3.embassy.ebi.ac.uk/idr/zarr/v0.3/9836842.zarr to load OME-NGFF Image")
    }

    if (!source.endsWith("/")) {
        source = source + "/";
    }
    window.source = source;

    console.log("source", source);
    log("source " + source);

    const zattrsUrl = source + ".zattrs"
    log("Fetching... " + zattrsUrl);

    let rootAttrs = await getJson(zattrsUrl);
    logJson(rootAttrs);

    let paths = rootAttrs.multiscales[0].datasets.map(d => d.path);
    let axesNames = ["t", "c", "z", "y", "x"];
    let axes = rootAttrs.multiscales[0].axes;
    if (axes) {
        axesNames = axes.map(axis => axis.name ? axis.name : axis);
    }
    log("Axes: " + JSON.stringify(axesNames));

    log("paths");
    logJson(paths);

    let channelRanges = [];
    if (rootAttrs?.omero?.channels) {
        channelRanges = rootAttrs?.omero?.channels.map(channel => {
            return [channel.window.start, channel.window.end];
        })
    }
    log("channelRanges")
    logJson(channelRanges);

    for (let i=paths.length - 1; i>=0; i--) {
        let path = paths[i];
        let arrayUrl = source + path;
        let zAttrsUrl = arrayUrl + "/.zarray";
        log("Loading..." + zAttrsUrl)
        let arrayAttrs = await getJson(zAttrsUrl);
        console.log(arrayAttrs);

        const shape = arrayAttrs.shape;
        log("Shape: " + JSON.stringify(shape));
        const nDims = shape.length;
        const sizeX = shape[nDims - 1];
        const sizeY = shape[nDims - 2];

        let xSlice = null;
        let ySlice = null;

        if (sizeX > 512) {
            // xSlice = "0:512";
            continue
        }
        if (sizeY > 512) {
            // ySlice = "0:512";
            continue;
        }

        let channelDim = axesNames.indexOf("c");
        let sizeC = shape[channelDim] || 1;
        console.log("sizeC", sizeC);
        console.log("axesNames", axesNames);

        let dims = axesNames.map((axis, index) => {
            console.log("axis, index", axis, index, axis === 'x' || axis === 'y');
            if (axis === 'z') {
                // Mid-point in Z-stack
                let sizeZ = shape[index];
                console.log('sizeZ', sizeZ);
                return parseInt(sizeZ / 2);
            }
            if (axis === 't' ) {
                // Start of time-lapse
                return 0
            }
            if (axis === 'x' || axis === 'y') {
                return null;
            }
            return 0;
        });
        console.log("channelDim", channelDim);
        console.log("dims", dims);

        for(let c=0; c<sizeC; c++) {
            log("Render region: c: " + c);
            dims[channelDim] = c;
            await renderRegion(path, dims);
        } 
    };
})();

</script>
