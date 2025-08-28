// This expression joins features from a feature service and table, the services are joined using a primary (feautreservice) and foreign (table) key --- creating one table
// Attributes must be explicitly defined. Replace all items with the ✏️ symbol next to them.  

var portal = Portal("https://maps.test.cce.af.mil/soe");

// Source feature layer 
var FL = FeatureSetByPortalItem(
  portal,
  "c8384d02df774ff6b3447a38f8f87fb9", //  ✏️ replace item ID
  0,                                  //  ✏️ replace layer ID (if needed)
  ["specieslocationidpk",             //  ✏️ list fields to add from the source layer
   "country"
  ]
);

// Standalone table
var TBL = FeatureSetByPortalItem(
  portal,
  "52e8674d7617470f8111f3257bf68476", // ✏️ replace item ID
  0,                                  // ✏️ replace layer ID (if needed)
  [
    "specieslocationidfk",            // ✏️ list fields to add from table
    "treenotes",
    "dbh",
    "treeconditionclass",
    "primarymaintenanceneed",
    "plantingsitepriority",
    "treecode",
    "treelocationtype"
  ]
);

// Define output schema (no geometry) ✏️ list all fields and schema type
var outFields = [
  {"name":"specieslocationidpk", "type":"esriFieldTypeString"},
  {"name":"country",             "type":"esriFieldTypeString"},
  {"name":"treenotes",           "type":"esriFieldTypeString"},
  {"name":"dbh",                 "type":"esriFieldTypeInteger"},
  {"name":"treeconditionclass",  "type":"esriFieldTypeString"},
  {"name":"primarymaintenanceneed","type":"esriFieldTypeString"},
  {"name":"plantingsitepriority","type":"esriFieldTypeString"},
  {"name":"treecode",            "type":"esriFieldTypeString"},
  {"name":"treelocationtype",    "type":"esriFieldTypeString"},
  {"name":"join_text",           "type":"esriFieldTypeString"} 
];

var schema = {
  "fields": outFields,
  "geometryType": "",
  "features": []
};

// Build output features (loops through all features in var id, finds all features in table with the matching foreign key - return their attributes 
for (var f in FL) {
  var id = f.specieslocationidpk; // ✏️ primary key from feature layer
  if (IsEmpty(id)) { continue; }

  var rel = First(Filter(TBL, "specieslocationidfk = @id")); // ✏️ foreign key from table

  function show(v) { return IIf(IsEmpty(v), "—", v); }

  var attrs = {  // ✏️ edit which attributes from both services to be displayed in the joined table
    "specieslocationidpk": id,
    "country":           f.country,
    "treenotes":           IIf(IsEmpty(rel), null, rel.treenotes),
    "dbh":                 IIf(IsEmpty(rel), null, rel.dbh),
    "treeconditionclass":  IIf(IsEmpty(rel), null, rel.treeconditionclass),
    "primarymaintenanceneed": IIf(IsEmpty(rel), null, rel.primarymaintenanceneed),
    "plantingsitepriority":IIf(IsEmpty(rel), null, rel.plantingsitepriority),
    "treecode":            IIf(IsEmpty(rel), null, rel.treecode),
    "treelocationtype":    IIf(IsEmpty(rel), null, rel.treelocationtype),
  };

  Push(schema.features, {"attributes": attrs});
}

// Return a FeatureSet
return FeatureSet(Text(schema));
