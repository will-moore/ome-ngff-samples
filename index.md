---
title: "IDR OME-NGFF Samples"
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
            <th>EMBL-EBI bucket (current)</th>
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
        </tr>
    </thead>
    <tbody>
{% for rec in site.data.table %}
        <tr>
            <td>{{ rec["OME-NGFF version"] }}</td>
            <td>{{ rec.["EMBL-EBI bucket (current)"] }}</td>
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
