---
permalink: /searchresults/
title: "Search Results"
header: 
    # overlay_color: "#333"
    overlay_image: "/assets/images/header.png"
    overlay_filter: 0.5
    caption: "Diagram Credit: [**EPIGENTEK**](https://www.epigentek.com/)"
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
<style>
#result-table {
  /* Remove default list styling */
  list-style-type: none;
  padding: 0;
  margin: 0;
}

#result-table li a {
  border: 1px solid #ddd; /* Add a border to all links */
  margin-top: -1px; /* Prevent double borders */
  background-color: #f6f6f6; /* Grey background color */
  padding: 12px; /* Add some padding */
  text-decoration: none; /* Remove default text underline */
  font-size: 18px; /* Increase the font-size */
  color: black; /* Add a black text color */
  display: block; /* Make it into a block element to fill the whole list */
}

#result-table li a:hover:not(.header) {
  background-color: #eee; /* Add a hover effect to all links, except for headers */
}
</style>
<script src="https://cdn.bootcdn.net/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<script src="https://cdn.bootcdn.net/ajax/libs/json2/20160511/json2.min.js"></script>
<script>
function GetQueryString(name) {
    var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
    var r = window.location.search.substr(1).match(reg);
    if(r != null) return decodeURI(r[2]);
    return null;
};
function UpdatePagination(abidxs) {
    $("#pagination-para").html("Page: ");
    var numrows = $("#pagination-numrow-select").val();
    var numpages = Math.ceil(abidxs.length / numrows);
    const shownumpages = 3;
    for (let i = 1; i <= numpages; i++) {
        $("#pagination-para").append("<a name='page-button' id='page-" + i + "' href='#searchresults' class='btn btn--inverse'>" + i + "</a> ");
        if ((i <= shownumpages) || (i === numpages)) {
        } else {
            $("#page-" + i).hide();
        };
        if (i === 1) {
            $("#pagination-para").append("<span id='page-sep-first'> </span>");
        } else if (i === numpages-1) {
            $("#pagination-para").append("<span id='page-sep-last'>... </span>");
        } else {
        };
    };
    $("#page-1").attr("class", "btn btn--light-outline");
};
function ChangePage(pageindex) {
    var pageindex = parseInt(pageindex);
    const shownumpages = 2;
    var numpages = $("#pagination-para a").length;
    $.each($("#pagination-para a"), function(index, element) {
        $(element).hide();
    });
    $("a[class='btn btn--light-outline']").attr("class", "btn btn--inverse");
    var firstpageindex = Math.max(1, pageindex - shownumpages);
    var lastpageindex = Math.min(pageindex + shownumpages, numpages);
    for (let i = firstpageindex; i <= lastpageindex; i++) {
        $("#page-" + i).show();
    };
    $("#page-" + pageindex).attr("class", "btn btn--light-outline");
    if (firstpageindex > 1) {
        $("#page-1").show();
    };
    if (lastpageindex < numpages) {
        $("#page-" + numpages).show();
    };
    if (firstpageindex > 2) {
        $("#page-sep-first").text("... ");
    } else {
        $("#page-sep-first").text(" ");
    };
    if (lastpageindex < numpages-1) {
        $("#page-sep-last").text("... ");
    } else {
        $("#page-sep-last").text(" ");
    };
};
function ShowSubsetResults(result_abidxs, result_names, pageindex) {
    var numrows = $("#pagination-numrow-select").val();
    var subset_abidxs = result_abidxs.slice((pageindex-1)*numrows, pageindex*numrows);
    $("#result-table").html("");
    for (var abidx of subset_abidxs) {
        $("#result-table").append("<li><a href='../abdetail/?ab_idx=" + abidx + "'>ID: "+ abidx + " <strong>" + result_names[abidx] + "</strong></a></li>");
    };
}
$(document).ready(function() {
    $("#loading-para").hide()
    var result_abidxs = GetQueryString("ab_idxs");
    var result_abidxs = result_abidxs.split(",");
    var num_results = result_abidxs.length;
    const maxnum_results = 400;
    if (num_results === maxnum_results) {
        $("#results-count").html("Too many matches. Showing first 400 results. Try narrowing your search.");
    } else {
        $("#results-count").html(num_results + " matches found.");
    };
    $.ajax({
            url: "../_data/tables/name.csv",
            type: "GET",
            async: false,
            dataType: "text",
            success: function(data) {
                var parsed_name = $.csv.toObjects(data);
                var result_names = {};
                for (var i = 0; i < parsed_name.length; i++) {
                    if (result_abidxs.includes(parsed_name[i]["ab_idx"])) {
                        result_names[parsed_name[i]["ab_idx"]] = parsed_name[i]["all_names"];
                    };
                };
                UpdatePagination(result_abidxs);
                $("a[name='page-button']").click(function() {
                    var pageindex = $(this).text();
                    ChangePage(pageindex);
                    ShowSubsetResults(result_abidxs, result_names, pageindex);
                });
                $("#page-1").click();
            }
    });
});

</script>
<h1 id="searchresults">Results</h1>
<div id="loading-para">Loading...</div>
<div><span id="results-count"></span></div>
<div>Number of rows: 
<select name="preview-numrow" id="pagination-numrow-select">
  <option selected>20</option>
  <option>50</option>
  <option>100</option>
  <option>200</option>
</select></div>
<div><span id="pagination-para" style="float: left">Page: </span><span style="float: right"><label for="pagination-input" style="display: flex"><a id="pagination-select" href="#" class="btn btn--primary">Jump to Page</a><input type="number" id="pagination-input" maxlength="4" style="width: 3em" min="1"></label></span></div><br>
<ul id="result-table"></ul>