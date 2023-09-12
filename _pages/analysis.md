---
permalink: /analysis/
title: "Analysis"
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
    form[id="vgene-options"] {
        display: flex;
    }
    form[id="vgene-options"] div {
        /* text-indent: 15px; */
        padding: 5px;
        display: inline;
        /* text-align: right; */
    }
    form[id="vgene-options"] select {
        display: inline;
    }
    form[id="vgene-options"] label {
        padding: 5px;
        text-indent: 15px;
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
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
<script type="text/javascript" src="https://cdn.plot.ly/plotly-2.25.2.min.js" charset="utf-8"></script>
<script src="https://cdn.jsdelivr.net/npm/d3@7"></script>
<script type="module">
function valuecount_keys(rows, keys) {
    var pair_values = [];
    for (var row of rows) {
        var row_pair_values = [];
        for (var key of keys) {
            row_pair_values.push(row[key]);
        };
        pair_values.push(row_pair_values);
    };
    var dedup_pair_values = uniq(pair_values);
    var dedup_pair_counts = dedup_pair_values.map(function(pair) {
        return pair_values.map(function(row) {
            return JSON.stringify(row) === JSON.stringify(pair) ? 1 : 0;
        }).reduce(function(a, b) {
            return a + b
        })
    });
    var dedup_pair = dedup_pair_values.map( (row, index) => [row, dedup_pair_counts[index]] );
    var sorted_pair = dedup_pair.sort( (a, b) => b[1] - a[1] );
    dedup_pair_counts = sorted_pair.map( (row) => row[1] );
    dedup_pair_values = sorted_pair.map( (row) => row[0] );
    return [dedup_pair_values, dedup_pair_counts]
};
function uniq(a) {
    return a.sort().filter(function(item, pos, ary) {
        return !pos || JSON.stringify(item) != JSON.stringify(ary[pos - 1]);
    });
};
function filter_by_threshold(values, threshold=20, ...otherarrs) {
    var ifgood = values.map( (value) => value > threshold );
    var filtered_values = values.filter( (value, index) => ifgood[index]);
    var filtered_arrs = [filtered_values];
    for (var otherarr of otherarrs) {
        filtered_arrs.push(otherarr.filter( (value, index) => ifgood[index]));
    };
    return filtered_arrs;
}
async function plot_sunburst(tablefilepath, placeid, parentcolname, childcolname, threshold=20) {
    var rows = await d3.csv(tablefilepath);
    if (tablefilepath.includes("record")) {
        const abtype = await d3.csv("../_data/tables/ab_type.csv");
        rows = rows.map( (row) => {
            var hseq = row["Hseq"];
            var lseq = row["Lseq"];
            var matched_abtype = abtype.filter( (rec) => (rec["Hseq"]===hseq) && (rec["Lseq"]===lseq) );
            var row_abtype = matched_abtype.map( (rec) => rec["ab_type"] );
            row["ab_type"] = row_abtype;
            return row
        });
        console.log(rows);
    }
    var [dedup_pair_values, dedup_pair_counts] = valuecount_keys(rows, [childcolname, parentcolname]);
    var [dedup_parent_values, dedup_parent_counts] = valuecount_keys(rows, [parentcolname]);
    var labels = dedup_pair_values.map( (row) => row[0] ).concat(dedup_parent_values.map( (row) => row[0] ));
    var parents = dedup_pair_values.map( (row) => row[1] ).concat(dedup_parent_values.map( (row) => "" ));
    var values = dedup_pair_counts.concat(dedup_parent_counts);
    [values, labels, parents] = filter_by_threshold(values, threshold, labels, parents);
    var ids = parents.map( (row, index) => row === "" ? labels[index] : row + " - " + labels[index] );
    var data = [{
        type: "sunburst",
        // maxdepth: 2,
        ids: ids,
        labels: labels,
        parents: parents,
        values: values,
        outsidetextfont: {size: 20, color: "#377eb8"},
        leaf: {opacity: 0.4},
        marker: {line: {width: 2}},
        branchvalues: "total",
    }];
    var layout = {
        margin: {l: 0, r: 0, b: 30, t:0},
        height: 600,
    };
    Plotly.newPlot(placeid, data, layout);
}
async function plot_vgene_stackedbar(placeid, rbd_varant="WT", chain="H") {
    const vgene = await d3.csv("../_data/tables/vgene.csv");
    const evidence = await d3.csv("../_data/tables/evidence.csv");
    var rbd_evidence = evidence.filter( (row) => row["target_type"] === "RBD");
    var variant_rbd_evidence = rbd_evidence.filter( (row) => row["target"] === rbd_varant);
    var variant_rbd_evidence_abidxs = variant_rbd_evidence.map( (row) => row["ab_idx"] );
    var joined_vgene = [];
    for (var row of vgene) {
        if ((row["ab_idx"] === "") || (row["ab_idx"].startsWith("added"))) {
            row["binding"] = "Not binding";
            joined_vgene.push(row);
        } else if (variant_rbd_evidence_abidxs.includes(row["ab_idx"])) {
            var ab_evidence_binding = variant_rbd_evidence.filter( (evidence) => evidence["ab_idx"] === row["ab_idx"] ).map( (evidence) => evidence["binding"] === "1" );
            row["binding"] = ab_evidence_binding.every( (evidence) => evidence) ? "Binding" : "Not binding";
            joined_vgene.push(row);
        }
    };
    var chain_vgene = joined_vgene.filter( (row) => row["chain"] === chain );
    var [dedup_pair_values, dedup_pair_counts] = valuecount_keys(chain_vgene, ["V gene", "binding"]);
    var isbinding = dedup_pair_values.map( (row) => row[1] === "Binding" );
    var binding_vgenes = dedup_pair_values.filter( (row, index) => isbinding[index] ).map( (row) => row[0] );
    var notbinding_vgenes = dedup_pair_values.filter( (row, index) => !isbinding[index] ).map( (row) => row[0] );
    var binding_vgene_counts = dedup_pair_counts.filter( (row, index) => isbinding[index] );
    var notbinding_vgene_counts = dedup_pair_counts.filter( (row, index) => !isbinding[index] );
    var binding = {
        x: binding_vgenes,
        y: binding_vgene_counts,
        name: "Binding",
        type: "bar",
    };
    var notbinding = {
        x: notbinding_vgenes,
        y: notbinding_vgene_counts,
        name: "Not binding",
        type: "bar",
    };
    var data = [binding, notbinding];
    var layout = {
        margin: {l: 100, r: 0, b: 100, t:0},
        height: 600,
        barmode: "stack",
        xaxis: {tickangle: -45},
    };
    Plotly.newPlot(placeid, data, layout);
}
async function plot_box(tablefilepath, placeid, rbd_varant="WT", column="checked_epitope_group", title="Epitope Group", chain="H", threshold=5) {
    var epitopegrp = await d3.csv(tablefilepath);
    if (tablefilepath.includes("vgene")) {
       epitopegrp = epitopegrp.filter( (row) => row["chain"] === chain ); 
    }
    const evidence = await d3.csv("../_data/tables/evidence.csv");
    var rbd_evidence = evidence.filter( (row) => (row["target_type"] === "RBD") && (["SPR","BLI"].includes(row["evidence"])) );
    var variant_rbd_evidence = rbd_evidence.filter( (row) => row["target"] === rbd_varant);
    var variant_rbd_evidence_abidxs = variant_rbd_evidence.map( (row) => row["ab_idx"] );
    var joined = [];
    for (var row of epitopegrp) {
         if (variant_rbd_evidence_abidxs.includes(row["ab_idx"])) {
            var ab_evidence_kds = variant_rbd_evidence.filter( (evidence) => evidence["ab_idx"] === row["ab_idx"] ).map( (evidence) => parseFloat(evidence["KD"]) );
            row["KD"] = ab_evidence_kds.reduce( (a, b) => a + b ) / ab_evidence_kds.length;
            joined.push(row);
        }
    };
    var [uniq_epitopegrps, uniq_epitopegrp_counts] = valuecount_keys(joined, [column]);
    var traces = [];
    for (var epitopegrp of uniq_epitopegrps) {
        var epitopegrp_subset = joined.filter( (row) => row[column] === epitopegrp[0] );
        if (epitopegrp_subset.length >= threshold) {
            var trace = {
                y: epitopegrp_subset.map( (row) => Math.log10(row["KD"]) ),
                type: "box",
                name: epitopegrp[0],
                boxpoints: 'all',
            };
            traces.push(trace);
        }
    };
    var data = traces;
    var layout = {
        margin: {l: 50, r: 0, b: 100, t:0},
        height: 600,
        xaxis: {
            title: title
        },
        yaxis: {title: "log10(KD)"},
    };
    if (traces.length >= 12) {
        layout.xaxis.tickangle = -45;
    }
    Plotly.newPlot(placeid, data, layout);
}
async function plot_parcats(tablefilepath, placeid, columns=["ds","usage","matched_lineage","binding"], colorselected=false) {
    if (tablefilepath.includes("csv")) {
        var sep = ",";
    } else if (tablefilepath.includes("tsv")) {
        var sep = "\t";
    }
    const ds = await d3.dsv(sep, tablefilepath);
    var colorscale = [[0, 'lightsteelblue'], [1, 'mediumseagreen']];
    var dimensions = [];
    for (var column of columns) {
        if ( column === "ds" ) {
            var dsDim = {
                values: ds.map( (row) => row["ds"] ),
                categoryarray: ["train","val","test","unused"],
                label: "Dataset"
            };
            dimensions.push(dsDim);
        } else if ( column === "usage" ) {
            var usageDim = {
                values: ds.map( (row) => row["usage"] ),
                categoryarray: ["trainval","trainval;negative","unseen_wt","unseen_wt;negative","unseen_omicron","seen_omicron"],
                label: "Usage"
            };
            dimensions.push(usageDim);
        } else if ( column === "matched_lineage" ) {
            var lineageDim = {
                values: ds.map( (row) => row["matched_lineage"] ),
                categoryarray: ["WT","Alpha","R346S","V367F","Q493R","A475V","N440K","L452R","E484K","K417N","S477N","N439K","V483A","Delta","Beta","Mu","Gamma","OmicronBA.1","OmicronBA.1.1","OmicronBA.2"],
                label: "Lineage"
            };
            dimensions.push(lineageDim);
        } else if ( column === "binding" ) {
            var bindingDim = {
                values: ds.map( (row) => row["binding"] ),
                label: "Binding or not",
                categoryarray: ["True", "False"],
                ticktext: ["Binding", "Not binding"],
            };
            dimensions.push(bindingDim);
            var color = bindingDim.values.map( (row) => row === "True" ? 0 : 1 );
            var colorscale = [[0, 'mediumseagreen'], [1, 'lightsteelblue']];
        } else if ( column === "Ab_type" ) {
            var Abtypes_order = ["IgG","Fab","Fv","Nb"];
            Abtypes_order = Abtypes_order.filter( (abtype) => ds.some( (row) => row["Ab_type"] === abtype ) );
            var AbtypeDim = {
                values: ds.map( (row) => row["Ab_type"] ),
                label: "Ab form",
                categoryarray: Abtypes_order,
            };
            dimensions.push(AbtypeDim);
        } else if ( column === "ab_type" ) {
            var abtypeDim = {
                values: ds.map( (row) => row["ab_type"] ),
                label: "Ab form",
                categoryarray: ["IgG","Fab","Fv","Nb"],
            };
            dimensions.push(abtypeDim);
        } else if ( column === "evidence" ) {
            var evidenceDim = {
                values: ds.map( (row) => row["evidence"] ),
                label: "Experiments",
            }
            dimensions.push(evidenceDim);
        } else if ( column === "binding_lv" ) {
            var kdDim = {
                values: ds.map( (row) => parseFloat(row["binding_lv"])-3 ),
                label: "log10(Avg. KD)",
                categoryorder: 'category ascending',
            }
            dimensions.push(kdDim);
            var color = kdDim.values;
            var colorscale = [[0, 'firebrick'], [1, 'lightsteelblue']];
        } else if ( column === "target_type" ) {
            var targettypeDim = {
                values: ds.map( (row) => row["target_type"] ),
                label: "Binding to",
                categoryorder: 'category ascending',
            };
            dimensions.push(targettypeDim);
        } else {
        }
    }
    var traces = [{
        type: "parcats",
        dimensions: dimensions,
        hoverinfo: 'count+probability',
        labelfont: {size: 18},
    }];
    if (colorselected) {
        traces[0].line = {};
        traces[0].line.color = new Int8Array(ds.length);
        traces[0].line.colorscale = [[0, 'gray'], [1, 'firebrick']];
        traces[0].line.cmin = 0;
        traces[0].line.cmax = 1;
    } else {
        traces[0].line = {};
        traces[0].line.color = color;
        traces[0].line.colorscale = colorscale;
    }
    var layout = {
        margin: {l: 20, r: 30, b: 50, t:60},
        height: 600,
    };
    Plotly.newPlot(placeid, traces, layout);
    if (colorselected) {
        function update_color(points_data) {
            var new_color = new Int8Array(ds.length);
            for(var i = 0; i < points_data.points.length; i++) {
                new_color[points_data.points[i].pointNumber] = 1;
            }
            Plotly.restyle(placeid, {'line.color': [new_color]}, 0);
        };
        document.getElementById(placeid).on('plotly_click', update_color);
    }
};
plot_parcats("../_data/tables/evidence.csv", "plot-distribution-evidence", ["evidence","Ab_type","target_type"], true);
// plot_sunburst("../_data/tables/evidence.csv", "plot-distribution-evidence", "evidence", "Ab_type");
plot_sunburst("../_data/tables/record.csv", "plot-distribution-record", "source", "ab_type");
plot_vgene_stackedbar("plot-correlations-vgene");
// plot_box("../_data/tables/vgene.csv", "plot-correlations-vgene", "WT", "V gene", "IGHV Gene", "H", 20);
plot_box("../_data/tables/epitope_group.csv", "plot-correlations-epitope");
plot_parcats("../_data/datasets/classification_variantrbd.tsv", "plot-ds-split-clsds");
plot_parcats("../_data/datasets/regression_wtrbd.tsv", "plot-ds-split-rankds", ["ds","Ab_type","evidence","binding_lv"]);
$("#plot-distribution").text("");
$("#plot-correlations").text("");
$("#plot-ds-split").text("");
function click_changecolor(element) {
    if (element.style.color === "grey") {
        element.style.color = "black";
    } else {
        element.style.color = "grey";
    }
}
$(document).ready(function() {
    $("#vgene-button").click(function() {
        var subset = $("#vgene-select-subset").val();
        var chain = $("#vgene-select-chain").val() === "Light chain" ? "L" : "H";
        $("#vgene-button").text("Loading...");
        if (subset === "All antibodies") {
            plot_vgene_stackedbar("plot-correlations-vgene","WT",chain);
        } else {
            plot_box("../_data/tables/vgene.csv", "plot-correlations-vgene", "WT", "V gene", "IGV Gene", chain, 20);
        }
        $("#vgene-button").text("Apply");
    });
    $("details summary h2").click(function(){
        click_changecolor(this);
    })
})
</script>
<h1 id="distribution">Distribution of collected data</h1>
<div id="plot-distribution">Loading...</div>
<details open>
<summary><h2>Collected binding evidence  <i class="fas fa-angle-right"></i></h2></summary>
<div id="plot-distribution-evidence"></div></details>
<details open>
<summary><h2>Collected records of sequences  <i class="fas fa-angle-right"></i></h2></summary>
<div id="plot-distribution-record"></div></details>

<h1 id="correlations">Correlations with binding affinity</h1>
<div id="plot-correlations">Loading...</div>
<details open>
<summary><h2>IGV genes and binding affinities  <i class="fas fa-angle-right"></i></h2></summary>
<form id="vgene-options">
<label for="vgene-select-subset">Subset to show: <select id="vgene-select-subset">
<option selected>All antibodies</option>
<option>Only those with <emphasis>K<sub>D</sub></emphasis></option>
</select></label>
<label for="vgene-select-chain">Chain: <select id="vgene-select-chain">
<option selected>Heavy chain</option>
<option>Light chain</option>
</select></label>
<div><a href="#correlations" class="btn btn--primary" id="vgene-button">Apply</a></div>
</form>
<div id="plot-correlations-vgene"></div></details>
<details open>
<summary><h2>Epitope groups and binding affinities  <i class="fas fa-angle-right"></i></h2></summary>
<div id="plot-correlations-epitope"></div></details>

<h1 id="ds-split">Split of datasets</h1>
<div id="plot-ds-split">Loading...</div>
<details open>
<summary><h2>Binary classification dataset  <i class="fas fa-angle-right"></i></h2></summary>
<div id="plot-ds-split-clsds"></div></details>
<details open>
<summary><h2>Ranking dataset  <i class="fas fa-angle-right"></i></h2></summary>
<div id="plot-ds-split-rankds"></div></details>