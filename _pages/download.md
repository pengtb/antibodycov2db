---
permalink: /download/
title: "Download"
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
h2 {
    font-size: 1.1em;
}
td {
  white-space: nowrap;
}
</style>

<!-- <script src="../assets/js/main.min.js"></script> -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>

<!-- change introduction according to selected -->
<script>
function UpdateTableIntro() {
  var table2dl = $("#table-select").val();
  if (table2dl === "name") {
    $("#dbtable-intro").html("The table <em><strong>name</strong></em> consists of two columns: <em>names</em> of antibodies and built-in <em>unique IDs</em>.");
  } else if (table2dl === "evidence") {
    $("#dbtable-intro").html("The table <em><strong>evidence</strong></em> contains binding information of antiobdies. Each entry of the table corresponds to one piece of evidence, including the <em>source</em> (such as an article, a PDB entry or a patent file etc.), the binary or quantitative <em>binding affinity</em>, the <em>target</em> that the antibody binds to (usually RBD), and the <em>unique ID</em> of the antibody.");
  } else if (table2dl === "sequence") {
    $("#dbtable-intro").html("The table <em><strong>sequence</strong></em> simply includes all the sequences of antibodies mentioned in the database.");
  } else if (table2dl === "record") {
    $("#dbtable-intro").html("The table <em><strong>record</strong></em> includes all collected records of antiobdy sequences. Speicifically it contains the <em>sources</em> of records (such as suppl. files of articles, database entries or patents files etc.), <em>sequences</em> of heavy/light chains provided by these records and corresponding <em>unique IDs</em> of antibodies.");
  } else if (table2dl === "region") {
    $("#dbtable-intro").html("The table <em><strong>region</strong></em> contains the information of detailed regions of antibodies. The variable domain of the heavy/light chain of an antibody consists of CDR (complementarity-determining region) and FR (framework region). Each residue was numbered following the IMGT scheme using <a href=\"https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/sabpred/anarci/\">ANARCI</a>, then it was assigned to a specific region according the <a href=\"https://www.imgt.org/IMGTScientificChart/Numbering/IMGTcorrespondence.html\">correspondence</a>. Numbers from 1 to 7 represent CDR1, FR1, CDR2, FR2, CDR3, FR3 and CDR4 respectively, and 0 represents that the residue is not numbered. Therefore the region information of a sequence is represented as an array of numbers, which is stored as a string seperated with space. The table thus consists of two columns: the <em>sequence</em> and its <em>region</em> string.");
  } else if (table2dl === "num_domain") {
    $("#dbtable-intro").html("The table <em><strong>num_domain</strong></em> indicates number of variabl/constant domains in a heavy/light chain sequence. A full-length heavy chain sequence includes one variable domain and three constant domains, while a full-length light chain sequence includes one variable domain and one constant domain, as shown in an illustration from <a href=\"https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/sabdab\">SAbDab</a><br><img src=\"https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/static/img/antibody_schematic.png\"><br>However, the form of our collected antibodies can be IgG (the full-length form), Fab, Fv or even scFv. Domain of each sequence was identified using <a href=\"https://www.ebi.ac.uk/interpro/about/interproscan\">InterProScan</a>. The table thus consists of two columns: the <em>sequence</em> and its <em>number of domains</em>.");
  } else if (table2dl === "ab_type") {
    $("#dbtable-intro").html("The table <em><strong>ab_type</strong></em> indicates the form of an antibody. Number of varible/constant domains in a sequence was specified using <a href=\"https://www.ebi.ac.uk/interpro/about/interproscan\">InterProScan</a> and then was used to determine the form of the antibody. The form of antibodies can be IgG, Fab and Fv, as shown in an illustration from <a href=\"https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/sabdab\">SAbDab</a><br><img src=\"https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/static/img/antibody_schematic.png\"><br>The table thus consists of two columns: the <em>sequence</em> and its <em>form</em>.");
  } else if (table2dl === "trunct2fv") {
    $("#dbtable-intro").html("The table <em><strong>trunct2fv</strong></em> indicates the variable domain of a heavy/light chain antiobdy sequence. The variable domain was identified using <a href=\"https://www.ebi.ac.uk/interpro/about/interproscan\">InterProScan</a>. The identification involves analysis CDD, Pfam and SUPERFAMILY. The table thus consists of two columns: the <em>sequence</em> and corresponding extracted <em>variable domain sequence</em>.");
  } else if (table2dl === "pdb_chain_idmapping") {
    $("#dbtable-intro").html("The table <em><strong>pdb_chain_idmapping</strong></em> contains mappings between instance ids and entity ids in <a href=\"https://www.rcsb.org/\">PDB</a>. The mappings were retrieved from PDB data API.");
  } else if (table2dl === "pdb_ab_rbd_pairing") {
    $("#dbtable-intro").html("The table <em><strong>pdb_ab_rbd_pairing</strong></em> provides the pairing between PDB heavy chain instance id, light chain instance id and SARS-CoV2 spike/RBD instance id for each antibody-RBD complex. Most of these were retrieved from <a href=\"https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/sabdab\">SAbDab</a>, and all were manually checked and curated.");
  } else if (table2dl === "pdb_nb_rbd_pairing") {
    $("#dbtable-intro").html("The table <em><strong>pdb_nb_rbd_pairing</strong></em> provides the pairing between PDB heavy chain instance id and SARS-CoV2 spike/RBD instance id for each nanobody-RBD complex. Most of these were retrieved from <a href=\"https://opig.stats.ox.ac.uk/webapps/sabdab-sabpred/sabdab\">SAbDab</a>, and all were manually checked and curated.");
  } else if (table2dl === "epitope_group") {
    $("#dbtable-intro").html("The table <em><strong>epitope_group</strong></em> provides the epitope group of each antibody-RBD complex. With key residues of each epitope group from <a href=\"https://www.nature.com/articles/s41586-022-04980-y\">Cao's work</a>, each complex (or pair of instance ids) was assigned to one epitope group. All assignments were manually checked and curated as well.)");
  } else if (table2dl === "target_rbdseq") {
    $("#dbtable-intro").html("The table <em><strong>target_rbdseq</strong></em> provides sequence of each variant (or WT) of targeting RBD. Mutations of each variant were retrieved from <a href=\"https://outbreak.info/\">outbreak.info</a> using its API with the frequency threshold set to 0.75. The sequence of each mutant was then generated. The table consists of two columns: the <em>target RBD</em> and its <em>sequence</em>.");
  } else if (table2dl === "vgene") {
    $("#dbtable-intro").html("The table <em><strong>vgene</strong></em> provides the IGHV gene of every heavy chain protein sequence. The gene was identified using <a href=\"https://www.ncbi.nlm.nih.gov/igblast/\">IgBlast</a>. We searched the sequence against IMGT germline database which contains all human V genes. The table consists of two columns: the antibody <em>unique IDs</em> and its <em>IGHV gene</em>");
  } else {
    $("#dbtable-intro").text("");
  }
}
$(document).ready(function(){
  UpdateTableIntro();
  $("#table-select").change(UpdateTableIntro);
});
</script>
<!-- change dllink according to selected -->
<script>
$(document).ready(function(){
  $("#table-select").click(function() {
    var table2dl = $("#table-select").val();
    $("#dbtable-dllink").attr("href", "../_data/tables/" + table2dl + ".csv");
    $("#dbtable-dllink").attr("download", table2dl + ".csv");
  });
});
</script>

<script>
function ShowTable() {
  var tablename = $("#table-select").val();
  $("#preview-button").text("Loading...");
  $.get("../_data/tables/" + tablename + ".csv", function(data) {
    var parsed = $.csv.toObjects(data);
    var numrow = $("#numrow-select").val();
    $("#table-preview-header").html("");
    $("#table-preview-header").append("<tr>");
    $.each(parsed[0], function(key, value) {
      $("#table-preview-header").append("<th>" + key + "</th>");
    });
    $("#table-preview-header").append("</tr>");
    $("#table-preview-body").html("");
    for (var i = 0; i < numrow; i++) {
      $("#table-preview-body").append("<tr>");
      $.each(parsed[i], function(key, value) {
        $("#table-preview-body").append("<td>" + value + "</td>");
      });
      $("#table-preview-body").append("</tr>");
    }
  }, "text")
  .done(function() {
    $("#preview-button").text("Apply");
  })
}
$(document).ready(function(){
  ShowTable();
  $("#preview-button").click(ShowTable);
  $("#table-select").change(ShowTable);
});
</script>

# One click for all {#download-all}
Just click <a href="../compressed/all_db_tables.7z" download="all_db_tables.7z">here</a> to download all the tables in the database and <a href="../compressed/all_ds_tables.7z" download="all_ds_tables.7z">here</a> for all the processed datasets.

# Or select one table to preview and download {#download-one}
Please choose a table firstly: 
<select name="table2dl" id="table-select">
  <option value="name" selected>name</option>
  <option value="evidence">evidence</option>
  <option value="sequence">sequence</option>
  <option value="record">record</option>
  <option value="region">region</option>
  <option value="num_domain">num_domain</option>
  <option value="ab_type">ab_type</option>
  <option value="trunct2fv">trunct2fv</option>
  <option value="pdb_chain_idmapping">pdb_chain_idmapping</option>
  <option value="pdb_ab_rbd_pairing">pdb_ab_rbd_pairing</option>
  <option value="pdb_nb_rbd_pairing">pdb_nb_rbd_pairing</option>
  <option value="epitope_group">epitope_group</option>
  <option value="target_rbdseq">target_rbdseq</option>
  <option value="vgene">vgene</option>
</select>
<a id="dbtable-dllink" href="../_data/tables/name.csv" download="name.csv" class="btn btn--primary">Download</a>
<img src="../assets/images/dbmodel_20230912.png" width="100%">
<h2 id="header-introduction">Introduction</h2>
<p id="dbtable-intro"></p>

<h2 id="header-preview">Preview</h2>
<p><span>Number of rows to show: </span>
<select name="preview-numrow" id="numrow-select">
  <option selected>10</option>
  <option >20</option>
  <option >50</option>
</select>
<span><a href="#preview" class="btn btn--primary" id="preview-button">Apply</a></span></p>
<p class="text-center"><table id="table-preview">
<thead id="table-preview-header"></thead>
<tbody id="table-preview-body"></tbody>
</table></p>
