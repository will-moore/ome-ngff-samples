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

    import { slice, addCodec, openArray } from "https://cdn.skypack.dev/zarr";


    async function exampleES6() {

        let attrs = await fetch(url + "/.zattrs").then(rsp => rsp.json());
        console.log("attrs", attrs);
        let paths = attrs.multiscales[0].datasets.map(d => d.path);
        let lastPath = paths[paths.length - 2];

        const channels = rootAttrs.omero.channels;
        const ranges = channels.map((ch) => {return {min: ch.window.start, max: ch.window.end}});

        // Dynamic import 
        addCodec("blosc", async () => (await import("https://cdn.skypack.dev/numcodecs/blosc")).default);
        const z = await openArray({ store: url + "/" + lastPath });

        let c = 0;
        let zarray = await z.get([0, c, 80, null, null]);
        console.log(zarray);


        const RED = 0;
        const GREEN = 1;
        const BLUE = 2;


        
        // Need to interleave the channels in RGBA buffer...
        const rgba = new Uint8ClampedArray(4 * zarray.data.length).fill(0);
        let offset = 0;
        for (let i = 0; i < raw_chs[0].data.length; i++) {
            for (let c=0; c < raw_chs.length; c++) {
            let v = ((raw_chs[c].data[i] - ranges[c].min) / (ranges[c].max - ranges[c].min));
            rgba[offset + c] = (v * 256) << 0;
            }
            rgba[offset + 3] = 255; // alpha
            offset += 4;
        }
        return rgba;
    }

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

async function renderRegion(path, region, range) {
    const zarray = await loadRegion(path, region);
    const shape = zarray.shape;
    const data = zarray.data;
    const height = shape[0];
    const width = shape[1];

    let red = 0;
    let green = 1;
    let blue = 2;
    let alpha = 3;
    console.log("SHAPE", shape, shape[0] * shape[1]);
    console.log("data.length", data.length, data.length * 4)

    const rgba = new Uint8ClampedArray(4 * height * width).fill(0);
    let offset = 0;
    console.log(range)
    let maxPixel = 0
    let maxValue = 0;
    for (let y = 0; y < height; y++) {
        for (let x = 0; x < width; x++) {
        // for (let c=0; c < raw_chs.length; c++) {
            let rawValue = data[y][x];
            maxPixel = Math.max(maxPixel, rawValue);
            let fraction = ((rawValue - range[0]) / (range[1] - range[0]));
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
        const dims = shape.length;
        const sizeX = shape[dims - 1];
        const sizeY = shape[dims - 2];

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

        for(let c=0; c<channelRanges.length; c++) {
            log("Render region: c: " + c);
            await renderRegion(path, [0, c, 200, ySlice, xSlice], [0, 10000]);
        } 
    };
})();

</script>
