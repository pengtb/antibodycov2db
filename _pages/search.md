---
permalink: /search/
title: "Search"
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
  input.oneline {
    width: 25%;
    display: inline-block;
  }
  textarea.multiline {
    display: block;
  }
  a[class="btn btn--primary"] {
    display: block;
    float: center;
    text-align: center;
  }
  label.labeloneline {
    width: 70%;
    display: inline;
  }
  label.multiline {
    /* display: block; */
  }
  label.radio {
    display: flex;
    text-indent: 5px;
    padding: 0 0 0 30px;
  }
  div.divradio {
  display: flex;
  /* line-height: 0.75; */
  /* padding: 0 15px 0 0; */
}
</style>
<script type="text/javascript" src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js"></script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/fuse.js@6.6.2"></script>
<script type="text/javascript" src="https://raw.githubusercontent.com/douglascrockford/JSON-js/master/json2.js"></script>
<script>
$(document).ready(function() {
    $.ajax({
          url: "../_data/tables/name.csv",
          type: "GET",
          async: false,
          dataType: "text",
          success: function(data) {
              var parsed = $.csv.toObjects(data);
              var valid_ids = [];
              for (var i = 0; i < parsed.length; i++) {
                  valid_ids.push(parsed[i]["ab_idx"]);
              }
              $("#search-by-id-button").click(function() {
                  var id = $("#search-by-id input").val();
                  if (valid_ids.includes(id)) {
                      window.location.href = "../abdetail/?ab_idx=" + id;
                  } else {
                      var maxid = Math.max.apply(Math, valid_ids);
                      alert("Invalid ID. ID is between 0 and " + maxid);
                  }
              });
              $("#search-by-name-button").click(function() {
                  $("#search-by-name-button").text("Searching...");
                  var queryname = $("#search-by-name input").val();
                  var method = $("#search-name-radio-method input:checked").val();
                  if (method == "exact") {
                      var found = false;
                      for (var i = 0; i < parsed.length; i++) {
                          var all_names = parsed[i]["all_names"].split(";");
                          if (all_names.includes(queryname)) {
                              found = true;
                              window.location.href = "../abdetail/?ab_idx=" + parsed[i]["ab_idx"];
                              break;
                          };
                      };
                      if (!found) {
                          alert("No results found. Try approximate search.");
                      }
                  } else {
                      const namefuse = new Fuse(parsed, {
                          keys: ["ab_idx", "all_names"],
                      });
                      var results = namefuse.search(queryname);
                      const maxnum_results = 400;
                      results = results.slice(0, maxnum_results);
                      var searched_abidxs = [];
                      for (var i = 0; i < results.length; i++) {
                          searched_abidxs.push(results[i]["item"]["ab_idx"]);
                      }
                      window.location.href = "../searchresults/?ab_idxs=" + JSON.stringify(searched_abidxs);
                  };
              });
              $.ajax({
                  url: "../_data/tables/record.csv",
                  type: "GET",
                  async: false,
                  dataType: "text",
                  success: function(data) {
                      var parsed_record = $.csv.toObjects(data);
                      $("#search-by-sequence-button").click(function() {
                          $("#search-by-sequence-button").text("Searching...");
                          var queryseq = $("#search-by-sequence textarea").val();
                          var method = $("#search-name-radio-method input:checked").val();
                          var threshold = $("#search-by-sequence-identity input").val();
                          var chain = $("#search-by-sequence-chain-select").val();
                          if (chain == "Both") {
                              var recordkeys = ["Hseq","Lseq"];
                          } else if (chain == "Heavy chain") {
                              var recordkeys = ["Hseq"];
                          } else {
                              var recordkeys = ["Lseq"];
                          };
                          const recordfuse = new Fuse(parsed_record, {
                              keys: recordkeys,
                              threshold: threshold
                          });
                          var results = recordfuse.search(queryseq);
                          const maxnum_results = 400;
                          results = results.slice(0, maxnum_results);
                          var searched_abidxs = [];
                          for (var i = 0; i < results.length; i++) {
                              searched_abidxs.push(results[i]["item"]["ab_idx"]);
                          };
                          var rmdup_abidxs = Array.from(new Set(searched_abidxs));
                          var joined_abidxs = rmdup_abidxs.join(",");
                          window.location.href = "../searchresults/?ab_idxs=" + joined_abidxs;
                      });
                  }
             })
          }
    });

});
</script>

<h1 id="by-id">By ID</h1>
<form>
<div id="search-by-id"><label class="labeloneline">Type in Ab ID: <input type="number" min="0" class="oneline"></label></div>
</form>
<div><a href="#by-id" class="btn btn--primary" id="search-by-id-button">Go</a></div><br>
<h1 id="by-name">By name</h1>
<form>
<div id="search-by-name"><label class="labeloneline">Type in Ab name: <input type="text" class="oneline"></label></div>
<div class="divradio" id="search-name-radio-method">Method:  <label class="radio" for="search-name-radio-exact"><input type="radio" id="search-name-radio-exact" value="exact" name="search-name-radio" checked>Exact</label><label class="radio" for="search-name-radio-approximate"><input type="radio" id="search-name-radio-approximate" value="approximate" name="search-name-radio">Approximate</label></div>
</form>
<div><a href="#by-name" class="btn btn--primary" id="search-by-name-button">Search</a></div><br>
<h1 id="by-sequence">By sequence</h1>
<form>
<div id="search-by-sequence-chain">Select the chain: <select id="search-by-sequence-chain-select"><option>Heavy chain</option><option>Light chain</option><option selected>Both</option></select></div>
<div id="search-by-sequence-identity" style="display: flex"><label class="labeloneline">Type in identity threshold: <input type="number" min="0" max="1" placeholder="0.9" class="oneline"></label></div>
<div id="search-by-sequence"><label class="multiline">Type in HC/LC sequence: <textarea rows="5" cols="100" class="multiline"></textarea></label></div>
</form>
<div><a href="#by-sequence" class="btn btn--primary" id="search-by-sequence-button">Search</a></div><br>
