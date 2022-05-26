---
title: "Catalog of IDR images formatted as OME-NGFF"
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

<style>
    .page-content .wrapper {
        box-sizing: border-box;
        width: 100%;
        max-width: 100%;
    }
    .dataTables_scrollHeadInner {
        margin: 0 auto;
    }
    .icon {
        width: 24px;
        height: 24px;
    }
    .no_border {
        border: none;
        background: none;
        padding: 0;
    }
</style>

<table class="display table" id="table">
    <thead>
<!-- TODO: should be read from data file -->
        <tr>
            <th>OME-NGFF version</th>
            <th>Thumbnail (open in <a target="_blank" href="https://github.com/hms-dbmi/vizarr">Vizarr</a>)</th>
            <th>EMBL-EBI S3 key</th>
            <th>SizeX</th>
            <th>SizeY</th>
            <th>SizeZ</th>
            <th>SizeC</th>
            <th>SizeT</th>
            <th>Axes</th>
            <th>Wells</th>
            <th>Fields</th>
            <th>Keywords</th>
            <th>License</th>
            <th>Study</th>
            <th>View in IDR</th>
            <th>DOI</th>
            <th>Date added</th>
        </tr>
    </thead>
    <tbody>
{% assign s3key = "EMBL-EBI bucket (current)" %}
{% assign studykey = "Study" %}
{% for rec in site.data.table %}
{% capture image_name %}{{ rec.[s3key] | split: "/" | last }}{% endcapture %}
{% capture image_id %}{{ image_name | split: "." | first}}{% endcapture %}
        <tr>
            <td>{{ rec["OME-NGFF version"] }}</td>
            <td>
                <a target="_blank"
                    title="Open NGFF {% if rec['Wells'] %}Plate{% else %}Image{% endif %} in Vizarr"
                    href="http://hms-dbmi.github.io/vizarr/?source={{ rec[s3key] }}">
                    <img
                        alt="IDR thumbnail for Image:{{ rec["Representative Image ID"] }}"
                        style="margin:0"
                        src="https://idr.openmicroscopy.org/webclient/render_thumbnail/{{ rec["Representative Image ID"] }}/"
                    />
                </a>
            </td>
            <td>
                <a href="{{ rec[s3key] }}">
                    {{ image_name }}
                </a><br>
                <button class="no_border" title="Copy S3 URL to clipboard" onclick="copyTextToClipboard('{{ rec[s3key] }}')">
                    <img class="icon" src="assets/img/copy.png"/>
                </button>
                <a title="View NGFF {% if rec['Wells'] %}Plate{% else %}Image{% endif %} in Vizarr" target="_blank"
                    href="http://hms-dbmi.github.io/vizarr/?source={{ rec[s3key] }}">
                    <img class="icon" src="assets/img/vizarr.png"/></a>
                <a title="Validate NGFF with 'ome-ngff-validator' in new browser tab" target="_blank"
                    href="https://ome.github.io/ome-ngff-validator/?source={{ rec[s3key] }}">
                    <img class="icon" src="assets/img/check.png"/></a>
            </td>
            <td>{{ rec.["SizeX"] }}</td>
            <td>{{ rec.["SizeY"] }}</td>
            <td>{{ rec.["SizeZ"] }}</td>
            <td>{{ rec.["SizeC"] }}</td>
            <td>{{ rec.["SizeT"] }}</td>
            <td>{{ rec.["Axes"] }}</td>
            <td>{{ rec.["Wells"] }}</td>
            <td>{{ rec.["Fields"] }}</td>
            <td>{{ rec.["Keywords"] }}</td>
            <td>{{ rec.["License"] }}</td>
            <td>
                <a href="https://idr.openmicroscopy.org/search/?query=Name:{{ rec[studykey] }}">
                    {{ rec.["Study"] }}
                </a>
            </td>
            <td>
                {% if rec["Wells"] %}
                    <a target="_blank" href="https://idr.openmicroscopy.org/webclient/?show=plate-{{ image_id }}">
                        Plate in IDR
                    </a>
                {% else %}
                    <a target="_blank" href="https://idr.openmicroscopy.org/webclient/img_detail/{{ image_id }}/">
                        Image in IDR
                    </a>
                {% endif %}
            </td>
            <td>{{ rec.["DOI"] }}</td>
            <td>{{ rec.["Date added"] }}</td>
        </tr>
{% endfor %}
    </tbody>
</table>

<script>
$(document).ready( function () {
    $('#table').DataTable( {
          "scrollX": true,
          "pageLength": 100
    });
} );


function copyTextToClipboard(text) {
    var textArea = document.createElement("textarea");
    // Place in the top-left corner of screen regardless of scroll position.
    textArea.style.position = 'fixed';

    textArea.value = text;

    document.body.appendChild(textArea);
    textArea.focus();
    textArea.select();

    var successful;
    try {
        successful = document.execCommand('copy');
    } catch (err) {
        console.log('Oops, unable to copy');
    }
    document.body.removeChild(textArea);

    if (successful) {
        // show user that copying happened - update text on element (e.g. button)
        let target = event.target;
        let html = target.innerHTML;
        target.innerHTML = "Copied!"
        setTimeout(() => {
            // reset after 1 second
            target.innerHTML = html
        }, 1000)
    } else {
        console.log("Copying failed")
    }
}
</script>
