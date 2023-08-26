---
permalink: /download/
title: "Download"
header: 
    overlay_color: "#333"
    # actions:
    #  - label: "Download All"
    #    url: "https://github.com"
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
# One click for all
Just click <a href="../compressed/all_db_tables.7z" download="all_db_tables.7z">here</a> to download all the tables in the database and <a href="../compressed/all_ds_tables.7z" download="all_ds_tables.7z">here</a> for all the processed datasets.

# Or Select one table to preview and download
Please choose a table firstly:  
<select name="table2dl" id="table-select">
  <optgroup label="Antibody Binding Evidence">
    <option value="name" selected>name</option>
    <option value="evidence">evidence</option>
  </optgroup>
  <optgroup label="Antibody Sequence Properties">
    <option value="sequence">sequence</option>
    <option value="record">record</option>
    <option value="region">region</option>
    <option value="num_domain">num_domain</option>
    <option value="ab_type">ab_type</option>
    <option value="trunct2fv">trunct2fv</option>
  </optgroup>
  <optgroup label="Antibody Epitopes">
    <option value="pdb_chain_idmapping">pdb_chain_idmapping</option>
    <option value="pdb_ab_rbd_pairing">pdb_ab_rbd_pairing</option>
    <option value="pdb_nb_rbd_pairing">pdb_nb_rbd_pairing</option>
    <option value="epitope_group">epitope_group</option>
    <option value="target_rbdseq">target_rbdseq</option>
  </optgroup>
  <optgroup label="Others">
    <option value="vgene">vgene</option>
  </optgroup>
</select>
<p id="dbtable-intro">Some introduction</p>
<script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.8.0.js">
  // function showIntro() {
  //   var table2dl = $("#table-select").val();
  //   var intro = $("#dbtable-intro").textContent;
  //   if (table2dl === "evidence") {
  //     // intro.textContent = "The table name containts names of antibodies and their unique IDs in the database.";
  //     intro.append("The table name containts names of antibodies and their unique IDs in the database.");
  //   } else {
  //     intro.textContent = "";
  //   }
  // };
  // $(#table-select).change(showIntro);
  $(#dbtable-intro).click(function(){$(#dbtable-intro).hide();});
 
</script>