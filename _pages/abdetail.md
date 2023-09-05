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
<style>
th {
    font-size: 1.4em
}
td {
    font-size: 1.2em; 
    word-break: break-all;
}
div.structure {
    width: 800;
    height: 400px;
    position:relative;
}
td.clickable {
    cursor: pointer;
}
i.clickable {
    cursor: pointer;
}
summary {
    display: inline;
    cursor: pointer;
}
summary h2 {
    display: inline;
    font-size: 1.1em;
}
summary::-webkit-details-marker {
    display: none;
}
details[open] summary * i[class="fas fa-angle-right"] {
    transform: rotate(90deg);
}
</style>
<!-- <link rel="stylesheet" type="text/css" href="https://www.ebi.ac.uk/pdbe/pdb-component-library/css/pdbe-molstar-3.1.0.css"> -->
<link rel="stylesheet" type="text/css" href="https://www.ebi.ac.uk/pdbe/pdb-component-library/css/pdbe-molstar-light-3.1.0.css">
<script type="text/javascript" src="https://www.ebi.ac.uk/pdbe/pdb-component-library/js/pdbe-molstar-plugin-3.1.0.js"></script>
<script type="text/javascript" src="https://raw.githubusercontent.com/douglascrockford/JSON-js/master/json2.js"></script>
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
<script>
function GetQueryString(name) {
    var reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)");
    var r = window.location.search.substr(1).match(reg);
    if(r != null) return decodeURI(r[2]);
    return null;
};
</script>
<!-- show collected info -->
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
                ShowData(subset, detailid, showcolname, onlyfirst=onlyfirst);
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
            $("#"+detailparaid).html("<ul></ul>");
            $.each(subset, function(key, value) {
                $("#"+detailparaid+" ul").append("<li>" + value[colname] + "</li>"); 
            });
        } else {
            $("#"+detailparaid).html(subset[0][colname]);
        };
    } else {
        $("#"+detailparaid).html("<table><thead><tr></tr></thead><tbody></tbody></table>");
        $.each(colname, function(key, value) {
            $("#"+detailparaid+" thead tr").append("<th>" + value + "</th>");
        });
        $.each(subset, function(rowinidx, row) {
            var row_contents = "<tr style='height: 15px'>";
            $.each(colname, function(key, value) {
                row_contents +="<td column='" + value + "'>" + row[value] + "</td>";
            });
            row_contents += "</tr>";
            $("#"+detailparaid+" tbody").append(row_contents);
        });
    };
};
function DOI_callback() { $.each($("td[column='DOI']"), function(idx, td) { $(td).html("<a href='https://doi.org/" + $(td).text() + "'>" + $(td).text() + "</a>")}) };
function binding_callback() {
    $.each($("td[column='binding']"), function(idx, td) {
        if ($(td).text() === "1") {
            var bool_binding = "Yes";
        } else if ($(td).text() === "0") {
            var bool_binding = "No";
        };
        $(td).html(bool_binding);
    });
};
function seq_callback(selector="td[column='Hseq']", style="overflow-y: scroll; overflow-x: hidden; max-height: 100px", linelength=-1) {
    $.each($(selector), function(idx, td) {
        var seq = $(td).text();
        if (linelength > 0) {
            var num_lines = Math.ceil(seq.length / linelength);
            $(td).html("");
            for (var i = 0; i < num_lines; i++) {
                var styleddiv = "<div id='fvseq-line-"+i+"' style='"+style+"' datatype='seq'>";
                var subseq = seq.substring(i * linelength, (i + 1) * linelength);
                styleddiv = styleddiv + subseq + "</div>";
                $(td).append(styleddiv);
            };
            $(td).append("<div id='temp-region' hidden></div>");
            LoadData("../_data/tables/region.csv", "temp-region", function() {}, function(value) { return value["seq"]===seq }, "region", false, onlyfirst=true);
            var region = $("#temp-region").text().replace(/\s+/g, "");
            for (var i = 0; i < num_lines; i++) {
                var subregion = region.substring(i * linelength, (i + 1) * linelength);
                var styledregiondiv = "<div id='fvregion-line-"+i+"' style='"+style+"' datatype='region'>"+subregion+"</div>";
                $(td).find("#fvseq-line-"+i).after(styledregiondiv);
            };
            $("#temp-region").remove();
        } else {
            var styleddiv = "<div style='"+style+"'>";
            $(td).html(styleddiv + seq + "</div>");
        }
    });
};
function copy_fvseq(tocopy_id) {
    var tocopytext = "";
    $.each($("#"+tocopy_id+" div[datatype='seq']"), function(idx, div) {
        var partseq = $(div).text();
        tocopytext += partseq;
        navigator.clipboard.writeText(tocopytext);
    });
};
function db_callback(selector) {
    $.each($(selector+" td[column='source']"), function(idx, td) {
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
        } else if ($(td).text() === "sup.") {
            var hsource_id = $(td).parent().find("td[column='Hsource_id']");
            var lsource_id = $(td).parent().find("td[column='Lsource_id']");
            hsource_id.html("<a href='https://doi.org/" + hsource_id.text() + "'>" + hsource_id.text() + "</a>");
            lsource_id.html("<a href='https://doi.org/" + lsource_id.text() + "'>" + lsource_id.text() + "</a>");
        // } else if ($(td).text() === "INN") {
        //     var hsource_id = $(td).parent().find("td[column='Hsource_id']");
        //     var lsource_id = $(td).parent().find("td[column='Lsource_id']");
        } else {
        };
        $(td).html(formatted_db);
    });
};
function pdb_callback(selector, columnname="HC_instance_id") {
    $.each($(selector+" td[column='" + columnname + "']"), function(idx, td) {
        var pdbcode = $(td).text().split(".")[0];
        var formatted_pdb = "<a href='https://www.rcsb.org/3d-view/" + pdbcode + "'>" + $(td).text() + "</a>";
        $(td).html(formatted_pdb);
    });
};
function visualize_epitope(epitope_viewerInstance, epitopes_selections) {
    epitope_viewerInstance.visual.reset({ camera: false, theme: true });
//    epitope_viewerInstance.visual.focus([{start_residue_number: 319, end_residue_number: 541}]);
    epitope_viewerInstance.visual.select({data:[{start_residue_number: 319, end_residue_number: 541, focus: true, sideChain: false}], nonSelectedColor:{r:255, g:255, b:255}});
//    epitope_viewerInstance.visual.clearSelection();
    epitope_viewerInstance.visual.select(epitopes_selections);
    epitope_viewerInstance.visual.highlight(epitopes_selections);
};
function epitope_callback(toshow_id) {
    $.ajax({
        url: "../_data/tables/epitope_group.csv",
        type: "GET",
        async: false,
        dataType: "text",
        success: function(data) {
            var parsed = $.csv.toObjects(data);
            $("#"+toshow_id+" thead tr").append("<th>epitope</th>");
            $.each($("#"+toshow_id+" tbody td[column='HC_instance_id']"), function(key, td) {
                var hc_instance_id = $(td).text();
                var epitopes = "";
                $.each(parsed, function(key, value) {
                    if (value["HC_instance_id"] === hc_instance_id) {
                        epitopes = value["epitope"];
                    };
                });
                var epitopes_selections = [];
                $.each(epitopes.split(";"), function(key, value) {
                    epitopes_selections.push({
                        color:{r:0, g:0, b:255},
                        residue_number: parseInt(value.substring(0, value.length-1)),
                        sideChain: false,
                        focus: false,
                    });
                });
                var serialized_epitopes_selections = "{data:" + JSON.stringify(epitopes_selections) + "}";
                $(td).parent().append("<td class='clickable' onclick='visualize_epitope(epitope_viewerInstance, " + serialized_epitopes_selections + ")'>"+epitopes+"</td>");
            })
        }
    });
};
function abtype_callback(toshow_id) {
    $.ajax({
        url: "../_data/tables/ab_type.csv",
        type: "GET",
        async: true,
        dataType: "text",
        success: function(data) {
            var parsed = $.csv.toObjects(data);
            $("#"+toshow_id+" thead tr").append("<th>form</th>");
            $.each($("#"+toshow_id+" tbody tr"), function(key, tr) {
                var hseq = $(tr).find("td[column='Hseq']").text();
                var lseq = $(tr).find("td[column='Lseq']").text();
                var form = "";
                $.each(parsed, function(key, value) {
                    if ((value["Hseq"] === hseq) && (value["Lseq"] === lseq)) {
                        form = value["ab_type"];
                    };
                });
                $(tr).append("<td>"+form+"</td>");
            })
        }
    });
}
$(document).ready(function(){
    // names
    var ab_idx = GetQueryString("ab_idx");
    LoadData("../_data/tables/name.csv", "detail-all_names", function() {}, function(value) { return ((value["all_names"]!=="") && (value["ab_idx"]===ab_idx)) }, "all_names");
    // evidence
    var exp_evidence_colnames = ["DOI", "target", "binding", "target_type", "Ab_type", "evidence", "KD", "source"];
    LoadData("../_data/tables/evidence.csv", "detail-exp_evidence", function() { DOI_callback(); binding_callback(); }, function(value) { return (value["DOI"]!=="") && (value["ab_idx"]===ab_idx) }, exp_evidence_colnames);
    var struct_evidence_colnames = ["source", "target", "binding", "target_type", "Ab_type"];
    LoadData("../_data/tables/evidence.csv", "detail-struct_evidence", binding_callback, function(value) { return( value["pdb"]!=="") && (value["ab_idx"]===ab_idx) }, struct_evidence_colnames, false);
    // record
    var pdb_record_colnames = ["Hseq", "Lseq", "source", "Hsource_id", "Lsource_id"];
    LoadData("../_data/tables/record.csv", "detail-pdb_record", function() { db_callback("#detail-pdb_record"); seq_callback("td[column='Hseq']"); seq_callback("td[column='Lseq']"); abtype_callback("detail-pdb_record");}, function(value) {return (["pdb","genbank","CoVAbDab","INN"].includes(value["source"])) && (value["ab_idx"]===ab_idx) }, pdb_record_colnames, false);
    var sup_record_colnames = ["Hseq", "Lseq", "source", "Hsource_id", "Lsource_id"];
    LoadData("../_data/tables/record.csv", "detail-sup_record", function() { db_callback("#detail-sup_record"); seq_callback("td[column='Hseq']"); seq_callback("td[column='Lseq']"); abtype_callback("detail-sup_record");}, function(value) {return (["sup.","patent"].includes(value["source"])) && (value["ab_idx"]===ab_idx) }, sup_record_colnames, false);
    LoadData("../_data/tables/record.csv", "detail-transform_record", function() { seq_callback("td[column='Hseq']"); seq_callback("td[column='Lseq']"); abtype_callback("detail-transform_record");}, function(value) {return (["combination","mutation","split"].includes(value["source"])) && (value["ab_idx"]===ab_idx) }, sup_record_colnames, false);
    // vgene
    LoadData("../_data/tables/vgene.csv", "detail-vhgene", function() {}, function(value) { return (value["chain"]==="H") && (value["ab_idx"]===ab_idx) }, "V gene");
    LoadData("../_data/tables/vgene.csv", "detail-vlgene", function() {}, function(value) { return (value["chain"]==="L") && (value["ab_idx"]===ab_idx) }, "V gene");
    // vseq & numbering
    $("div[dbsource='record']").ready(function() {
        var hseq_cols = $("td[column='Hseq']");
        var hseqs = [];
        $.each(hseq_cols, function(idx, td) {
            hseqs.push($(td).text());
        });
        LoadData("../_data/tables/trunct2fv.csv", "detail-hvseq", function() { seq_callback("#detail-hvseq","font-family: monospace;",linelength=80); }, function(value) { return hseqs.includes(value["seq"]) }, "seq_vdomain", false, onlyfirst=true);
        var lseq_cols = $("td[column='Lseq']");
        var lseqs = [];
        $.each(lseq_cols, function(idx, td) {
            lseqs.push($(td).text());
        });
        LoadData("../_data/tables/trunct2fv.csv", "detail-lvseq", function() { seq_callback("#detail-lvseq","font-family: monospace;",linelength=80);}, function(value) { return lseqs.includes(value["seq"]) }, "seq_vdomain", false, onlyfirst=true);
    });
    // epitope
    $("div[dbsource='evidence']").ready(function() {
        $.ajax({
            url: "../_data/tables/pdb_chain_idmapping.csv",
            type: "GET",
            async: false,
            dataType: "text",
            success: function(data) {
                var parsed = $.csv.toObjects(data);
                var pdb_instances = [];
                for (var td of $("#detail-struct_evidence td[column='source']")) {
                    var pdb_entities = $(td).text().split(";");
                    if (pdb_entities.length === 2) {
                        var pairing_filename = "pdb_ab_rbd_pairing.csv";
                        $("#detail-pdb_nb").hide();
                        var toshow_id = "detail-pdb_ab";
                        var toshow_colnames = ["HC_instance_id","LC_instance_id","spike_instance_id"];
                    } else if (pdb_entities.length === 1) {
                        var pairing_filename = "pdb_nb_rbd_pairing.csv";
                        $("#detail-pdb_ab").hide();
                        var toshow_id = "detail-pdb_nb";
                        var toshow_colnames = ["HC_instance_id","spike_instance_id"];
                    } else {
                    };
                    var subset = [];
                    $.each(parsed, function(key, value) {
                        if (pdb_entities.includes(value["entity"])) { subset.push(value); };
                    });
                    var sorted = subset.sort(function(a, b) {
                        return a["chain"].localeCompare(b["chain"]);
                    });``
                    $.each(sorted, function(key, value) {
                        pdb_instances.push(value["instance"]);
                    })
                };
                LoadData("../_data/tables/"+pairing_filename, toshow_id, function() {pdb_callback("#"+toshow_id, "HC_instance_id"); pdb_callback("#"+toshow_id, "LC_instance_id"); pdb_callback("#"+toshow_id, "spike_instance_id"); epitope_callback(toshow_id);}, function(value) { return pdb_instances.includes(value["HC_instance_id"]) }, toshow_colnames, false);
            },
        });
    });
});

</script>

<h1><span><em>names</em> of this antibody: </span><span id="detail-all_names">Loading...</span></h1>
<h1 id="header-binding-evidence">Binding evidence</h1>
<details open><summary>
<h2 id="header-detail-exp_evidence">Experimental evidence  <i class="fas fa-angle-right"></i></h2></summary>
<div class="notice" id="detail-exp_evidence" dbsource="evidence">Loading...</div></details>
<details open><summary>
<h2 id="header-detail-struct_evidence">Structural evidence  <i class="fas fa-angle-right"></i></h2></summary>
<div class="notice" id="detail-struct_evidence" dbsource="evidence">Loading...</div></details>
<h1 id="header-sequences">Sequences</h1>
<details open><summary>
<h2 id="header-collected-sequences">Collected sequences  <i class="fas fa-angle-right"></i></h2></summary>
<ul>
<li id="header-detail-pdb_record">from databases: PDB/GenBank/CovAb-Dab/INN
<div class="notice" id="detail-pdb_record" dbsource="record">Loading...</div></li>
<li id="header-detail-sup_record">from articles/supplemental files/patents
<div class="notice" id="detail-sup_record" dbsource="record">Loading...</div></li>
<li id="header-detail-transform_record">from combination/mutation/split of other antibody chain sequences
<div class="notice" id="detail-transform_record" dbsource="record">Loading...</div></li>
</ul>
</details>
<details open><summary>
<h2 id="header-fv-region">Predicted variable domain sequences & split of CDR/FR  <i class="fas fa-angle-right"></i></h2></summary>
<ul>
<li id="header-detail-hvseq">Heavy Chain <i class="fas fa-copy clickable" onclick="copy_fvseq('detail-hvseq')"></i>
<div class="notice" id="detail-hvseq" dbsource="trunct2fv">>Loading...</div></li>
<li id="header-detail-lvseq">Light Chain <i class="fas fa-copy clickable" onclick="copy_fvseq('detail-lvseq')"></i>
<div class="notice" id="detail-lvseq" dbsource="trunct2fv">Loading...</div></li>
</ul>
</details>
<details open><summary>
<h2 id="header-identified-vgene">Identified V gene  <i class="fas fa-angle-right"></i></h2></summary>
<p class="notice"><span><em><strong>IGHV</strong></em> gene: </span><span id="detail-vhgene" dbsource="vgene">Loading...</span><br><span><em><strong>IGLV/IGKV</strong></em> gene: </span><span id="detail-vlgene" dbsource="vgene">Loading...</span></p>
</details>
<h1 id="header-structures">Structures</h1>
<details open><summary>
<h2 id="header-epitope">Epitopes  <i class="fas fa-angle-right"></i></h2></summary>
<div class="notice" id="detail-pdb_ab" dbsource="pdb_chain_idmapping">Loading...</div>
<div class="notice" id="detail-pdb_nb" dbsource="pdb_chain_idmapping">Loading...</div>
<div id="visualize-epitope" class="structure" hidden></div>
</details>
<details open><summary>
<h2 id="header-igfold">Predicted apo-form structures <i class="fas fa-angle-right"></i></h2></summary>
<div id="loading-visualize-igfold">Loading...</div>
<div id="visualize-igfold" class="structure" hidden></div>
</details>

<script>
$("#visualize-epitope").show();
var epitope_viewerInstance = new PDBeMolstarPlugin();
var options = {
    customData: { url: 'https://www.ebi.ac.uk/pdbe/model-server/v1/7wz2/atoms?label_entity_id=1&auth_asym_id=A&encoding=bcif', format: 'cif', binary:true },
    hideControls: true,
    landscape: true,
    pdbeLink: true,
    sequencePanel: true,
    bgColor: {r:255, g:255, b:255},
}
var viewerContainer = document.getElementById('visualize-epitope');
epitope_viewerInstance.render(viewerContainer, options);
epitope_viewerInstance.events.loadComplete.subscribe(function() {
//    epitope_viewerInstance.visual.focus([{start_residue_number: 319, end_residue_number: 541}]);
    epitope_viewerInstance.visual.select({data:[{start_residue_number: 319, end_residue_number: 541, focus: true, sideChain: false}], nonSelectedColor:{r:255, g:255, b:255}});
//    epitope_viewerInstance.visual.clearSelection();
});
// visualize igfold structures
var ab_idx = GetQueryString("ab_idx");
var structurefilepath = "../assets/igfold/"+ab_idx+".pdb";
$.ajax({
    url: structurefilepath,
    type: "HEAD",
    error: function() {
        $("#loading-visualize-igfold").text("Structures not available.");
    },
    success: function() {
        $("#loading-visualize-igfold").text("");
        $("#visualize-igfold").show();
        var viewerInstance = new PDBeMolstarPlugin();
        var options = {
            customData: { url: structurefilepath, format: 'pdb'},
            hideControls: true,
            landscape: true,
            pdbeLink: false,
            sequencePanel: true,
            bgColor: {r:255, g:255, b:255},
        }
        var viewerContainer = document.getElementById('visualize-igfold');
        viewerInstance.render(viewerContainer, options);
    }
})

</script>