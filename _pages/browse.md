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
  <option value="classification" selected>classification</option>
  <option value="ranking">ranking</option>
  <option value="epitope prediction">epitope prediction</option>
</select><br>

#### Introduction
<p id="dataset-intro-1">The <em><strong>classification</strong></em> dataset includes all collected antibodies that show direct binding (or not binding) evidence to WT or mutant SARS-CoV2 spike RBD. The dataset is designed for discovery of new binding antibodies only using <em>heavy/light chain variable-domain sequences</em> of antibodies and sequence of RBD. <em>Region information</em> of sequences (a.k.a. CDR/FR) is also included for prediction.</p>
<p id="dataset-intro-2">For collected samples are mostly positive (mainly because of manual selection), antibodies showing clear binding evidence to other epitopes of spike protein (such as NTD) or even other proteins are included as well as negative samples. The dataset is split into training, validation and test sets, as shown in the <em>ds</em> column, meanwhile positive/negative samples and samples targeting variants of RBD are both evenly distributed in each set. The test set can be further split into 3 subset: unseen WT, unseen Omicron and seen Omicron, as shown in the <em>usage</em> column. "Unseen" samples are those not seen in the training/validation sets, while "seen" samples are the opposite.</p>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
<!-- change introduction according to selected -->
<script>
$(document).ready(function(){
  $("#dataset-select").change(function() {
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
  });
});
</script>

#### Browse
Number of rows: 
<select name="preview-numrow" id="numrow-select">
  <option value="10" selected>10</option>
  <option value="50">50</option>
  <option value="100">100</option>
</select><br>
Columns to show:  
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
<form id="dscolumns-select">
<div class="divcheckbox"><label for="columns-select-box-1" class="labelcheckbox"><input type="checkbox" id="columns-select-box-1" checked />ab_idx</label></div>
<div class="divcheckbox"><label for="columns-select-box-2" class="labelcheckbox"><input type="checkbox" id="columns-select-box-2" checked />target</label></div>
<div class="divcheckbox"><label for="columns-select-box-3" class="labelcheckbox"><input type="checkbox" id="columns-select-box-3" checked />evidence</label></div>
<div class="divcheckbox"><label for="columns-select-box-4" class="labelcheckbox"><input type="checkbox" id="columns-select-box-4" checked />binding</label></div>
<div class="divcheckbox"><label for="columns-select-box-5" class="labelcheckbox"><input type="checkbox" id="columns-select-box-5" checked />KD</label></div>
<div class="divcheckbox"><label for="columns-select-box-6" class="labelcheckbox"><input type="checkbox" id="columns-select-box-6" checked />source</label></div>
<div class="divcheckbox"><label for="columns-select-box-7" class="labelcheckbox"><input type="checkbox" id="columns-select-box-7" checked />Hseq</label></div>
<div class="divcheckbox"><label for="columns-select-box-8" class="labelcheckbox"><input type="checkbox" id="columns-select-box-8" checked />Lseq</label></div>
<div class="divcheckbox"><label for="columns-select-box-9" class="labelcheckbox"><input type="checkbox" id="columns-select-box-9" checked />Hregion</label></div>
<div class="divcheckbox"><label for="columns-select-box-10" class="labelcheckbox"><input type="checkbox" id="columns-select-box-10" checked />Lregion</label></div>
<div class="divcheckbox"><label for="columns-select-box-11" class="labelcheckbox"><input type="checkbox" id="columns-select-box-11" checked />rbd_seq</label></div>
<div class="divcheckbox"><label for="columns-select-box-12" class="labelcheckbox"><input type="checkbox" id="columns-select-box-12" checked />matched_lineage</label></div>
<div class="divcheckbox"><label for="columns-select-box-13" class="labelcheckbox"><input type="checkbox" id="columns-select-box-13" checked />ds</label></div>
<div class="divcheckbox"><label for="columns-select-box-14" class="labelcheckbox"><input type="checkbox" id="columns-select-box-14" checked />usage</label></div>
<div class="divcheckbox"><label for="columns-select-box-15" class="labelcheckbox"><input type="checkbox" id="columns-select-box-15" checked />confident</label></div>
</form>
<p id="loading-para" class="text-right"><a href="#browse" class="btn btn--primary" id="preview-button">Apply</a></p>
<style>
td {
  white-space: nowrap;
}
</style>
<!-- show table -->
<table id="table-browse">
<!-- add table header -->
<thead id="table-browse-header"><tr>
{% for cell in site.data.datasets.classification_variantrbd[0] %}
  <th>{{ cell[0] }}</th>
{% endfor %}
</tr></thead>
<!-- add table contents -->
<tbody id="table-browse-body">
{% for row in site.data.datasets.classification_variantrbd limit:10 %}
  <tr>
  {% for cell in row %}
    <td>{{ cell[1] }}</td>
  {% endfor %}
  </tr>
{% endfor %}
</tbody>
</table>{: .full}
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
}
function GetSelectedColumns() {
  var checkedcolumns = [];
  for (checkedinput of document.querySelectorAll("#dscolumns-select input[type='checkbox']:checked")) {
    var checkboxlabel = checkedinput.parentElement;
    checkedcolumns.push(checkboxlabel.innerText);
  };
  return checkedcolumns
}
function ShowTable(checkedcolumns) {
  var datasetname = $("#dataset-select").val();
  var filebasename = GetDsBasename(datasetname);
  $("#preview-button").text("Loading...");
  $.get("../_data/datasets/" + filebasename + ".tsv", function(data) {
    // replace tab with comma
    data = data.replace(/\t/g, ",");
    var parsed = $.csv.toObjects(data);
    var numrow = $("#numrow-select").val();
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
    for (var i = 0; i < numrow; i++) {
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
    }
  }, "text")
  .done(function() {
    $("#preview-button").text("Apply");
  })
}
$(document).ready(function(){
  $("#dataset-select").change(function() {
    var datasetname = $("#dataset-select").val();
    var filebasename = GetDsBasename(datasetname);
    $.get("../_data/datasets/" + filebasename + ".tsv", function(data) {
      data = data.replace(/\t/g, ",");
      var parsed = $.csv.toObjects(data);
      $("#dscolumns-select").html("");
      var checkbox_idx = 1;
      $.each(parsed[0], function(key, value) {
        $("#dscolumns-select").append("<div class=\"divcheckbox\"><label for=\"columns-select-box-"+checkbox_idx+"\" class=\"labelcheckbox\"><input type=\"checkbox\" id=\"columns-select-box-"+checkbox_idx+"\" checked />" + key + "</label></div>");
        checkbox_idx += 1;
      });
    }, "text")
  });
  $("#dataset-select").change(function() {
      ShowTable();
  });
  $("#dscolumns-select").ready(function() {
    $("#preview-button").click(function() {
      var checkedcolumns = GetSelectedColumns();
      ShowTable(checkedcolumns);
    });
  })
});

</script>
