---
permalink: /abdetail/
title: "Details"
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
function LoadData(filepath, detailid, donecallback, subsetfunction, showcolname, async=true, onlyfirst=false) {
    $.ajax({
        url: filepath,
        type: "GET",
        async: async,
        dataType: "text",
        success: function(data) {
            var parsed = $.csv.toObjects(data);
            var subset = [];
            $.each(parsed, function(key, value) {
                if (subsetfunction(value)) {
                    subset.push(value);
                };
            });
            if (subset.length > 0) {
                ShowData(subset, detailid, showcolname, onlyfirst);
            } else {
                $("#header-"+detailid).hide();
                $("#"+detailid).hide();
            };
        },
        error: function() {
            alert("Error in loading data");
            $("#header-"+detailid).hide();
            $("#"+detailid).hide();
        }
    }).done(donecallback);
};
function ShowData(subset, detailparaid, colname, onlyfirst=false) {
    if (typeof colname === "string") {
        if ((subset.length > 1) && (!onlyfirst)) {
            $("#"+detailparaid).html("<ul>");
            $.each(subset, function(key, value) {
                $("#"+detailparaid).append("<li>" + value[colname] + "</li>"); 
            });
            $("#"+detailparaid).append("</ul>");
        } else {
            $("#"+detailparaid).html(subset[0][colname]);
        };
    } else {
        $("#"+detailparaid).html("<table><thead><tr>");
        $.each(colname, function(key, value) {
            $("#"+detailparaid).append("<th>" + value + "</th>");
        });
        $("#"+detailparaid).append("</tr></thead><tbody>");
        $.each(subset, function(rowinidx, row) {
            $("#"+detailparaid).append("<tr style='height: 15px'>");
            $.each(colname, function(key, value) {
                $("#"+detailparaid).append("<td style='word-break: break-all' column='" + value + "'>" + row[value] + "</td>");
            })
            $("#"+detailparaid).append("</tr>");
        });
        $("#"+detailparaid).append("</tbody></table>");
    };
};
function DOI_callback() { $.each($("td[column='DOI']"), function(idx, td) { $(td).html("<a href='https://doi.org/" + $(td).text() + "'>" + $(td).text() + "</a>")}) };
function binding_callback() {
    $.each($("td[column='binding']"), function(idx, td) {
        if ($(td).text() === "1") {
            var bool_binding = "Yes";
        } else {
            var bool_binding = "No";
        };
        $(td).html(bool_binding);
    });
};
function seq_callback(selector="td[column='Hseq']", style="overflow-y: scroll; overflow-x: hidden; max-height: 100px", linelength=-1) {
    var styleddiv = "<div style='"+style+"'>";
    $.each($(selector), function(idx, td) {
        var seq = $(td).text();
        if (linelength > 0) {
            var num_lines = Math.ceil(seq.length / linelength);
            $(td).html("");
            for (var i = 0; i < num_lines; i++) {
                var subseq = seq.substring(i * linelength, (i + 1) * linelength);
                $(td).append(styleddiv + subseq + "</div><br>");
            }
        } else {
            $(td).html(styleddiv + seq + "</div>");
        }
    });
};
function db_callback() {
    $.each($("td[column='source']"), function(idx, td) {
        if ($(td).text() === "pdb") {
            var formatted_db = "PDB";
            var hsource_id = $(td).parent().find("td[column='Hsource_id']");
            var lsource_id = $(td).parent().find("td[column='Lsource_id']");
            var pdbcode = $(hsource_id).text().substring(0, 4);
            hsource_id.html("<a href='https://www.rcsb.org/fasta/entry/" + pdbcode + "/display'>" + hsource_id.text() + "</a>");
            lsource_id.html("<a href='https://www.rcsb.org/fasta/entry/" + pdbcode + "/display'>" + lsource_id.text() + "</a>");
        } else if ($(td).text() === "genbank") {
            var formatted_db = "GenBank";
        } else if ($(td).text() === "CoVAbDab") {
            var formatted_db = "CoVAb-Dab";
        } else {
        };
        $(td).html(formatted_db);
    });
};
$(document).ready(function(){
    // names
    var ab_idx = GetQueryString("ab_idx");
    LoadData("../_data/tables/name.csv", "detail-all_names", function() {}, function(value) { return ((value["all_names"]!=="") && (value["ab_idx"]===ab_idx)) }, "all_names");
    // evidence
    var exp_evidence_colnames = ["DOI", "target", "binding", "target_type", "Ab_type", "evidence", "KD", "source"];
    LoadData("../_data/tables/evidence.csv", "detail-exp_evidence", function() { DOI_callback(); binding_callback(); }, function(value) { return (value["DOI"]!=="") && (value["ab_idx"]===ab_idx) }, exp_evidence_colnames);
    var struct_evidence_colnames = ["pdb", "target", "binding", "target_type", "Ab_type"];
    LoadData("../_data/tables/evidence.csv", "detail-struct_evidence", binding_callback, function(value) { return( value["pdb"]!=="") && (value["ab_idx"]===ab_idx) }, struct_evidence_colnames);
    // record
    var pdb_record_colnames = ["Hseq", "Lseq", "source", "Hsource_id", "Lsource_id"];
    LoadData("../_data/tables/record.csv", "detail-pdb_record", function() { seq_callback("td[column='Hseq']"); seq_callback("td[column='Lseq']"); db_callback(); }, function(value) {return (["pdb","genbank","CoVAbDab","INN"].includes(value["source"])) && (value["ab_idx"]===ab_idx) }, pdb_record_colnames, false);
    var sup_record_colnames = ["Hseq", "Lseq", "source"];
    LoadData("../_data/tables/record.csv", "detail-sup_record", function() { seq_callback("td[column='Hseq']"); seq_callback("td[column='Lseq']");}, function(value) {return (["sup.","patent"].includes(value["source"])) && (value["ab_idx"]===ab_idx) }, sup_record_colnames, false);
    LoadData("../_data/tables/record.csv", "detail-transform_record", function() { seq_callback("td[column='Hseq']"); seq_callback("td[column='Lseq']");}, function(value) {return (["combination","mutation","split"].includes(value["source"])) && (value["ab_idx"]===ab_idx) }, sup_record_colnames, false);
    // vseq & numbering
    $("div[source='record']").ready(function() {
        var hseq_cols = $("td[column='Hseq']");
        var hseqs = [];
        $.each(hseq_cols, function(idx, td) {
            hseqs.push($(td).text());
        });
        LoadData("../_data/tables/trunct2fv.csv", "detail-hvseq", function() { seq_callback("#detail-hvseq","",linelength=80);}, function(value) { return hseqs.includes(value["seq"]) }, "seq_vdomain", onlyfirst=true);
        var lseq_cols = $("td[column='Lseq']");
        var lseqs = [];
        $.each(lseq_cols, function(idx, td) {
            lseqs.push($(td).text());
        });
        LoadData("../_data/tables/trunct2fv.csv", "detail-lvseq", function() { seq_callback("#detail-lvseq","",linelength=80);}, function(value) { return lseqs.includes(value["seq"]) }, "seq_vdomain", onlyfirst=true);
    })
    // vgene
    LoadData("../_data/tables/vgene.csv", "detail-vhgene", function() {}, function(value) { return (value["chain"]==="H") && (value["ab_idx"]===ab_idx) }, "V gene");
    LoadData("../_data/tables/vgene.csv", "detail-vlgene", function() {}, function(value) { return (value["chain"]==="L") && (value["ab_idx"]===ab_idx) }, "V gene");
});
</script>
<h1><span><em>names</em> of this antibody: </span><span id="detail-all_names">Loading...</span></h1>
# Binding evidence
<h4 id="header-detail-exp_evidence">Experimental evidence</h4>
<div class="notice" id="detail-exp_evidence" dbsource="evidence">Loading...</div>
<h4 id="header-detail-struct_evidence">Structural evidence</h4>
<div class="notice" id="detail-struct_evidence" dbsource="evidence">Loading...</div>

# Sequences
#### Collected sequences
<tl>
<li id="header-detail-pdb_record">from databases: PDB/GenBank/CovAb-Dab/INN
<div class="notice" id="detail-pdb_record" dbsource="record">Loading...</div></li>
<li id="header-detail-sup_record">from articles/supplemental files/patents
<div class="notice" id="detail-sup_record" dbsource="record">Loading...</div></li>
<li id="header-detail-transform_record">from combination/mutation/split of other antibody chain sequences
<div class="notice" id="detail-transform_record" dbsource="record">Loading...</div></li>
</tl>
#### Predicted variable domain sequences & split of CDR/FR
<tl>
<li id="header-detail-hvseq">Heavy chain
<div class="notice" id="detail-hvseq">Loading...</div></li>
<li id="header-detail-lvseq">Light Chain
<div class="notice" id="detail-lvseq">Loading...</div></li>
</tl>
#### Identified V gene
<p class="notice"><span><em><strong>IGHV</strong></em> gene: </span><span id="detail-vhgene" dbsource="vgene">Loading...</span><br><span><em><strong>IGLV/IGKV</strong></em> gene: </span><span id="detail-vlgene" dbsource="vgene">Loading...</span></p>

# Structures
#### PDB references

#### Epitopes

#### Predicted structures by IgFold