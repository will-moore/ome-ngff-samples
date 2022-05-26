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

function range(count) {
    return new Array(count).fill(0).map((a, i) => i)
}

function renderTo8bitArray(planes, channelColors, channelRanges) {
    const height = planes[0].shape[0];
    const width = planes[0].shape[1];
    const rgba = new Uint8ClampedArray(4 * height * width).fill(0);
    let offset = 0;
    for (let y = 0; y < height; y++) {
        for (let x = 0; x < width; x++) {
            for (let p=0; p < planes.length; p++) {
                let data = planes[p].data;
                let rgb = channelColors[p];
                let range = channelRanges[p];
                let rawValue = data[y][x];
                let fraction = ((rawValue - range[0]) / (range[1] - range[0]));
                // for red, green, blue, 
                for(let i=0; i<3; i++) {
                    if (rgb[i] > 0) {
                        // rgb[i] is 0-255...
                        let v = (fraction * rgb[i]) << 0;
                        // increase pixel intensity if value is higher
                        rgba[offset + i] = Math.max(rgba[offset + i], v);
                    }
                }
            }
            rgba[offset + 3] = 255; // alpha
            offset += 4;
        }
    }
    return rgba;
}

async function renderRegion(path, axesNames, shape, omeroChannels) {

    const nDims = shape.length;
    const sizeX = shape[nDims - 1];
    const sizeY = shape[nDims - 2];
    if (sizeX > 512 || sizeY > 512) {
        // Don't try to load too much data
        return;
    }

    let channelDim = axesNames.indexOf("c");
    let sizeC = shape[channelDim] || 1;
    console.log("sizeC", sizeC);
    console.log("axesNames", axesNames);

    let dims = getDefaultSlice(axesNames, shape)
    console.log("channelDim", channelDim);
    console.log("dims", dims);

    let slices = range(sizeC).map(c => {
        let sl = [...dims];
        sl[channelDim] = c; 
        return sl;
    });
    console.log("slices", slices);


    let channelRanges = [];
    let channelColors = [];
    if (omeroChannels) {
        channelRanges = omeroChannels.map(channel => {
            return [channel.window.start, channel.window.end];
        });
        channelColors = omeroChannels.map(channel => {
            return hexToRgb(channel.color);
        });
    }
    log("channelRanges")
    logJson(channelRanges);

    // load all channels...
    const planes = await Promise.all(slices.map(s => loadRegion(path, s)));
    console.log("planes", planes);
    const data = planes[0].data;
    const height = planes[0].shape[0];
    const width = planes[0].shape[1];

    console.log("data.length", data.length, data.length * 4)
    console.log("channelColors", channelColors);
    console.log("channelRanges", channelRanges);

    const rgba = renderTo8bitArray(planes, channelColors, channelRanges);
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

function getDefaultSlice(axesNames, shape) {
    return axesNames.map((axis, index) => {
        if (axis === 'z') {
            // Mid-point in Z-stack
            let sizeZ = shape[index];
            return parseInt(sizeZ / 2);
        }
        if (axis === 'x' || axis === 'y') {
            return null;
        }
        // t and c
        return 0;
    });
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

    for (let i=paths.length - 1; i>=0; i--) {
        let path = paths[i];
        let zAttrsUrl = source + path + "/.zarray";
        log("Loading..." + zAttrsUrl)
        let arrayAttrs = await getJson(zAttrsUrl);
        console.log(arrayAttrs);

        const shape = arrayAttrs.shape;
        log("Shape: " + JSON.stringify(shape));
        await renderRegion(source + path, axesNames, shape, rootAttrs.omero?.channels);
    };
})();

</script>
