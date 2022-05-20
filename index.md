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

<table class="display table" id="table">
    <thead>
<!-- TODO: should be read from data file -->
        <tr>
            <th>OME-NGFF version</th>
            <th>Thumbnail</th>
            <th>EMBL-EBI S3 key</th>
            <th>View in IDR</th>
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
                <a href="http://hms-dbmi.github.io/vizarr/?source={{ rec[s3key] }}">
                    <img
                        alt="IDR thumbnail for image:{{image_id}}"
                        style="margin:0"
                        {% if rec["Thumb ID"] %}
                        src="https://idr.openmicroscopy.org/webclient/render_thumbnail/{{ rec["Thumb ID"] }}/"
                        {% else %}
                        src="https://idr.openmicroscopy.org/webclient/render_thumbnail/{{image_id}}/"
                        {% endif %}
                    />
                </a>
            </td>
            <td>
                <a href="{{ rec[s3key] }}">
                    {{ image_name }}
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
</script>
