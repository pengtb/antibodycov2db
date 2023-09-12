---
permalink: /utilities/
title: "Utilities"
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
textarea.multiline {
    display: block;
}
a[class="btn btn--primary"] {
    display: block;
    float: center;
    text-align: center;
}
textarea[id="mutant-rbd-seq-output"] {
    font-family: monospace;
}
div.structure {
    width: 900px;
    height: 400px;
    position:relative;
}
</style>
<link rel="stylesheet" type="text/css" href="https://www.ebi.ac.uk/pdbe/pdb-component-library/css/pdbe-molstar-light-3.1.0.css">
<script type="text/javascript" src="https://www.ebi.ac.uk/pdbe/pdb-component-library/js/pdbe-molstar-plugin-3.1.0.js"></script>
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
<script>
    $(document).ready(function() {
        $("#mutant-rbd-seq-button").click(function() {
            $("#mutant-rbd-seq-button").text("Generating...");
            var wt_seq = "RVQPTESIVRFPNITNLCPFGEVFNATRFASVYAWNRKRISNCVADYSVLYNSASFSTFKCYGVSPTKLNDLCFTNVYADSFVIRGDEVRQIAPGQTGKIADYNYKLPDDFTGCVIAWNSNNLDSKVGGNYNYLYRLFRKSNLKPFERDISTEIYQAGSTPCNGVEGFNCYFPLQSYGFQPTNGVGYQPYRVVVLSFELLHAPATVCGPKKSTNLVKNKCVNF";
            var wt_seqarr = wt_seq.split("");
            var rbd_resids = [];
            for (var i = 319; i <= 541; i++) {
                rbd_resids.push(i);
            };
            var mutant_seqarr = Array.from(wt_seqarr);
            var mutations = $("#mutant-rbd-seq-input textarea").val().split(",");
            for ( var mutation of mutations) {
                var mut_resid = parseInt(mutation.substring(1, mutation.length-1));
                var mut_aa = mutation.substring(mutation.length-1, mutation.length);
                rbd_resids.map(function(rbd_resid, idx) {
                    if (rbd_resid == mut_resid) {
                        mutant_seqarr[idx] = mut_aa;
                    }
                })
            };
            var mutant_seq = mutant_seqarr.join("");
            console.log(mutant_seq);
            $("#mutant-rbd-seq-output").val(mutant_seq);
            $("#mutant-rbd-seq-button").text("Generate");
        });
    })
</script>


<h1 id="mutant-rbd-seq">Generate mutated RBD sequence</h1>
<form>
<div id="mutant-rbd-seq-input"><label class="multiline">Type in mutations: <textarea rows="2" cols="100" class="multiline" placeholder="such as: E484K,N501Y"></textarea></label></div>
<div><a href="#mutant-rbd-seq" class="btn btn--primary" id="mutant-rbd-seq-button">Generate</a></div>
</form>
<textarea id="mutant-rbd-seq-output" rows="4" cols="80" readonly wrap="soft">RVQPTESIVRFPNITNLCPFGEVFNATRFASVYAWNRKRISNCVADYSVLYNSASFSTFKCYGVSPTKLNDLCFTNVYADSFVIRGDEVRQIAPGQTGKIADYNYKLPDDFTGCVIAWNSNNLDSKVGGNYNYLYRLFRKSNLKPFERDISTEIYQAGSTPCNGVEGFNCYFPLQSYGFQPTNGVGYQPYRVVVLSFELLHAPATVCGPKKSTNLVKNKCVNF</textarea>
<h1 id="rbd-epitope-viewer">Visualize RBD epitopes</h1>
<form>
<div id="rbd-epitope-viewer-input"><label class="multiline">Type in mutations: <textarea rows="2" cols="100" class="multiline" placeholder="such as: E484K,N501Y"></textarea></label></div>
<div><a href="#rbd-epitope-viewer" class="btn btn--primary" id="rbd-epitope-viewer-button">Visualize</a></div>
</form>
<div id="rbd-epitope-viewer-output" class="structure"></div>

<script>
var epitope_viewerInstance = new PDBeMolstarPlugin();
var options = {
    customData: { url: 'https://www.ebi.ac.uk/pdbe/model-server/v1/7wz2/atoms?label_entity_id=1&auth_asym_id=A&encoding=bcif', format: 'cif', binary:true },
    hideControls: true,
    landscape: true,
    pdbeLink: true,
    sequencePanel: true,
    bgColor: {r:255, g:255, b:255},
}
var viewerContainer = document.getElementById('rbd-epitope-viewer-output');
epitope_viewerInstance.render(viewerContainer, options);
epitope_viewerInstance.events.loadComplete.subscribe(function() {
    epitope_viewerInstance.visual.select({data:[{start_residue_number: 319, end_residue_number: 541, focus: false, sideChain: false}]});
})
$(document).ready(function() {
    $("#rbd-epitope-viewer-button").click(function() {
        $("#rbd-epitope-viewer-button").text("Visualizing...");
        var epitopes = $("#rbd-epitope-viewer-input textarea").val();
        epitope_viewerInstance.visual.reset({camera: true, theme: true})
        if (epitopes == "") {
        } else {
            var epitopes_selections = [];
            $.each(epitopes.split(","), function(key, value) {
                epitopes_selections.push({
                    color:{r:0, g:0, b:255},
                    residue_number: parseInt(value.substring(1, value.length-1)),
                    sideChain: true,
                    focus: true,
                });
            });
            epitope_viewerInstance.visual.select({data: epitopes_selections, nonSelectedColor:{r:255, g:255, b:255}});
        }
        $("#rbd-epitope-viewer-button").text("Visualize");
    })
})
</script>