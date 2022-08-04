# Chia-hashlips-and-photoshop-image-and-metadata-creation-tool

This is our example that we used to generate the Monkeyzoo collections.

// Simple art generator by HashLips <->

#include "./lib/json2.js";

function main() {
  var continueConfirmation = confirm(
    "You are about to use the HashLips art generator. Are you sure you want to continue?"
  );

  if (!continueConfirmation) return;

  var supply = prompt("How many images do you want to generate?", "10");
  
  var monkey = prompt("Which character are you creating?", "");

  alert(
    supply +
      " images will be generated, so sit back relax and enjoy the art being generated."
  );

  var groups = app.activeDocument.layerSets;

  resetLayers(groups);

  function getRWeights(_str) {
    var weight = Number(_str.split("#").pop());
    if(isNaN(weight)){
      weight = 1;
    }
    return weight;
  }

  function cleanName(_str) {
    return _str.split("#").shift();
  }

  for (var h = 1; h < parseInt(supply) + 1; h++) {
    var obj = {};
    obj.format = "Chip-0007";
    obj.name = monkey + "Monkeyzoo collection" + " #" + h;
    obj.description = "YOUR DESCRIPTION";
    obj.series_number = h;
    obj.series_total = "1000";
    obj.minting_tool = "NFTr_minting_tool";
    obj.sensitive_content = "false";
    obj.collection = [];
    obj.attributes = [];
    for (var i = 0; i < groups.length; i++) {
      var totalWeight = 0;
      var layerMap = [];

      for(var j = 0; j < groups[i].layers.length; j++){
        totalWeight += getRWeights(groups[i].layers[j].name);
        layerMap.push({
          index: j,
          name: cleanName(groups[i].layers[j].name),
          weight: getRWeights(groups[i].layers[j].name)
        });
      }

      var ran = Math.floor(Math.random() * totalWeight);

      (function() {
        for(var j = 0; j < groups[i].layers.length; j++){
          ran -= layerMap[j].weight;
          if(ran < 0) {
            groups[i].layers[j].visible = true;
            obj.attributes.push({
              trait_type: groups[i].name, 
              value: layerMap[j].name
            })
            return;
          }
        }
      })();
    }
    obj.collection.push(
      {name: "Monkeyzoo_collection"},
            {id: "3d63a115-c341-471c-8e50-377c4cb67f40"},
            {attributes : [
              { type : "icon" ,  value : "https://pbs.twimg.com/profile_images/1542615309969985536/wK9gfnMV_400x400.png" },
              { type : "banner" ,  value : "https://pbs.twimg.com/profile_banners/14711228/1656623125/600x200" },
              { type : "twitter" , value : "https://twitter.com/monkeyzoo" },
              { type : "website" , value : "https://www.monkeyzoo.net" }
              ]})
    saveImage(obj.series_number);
    saveMetadata(obj);
    resetLayers(groups);
  }
  alert("Generation process is complete.");
}

function resetLayers(_groups) {
  for (var i = 0; i < _groups.length; i++) {
    _groups[i].visible = true;
    for (var j = 0; j < _groups[i].layers.length; j++) {
      _groups[i].layers[j].visible = false;
    }
  }
}

function saveImage(_series_number) {
  var saveFile = new File(toFolder("build/images") + "/" + _series_number + ".png");
  exportOptions = new ExportOptionsSaveForWeb();
  exportOptions.format = SaveDocumentType.PNG;
  exportOptions.PNG24 = false;
  exportOptions.transparency = true;
  exportOptions.interlaced = false;
  app.activeDocument.exportDocument(
    saveFile,
    ExportType.SAVEFORWEB,
    exportOptions
  );
}

function saveMetadata(_data) {
  var file = new File(toFolder("build/metadata") + "/" + _data.series_number + ".json");
  file.open("w");
  file.write(JSON.stringify(_data));
  file.close();
}

function toFolder(_name) {
  var path = app.activeDocument.path;
  var folder = new Folder(path + "/" + _name);
  if (!folder.exists) {
    folder.create();
  }
  return folder;
}


main();
