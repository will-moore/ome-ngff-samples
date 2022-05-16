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
  "discussionUrl": "https://github.com/ome/idr-ome-ngff-samples/issues"
}
</script>

<table class="display" id="table">
    <thead>
<!-- TODO: should be read from data file -->
        <tr>
            <th>OME-NGFF version</th>
            <th>Thumbnail</th>
            <th>EMBL-EBI S3 key</th>
            <th>SizeX</th>
            <th>SizeY</th>
            <th>SizeZ</th>
            <th>SizeC</th>
            <th>SizeT</th>
            <th>Axes</th>
            <th>Wells</th>
            <th>Keywords</th>
            <th>Name</th>
            <th>Study</th>
            <th>DOI</th>
            <th>Date added</th>
        </tr>
    </thead>
    <tbody>
{% assign s3key = "EMBL-EBI bucket (current)" %}
{% for rec in site.data.table %}
{% capture image_name %}{{ rec.[s3key] | split: "/" | last }}{% endcapture %}
{% capture image_id %}{{ image_name | split: "." | first}}{% endcapture %}
        <tr>
            <td>{{ rec["OME-NGFF version"] }}</td>
            <td>
                <img
                    alt="IDR thumbnail for image:{{image_id}}"
                    style="margin:0"
                    src="https://idr.openmicroscopy.org/webclient/render_thumbnail/{{image_id}}/"/>
            </td>
            <td>
                <a href="{{ rec.[s3key] }}">
                    {{ image_name }}
                </a>
            </td>
            <td>{{ rec.["SizeX"] }}</td>
            <td>{{ rec.["SizeY"] }}</td>
            <td>{{ rec.["SizeZ"] }}</td>
            <td>{{ rec.["SizeC"] }}</td>
            <td>{{ rec.["SizeT"] }}</td>
            <td>{{ rec.["Axes"] }}</td>
            <td>{{ rec.["Wells"] }}</td>
            <td>{{ rec.["Keywords"] }}</td>
            <td>{{ rec.["Name"] }}</td>
            <td>{{ rec.["Study"] }}</td>
            <td>{{ rec.["DOI"] }}</td>
            <td>{{ rec.["Date added"] }}</td>
        </tr>
{% endfor %}
    </tbody>
</table>

<script>
$(document).ready( function () {
    $('#table').DataTable( {
          "scrollX": true
    });
} );
</script>
