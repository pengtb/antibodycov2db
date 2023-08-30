---
permalink: /abdetail/
title: "Details"
header: 
    # overlay_color: "#333"
    overlay_image: "/assets/images/header.png"
    overlay_filter: 0.5
    caption: "Credit: [**EPIGENTEK**](https://www.epigentek.com/)"
    # actions:
    #  - label: "Download All"
    #    url: "https://github.com"
classes: wide
csv_reader:
    converters:
      - all
    headers: true
    encoding: utf-8
tsv_reader:
    converters:
      - all
    headers: true
    encoding: utf-8
---
<!-- receive parameters from url -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
<script>
function GetQueryString(name) {
    var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
    var r = window.location.search.substr(1).match(reg);
    if(r != null) return decodeURI(r[2]);
    return null;
};
</script>
<!-- show collected info -->
<script src="../assets/js/plugins/jquery.csv.js"></script>
<script>
function LoadData(filepath, ab_idx, detailparaid, donecallback, showcolname) {
    $.ajax({
        url: filepath,
        type: "GET",
        async: false,
        dataType: "text",
        success: function(data) {
            var parsed = $.csv.toObjects(data);
            var subset = [];
            $.each(parsed, function(key, value) {
                if (value["ab_idx"] === ab_idx) {
                    subset.push(value);
                };
            });
            ShowData(subset, detailparaid, showcolname);
        }
    }).done(donecallback);
};
function ShowData(subset, detailparaid, colname) {
    if (subset.length > 1) {
        $("#"+detailparaid).html("<ul>");
        $.each(subset, function(key, value) {
        $("#"+detailparaid).append("<li>" + value[colname] + "</li>");
        });
        $("#"+detailparaid).append("</ul>");
    } else {
        $("#"+detailparaid).html(subset[0][colname]);
    };
    
};
$(document).ready(function(){
    var ab_idx = GetQueryString("ab_idx");
    LoadData("../_data/tables/name.csv", ab_idx, "detail-all_names", function() {}, "all_names");
});
</script>

# Binding evidence
<p class="notice"><span>All curated <em><strong>names</strong></em> of this antibody: </span><span id="detail-all_names"></span></p>

# Sequences

# Structures
#### PDB references

#### Epitopes

#### Predicted structures by IgFold