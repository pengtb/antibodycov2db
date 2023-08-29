---
permalink: /browse/
title: "Browse"
header: 
    overlay_color: "#333"
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
# Browse datasets
Select a processed dataset:
<select name="dataset2dl" id="dataset-select">
  <option value="classification">classification</option>
  <option value="ranking" selected>ranking</option>
  <option value="epitope prediction">epitope prediction</option>
</select><br>

#### Introduction
<p id="dataset-intro-1"></p>
<p id="dataset-intro-2"></p>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
<!-- change introduction according to selected -->
<script>
function UpdateDatasetIntro() {
    var table2dl = $("#dataset-select").val();
    if (table2dl === "classification") {
      $("#dataset-intro-1").html("The <em><strong>classification</strong></em> dataset includes all collected antibodies that show direct binding (or not binding) evidence to WT or mutant SARS-CoV2 spike RBD. The dataset is designed for discovery of new binding antibodies only using <em>heavy/light chain variable-domain sequences</em> of antibodies and sequence of RBD. <em>Region information</em> of sequences (a.k.a. CDR/FR) is also included for prediction.");
      $("#dataset-intro-2").html("For collected samples are mostly positive (mainly because of manual selection), antibodies showing clear binding evidence to other epitopes of spike protein (such as NTD) or even other proteins are included as well as negative samples. The dataset is split into training, validation and test sets, as shown in the <em>ds</em> column, meanwhile positive/negative samples and samples targeting variants of RBD are both evenly distributed in each set. The test set can be further split into 3 subset: unseen WT, unseen Omicron and seen Omicron, as shown in the <em>usage</em> column. \"Unseen\" samples are those not seen in the training/validation sets, while \"seen\" samples are the opposite.");
    } else if (table2dl === "ranking") {
      $("#dataset-intro-1").html("The <em><strong>ranking</strong></em> dataset includes all collected antibodies that have quantitative binding affinities against WT RBD. The dataset is designed for ranking of binding antibodies only using <em>heavy/light chain variable-domain sequences</em> of antibodies. <em>Region information</em> of sequences (a.k.a. CDR/FR) is also included for prediction. Additionally, binding affinities measured under various conditions are all included to increase number of samples. Specifically, the experimental methods can be BLI or SPR, as shown in the <em>evidence</em> column and forms of antibodies can be IgG or Fab, as shown in the <em>ab_type</em> column.");
      $("#dataset-intro-2").html("The dataset is split into training, validation and test sets, as shown in the <em>ds</em> column, meanwhile samples of difference scales of binding affinities (a.k.a. log<sub>10</sub> of binding affinities) are evenly distributed in each set. Split of dataset is shown in the <em>ds</em> column.");
    } else if (table2dl === "epitope prediction") {
      $("#dataset-intro-1").html("The <em><strong>epitope prediction</strong></em> dataset includes all collected antibodies that have complex structures with WT or mutant SARS-CoV2 spike RBD. The dataset is designed for prediction of epitopes (a.k.a. binding sites of RBD) only using <em>heavy/light chain variable-domain sequences</em> of antibodies and sequence of RBD. Epitopes are represented as binary vectors of length 223, indicating which residues are the binding sites of RBD, as shown in the <em>rbd_contacts</em> column. To remove redundancy, for antibodies with multiple complex structures available, only the one with most contacts to RBD is kept. <em>Region information</em> of antibody sequences (a.k.a. CDR/FR) is also included for prediction. ");
      $("#dataset-intro-2").html("The dataset is split into training, validation and test sets, as shown in the <em>ds</em> column, meanwhile samples of different epitope groups (annotated as ith key residues of each epitope group from <a href=\"https://www.nature.com/articles/s41586-022-04980-y\">Cao's work</a>) and samples targeting variants of RBD are both evenly distributed in each set. Split of dataset is shown in the <em>ds</em> column. The test set can be further split into 3 subset: unseen WT, unseen Omicron and seen Omicron, as shown in the <em>usage</em> column. \"Unseen\" samples are those not seen in the training/validation sets, while \"seen\" samples are the opposite.");
    } else {
      $("#dataset-intro-1").text("");
    }
}
$(document).ready(function(){
    UpdateDatasetIntro();
    $("#dataset-select").change(UpdateDatasetIntro);
});
</script>

#### Browse
<div>Number of rows: 
<select name="preview-numrow" id="numrow-select">
  <option value="10" selected>10</option>
  <option value="50">50</option>
  <option value="100">100</option>
</select></div>
<p><span>Columns to show:  </span><span style="float: right"><a href="#browse" class="btn btn--primary" id="preview-button">Apply</a></span></p>
<style>
label.labelcheckbox {
  display: flex;
  text-indent: 15px;
  padding: 5px;
}
div.divcheckbox {
  display: inline-block;
  line-height: 0.75;
}
</style>
<form id="dscolumns-select"></form>
<div><span id="pagination-para" style="float: left">Page: </span><span style="float: right"><label for="page-input" style="display: flex"><a id="page-select" href="#browse" class="btn btn--primary">Jump to Page</a><input type="number" id="page-input" maxlength="4" style="width: 4em"></label></span></div>
<style>
td {
  white-space: nowrap;
}
</style>
<p class="text-center"><table id="table-browse">
<thead id="table-browse-header"></thead>
<tbody id="table-browse-body"></tbody>
</table></p>
<!-- load & show & update table -->
<script src="../assets/js/plugins/jquery.csv.js"></script>
<script>
function GetDsBasename(datasetname) {
  if (datasetname === "classification") {
    var filebasename = "classification_variantrbd";
  } else if (datasetname === "ranking") {
    var filebasename = "regression_wtrbd";
  } else {
    var filebasename = "epitope_variantrbd";
  };
  return filebasename
};
function LoadDataset() {
    var datasetname = $("#dataset-select").val();
    var filebasename = GetDsBasename(datasetname);
    var result = null;
    var filepath = "../_data/datasets/" + filebasename + ".tsv";
    $("#preview-button").text("Loading...");
    $.ajax({
        url: filepath,
        type: "GET",
        async: false,
        dataType: "text",
        success: function(data) {
            data = data.replace(/\t/g, ",");
            var parsed = $.csv.toObjects(data);
            result = parsed;
        }
    }).done(function() {
        $("#preview-button").text("Apply");
    });
    return result;
}
function UpdateDatasetColumns(parsed) {
    $("#dscolumns-select").html("");
    var checkbox_idx = 1;
    $.each(parsed[0], function(key, value) {
        $("#dscolumns-select").append("<div class=\"divcheckbox\"><label for=\"columns-select-box-"+checkbox_idx+"\" class=\"labelcheckbox\"><input type=\"checkbox\" id=\"columns-select-box-"+checkbox_idx+"\" checked />" + key + "</label></div>");
        checkbox_idx += 1;
    });
};
function UpdatePagination(parsed) {
    $("#pagination-para").html("Page: ");
    const numrows = $("#numrow-select").val();
    var numpages = Math.ceil(parsed.length / numrows);
    const shownumpages = 3;
    for (let i = 1; i <= numpages; i++) {
        $("#pagination-para").append("<a id=\"page-" + i + "\" href=\"#browse\" class=\"btn btn--inverse\">" + i + "</a> ");
        $("#page-" + i).click(function() {
            ChangePage(i);
            ShowTable(parsed, GetSelectedColumns(), i-1);
        });
        if ((i <= shownumpages) || (i === numpages)) {
        } else {
            $("#page-" + i).hide();
        };
        if (i === 1) {
            $("#pagination-para").append("<span id=\"page-sep-first\"> </span>");
        } else if (i === numpages-1) {
            $("#pagination-para").append("<span id=\"page-sep-last\">... </span>");
        } else {
        };
    };
    $("#page-1").attr("class", "btn btn--light-outline");
};
function ChangePage(pageindex) {
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
function GetSelectedColumns() {
  var checkedcolumns = [];
  for (checkedinput of document.querySelectorAll("#dscolumns-select input[type='checkbox']:checked")) {
    var checkboxlabel = checkedinput.parentElement;
    checkedcolumns.push(checkboxlabel.innerText);
  };
  return checkedcolumns
};
function ShowTable(parsed, checkedcolumns, startindex=0) {
    var numrow = $("#numrow-select").val();
    var startrowindex = startindex * numrow;
    var endrowindex = (startindex + 1) * numrow;
    $("#preview-button").text("Loading...");
    $("#table-browse-header").html("");
    $("#table-browse-header").append("<tr>");
    $.each(parsed[0], function(key, value) {
        if (typeof checkedcolumns !== "undefined") {
        if (checkedcolumns.includes(key)) {
            $("#table-browse-header").append("<th>" + key + "</th>");
        };
        } else {
        $("#table-browse-header").append("<th>" + key + "</th>");
        };
    });
    $("#table-browse-header").append("</tr>");
    $("#table-browse-body").html("");
    for (var i = startrowindex; i < endrowindex; i++) {
        $("#table-browse-body").append("<tr>");
        $.each(parsed[i], function(key, value) {
        if (typeof checkedcolumns !== "undefined") {
            if (checkedcolumns.includes(key)) {
            $("#table-browse-body").append("<td>" + value + "</td>");
            };
        } else {
            $("#table-browse-body").append("<td>" + value + "</td>");
        }
        });
        $("#table-browse-body").append("</tr>");
    };
    $("#preview-button").text("Apply");
};
$(document).ready(function(){
    var parsed = LoadDataset();
    UpdateDatasetColumns(parsed);
    UpdatePagination(parsed);
    ShowTable(parsed);
    $("#dataset-select").change(function() {
        parsed = LoadDataset();
        UpdateDatasetColumns(parsed);
        UpdatePagination(parsed);
        ShowTable(parsed);
    });
    $("#dscolumns-select").ready(function() {
        $("#preview-button").click(function() {
            var checkedcolumns = GetSelectedColumns();
            UpdatePagination(parsed);
            ShowTable(parsed, checkedcolumns);
        });
    });
    $("#page-select").click(function() {
        var pageindex = $("#page-input").val();
        if ((pageindex >= 1) && (pageindex <= $("#pagination-para a").length)) {
            $("#page-"+pageindex).click();        
        } else if (pageindex === "") {
        } else {
            alert("Invalid page number.");
        }
    });
});
</script>
