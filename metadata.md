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

function hexToRgb(color) {
    return [0, 2, 4].map(i => parseInt(color.slice(i, i + 2), 16));
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

async function renderRegion(path, slices, channelColors, channelRanges) {

    // load all channels...
    const planes = await Promise.all(slices.map(s => loadRegion(path, s)));
    console.log("planes", planes);
    const shape = planes[0].shape;
    const data = planes[0].data;
    const height = shape[0];
    const width = shape[1];
    // const range = getMinMax(planes[0]);

    console.log("SHAPE", shape, height * width);
    console.log("data.length", data.length, data.length * 4)
    console.log("channelColors", channelColors);
    console.log("channelRanges", channelRanges);

    const rgba = new Uint8ClampedArray(4 * height * width).fill(0);
    let offset = 0;
    // let maxFraction = 0;
    // let maxValue = 0;
    // let maxRaw = 0;
    for (let y = 0; y < height; y++) {
        for (let x = 0; x < width; x++) {
            for (let p=0; p < planes.length; p++) {
                let data = planes[p].data;
                let rgb = channelColors[p];
                let range = channelRanges[p];
                // console.log({x, y, data});
                let rawValue = data[y][x];
                // maxRaw = Math.max(maxRaw, rawValue);
                // range = [0, 55000]
                let fraction = ((rawValue - range[0]) / (range[1] - range[0]));
                // maxFraction = Math.max(fraction, maxFraction);
                // fraction = fraction * 2; // boost
                for(let i=0; i<3; i++) {
                    if (rgb[i] > 0) {
                        let v = (fraction * rgb[i]) << 0;
                        // maxValue = Math.max(maxValue, v);
                        rgba[offset + i] = Math.max(rgba[offset + i], v);
                    }
                }
                // rgba[offset + 1] = (fraction * rgb[1]) << 0;
                // rgba[offset + 2] = (fraction * rgb[2]) << 0;
            }
            rgba[offset + 3] = 255; // alpha
            offset += 4;
        }
    }
    // console.log("maxRaw", maxRaw);
    // console.log("maxFraction", maxFraction);
    // console.log("maxValue", maxValue);
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
    let channelColors = [];
    if (rootAttrs?.omero?.channels) {
        let channels = rootAttrs.omero.channels;
        channelRanges = channels.map(channel => {
            return [channel.window.start, channel.window.end];
        });
        channelColors = channels.map(channel => {
            return hexToRgb(channel.color);
        });
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

        let slices = channelColors.map((color, c) => {
            let sl = [...dims];
            sl[channelDim] = c; 
            return sl;
        });
        console.log("slices", slices);


        await renderRegion(path, slices, channelColors, channelRanges);
        
    };
})();

</script>
