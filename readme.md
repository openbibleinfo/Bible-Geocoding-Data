# Introduction

This project attempts to catalog the location of every identifiable place mentioned in the Protestant Bible. It draws from over seventy modern sources (mostly Bible commentaries, dictionaries, encyclopedias, and atlases), with over 400 unique sources cited, and catalogs the possible locations of each place mentioned as well as the confidence in the identification of that place. It also catalogs the expression of these places in ten English translations of the Bible, links the data to semantic databases like Wikidata where possible, and provides a thumbnail image for every location.

It is a substantial update to my previous attempt to catalog this data in 2007.

Read a blog post for a [higher-level overview of this project](https://www.openbible.info/blog/2021/11/rethinking-the-bible-atlas/).

## Definitions

In this documentation, a "place" refers to an ancient place mentioned in the Bible, such as Jerusalem, and a "location" refers to a modern location, such as Jerusalem (ancient places and modern locations can have the same name but don't necessarily refer to exactly the same location).

## License

This data is licensed under a [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/) license.

OpenStreetMap data is licensed under [ODbL 1.0](https://opendatacommons.org/licenses/odbl/), which is similar to CC-BY-SA.

The licenses for images vary depending on the image but are open and are generally Creative Commons or similar.

# About these files

These files allow you to create your own expression of the data. Each file is in [JSON Lines](http://jsonlines.org/) format (i.e., each line in the file is a complete JSON object).

The data files are:

1. ancient.jsonl contains data about the places as the Bible text mentions them.
2. modern.jsonl contains data about the modern locations identifiable with ancient places.
3. geometry.jsonl contains metadata about complex objects such as rivers and regions that aren't expressible as a single point.
4. image.jsonl contains metadata for images.
5. source.jsonl contains data about the sources used to construct this dataset.
6. https://a.openbible.info/geo/thumbnails.zip (180 MB) contains 512x512 thumbnail images for every location.
7. all.kml is a partial representation of the data for quick previewing in Google Earth.
8. Thousands of GeoJSON and KML files provide geometry data.

The structure of the JSON is somewhat complex. Sorry!

# Ancient places (ancient.jsonl)

This file contains data about the places as mentioned in the Bible text, disambiguates them, catalogs them by verse reference, and evaluates the confidence of modern scholarship that attempts to identify ancient places with modern locations.

## Identifications (`identifications`)

`id_source` indicates whether the identification is "ancient", "modern", or "special". An "ancient" source is usually when a place is identified as being the same as another place with a different name, such as "Jerusalem" being a later name for "Jebus". A "modern" source is a location. With both these `id_source`s, look at the `id` property to resolve the target. For "special", see Special Resolutions below; a "special" `id_source` has a `special` property rather than an `id`.

`class` indicates whether the identification is human-made (such as a settlement) or natural (such as a mountain). This property is useful mostly for identifying whether regions are political ("Judea") or natural ("Arabah"). Its possible values are "human", "natural", "special", or "human,natural" (indicating that some resolutions are human-made and some are natural). In the last case, each item in `resolutions` has a `class` of "human" or "natural" unless it's a "special" resolution, in which case no `class` appears. The `class` doesn't necessarily match the `class` of the modern location. For example, an ancient settlement ("human") at a modern spring ("natural") doesn't match.

`contained_in` and `contains` reflect relationships among the identifications. For example, one identification might be "near Gibeah", and a second identification might be a specific location near Gibeah. The former encompasses the latter and has a `contains` key, while the latter is in the former and has a `contained_in` key. Each is an array of positions in the same `identifications` array, with the first being position 0. A `contains` key in one identification always has a corresponding `contained_in` key in another.

`description` is a human-readable description of the identification. For example: `along the <modern id="m664b51">Wadi el Esh</modern>`. The embedded XML tag can be `<modern>` or `<ancient>`, with `@id` indicating the id of the target.

`geometry_id` refers to a geometry definition; it is often used when the target location is a point, but the biblical place refers to a region. Alternatively, `geometry_radius_meters` indicates an approximate radius around the target location where the biblical location applies and no specific geometry is available. For example, several possible locations for [Havilah](https://www.openbible.info/geo/ancient/ad7e819/havilah-1) refer to a region of indeterminate size, which I've attempted to quantify--at least in terms of order of magnitude. When `geometry_radius_meters` appears, each item in `resolutions` has a `radius_geometry_id` pointing to approximate circle geometry.

`media.thumbnail` may exist if there's only one resolution, and logically it might make sense to show a thumbnail for the identification rather than for the resolution. For example, an identification that is "another name for [a particular region]" would have the thumbnail for the region in the identification, while the resolution might have a thumbnail for the specific coordinates in the region.

`modifier` reflects a relationship to the identified object. The possible values are:

* "<": the place is inside the identification; for example, the [East Gate](https://www.openbible.info/geo/ancient/a4e8401/east-gate) is in [Jerusalem](https://www.openbible.info/geo/ancient/a15257a/jerusalem)
* "along": the place is along the identified river
* "near": the place is near the location, with `geometry_radius_meters`, `geometry_id`, and `precise_geometry_id` approximately quantifying the distance in which the resolution likely appears
* "on": the place is on a mountain
* ">": the place is a region surrounding the location

`types` reflects the possible types that the identification can resolve to:

* altar
* body of water
* campsite
* canal
* cliff
* district in settlement
* field
* ford
* forest
* fortification
* garden
* gate
* hall
* hill
* island
* mine
* mountain
* mountain pass
* mountain range
* mountain ridge
* natural area
* people group
* pool
* promontory
* region
* river
* road
* rock
* room
* settlement
* settlement and spring
* special
* spring
* stone heap
* structure
* tree
* valley
* wadi
* well

### Votes

`sources` is an array containing all the books that contributed to this place's score.

`tags` is an object with aggregated vote counts. The possible tags are:

* confidence_yes: no substantial doubt about the identification
* confidence_likely: the identification is likely
* confidence_map: the identification is based on the position on a map rather than on text
* confidence_mostlikely: the identification is the most likely of the options
* confidence_possible: the identification is possible or tentative, or a few scholars make the identification
* confidence_unlikely: the identification is unlikely
* confidence_no: the identification is wrong
* identified_is: the place "is identified" as being at the location
* identified_been: the place "has been identified" as being at the location
* identified_adjective: the place, identified at the location, without a verb
* authority_old: this identification used to be popular but is no longer
* authority_parallel: a parallel passage makes the identification
* authority_preserved: the location preserves the place name
* authority_scholar: one scholar makes the identification
* authority_traditional: a tradition predating the year 1800 makes the identification
* authority_usually: scholars generally make the identification
* authority_variant: a variant manuscript or translation makes the identification
* unknown: the location is unknown
* uncertain: the location is uncertain

For example:

```json
"confidence_likely": 10,
"confidence_yes": 5,
"identified_been": 1,
"identified_is": 1
```

### Score

`score` is an object that records aggregate vote information for the identification. There are two types of scores: vote scores and time-weighted scores.

```json
"time_best_fits": [805, 737, 668, 600, 531, 462]
"time_intercept": 14331,
"time_r_squared": 0.263,
"time_slope": -6.865,
"time_values": [1000, 729, 260, 668, 677, 469],
"time_total": 462,
"vote_average": 24,
"vote_count": 19,
"vote_total": 456,
```

#### Vote scores

The three vote keys (`vote_average`, `vote_count`, and `vote_total`) reflect the average vote score, the total number of votes, and the sum of the vote values. A total of 500 with a single vote means there's no substantial disagreement on the location (e.g., [Dan](https://www.openbible.info/geo/ancient/a513646/dan)). An overall total of 500 or higher represents high confidence in the identification.

Here are the values that contribute to the vote score:

| Vote                  | Contribution to score |
| --------------------- | --------------------: |
| confidence_yes        |                    30 |
| identified_is         |                    25 |
| confidence_likely     |                    24 |
| confidence_map        |                    24 |
| confidence_mostlikely |                    23 |
| identified_been       |                    22 |
| identified_adjective  |                    21 |
| authority_preserved   |                    20 |
| authority_usually     |                    19 |
| authority_scholar     |                    17 |
| authority_parallel    |                    16 |
| authority_variant     |                    15 |
| authority_traditional |                    14 |
| confidence_possible   |                    10 |
| authority_old         |                     3 |
| unknown               |                     0 |
| uncertain             |                     0 |
| confidence_unlikely   |                   -10 |
| confidence_no         |                   -20 |


#### Time-weighted scores

Time-weighted scores attempt to quantify the trajectory of confidence in the identification. The `time_best_fits` and `time_values` each reflect about a decade of scholarship as reflected in the sources I used:

1. 1969 and before (first value, position 0)
2. 1970-1979
3. 1980-1989
4. 1990-1999
5. 2000-2009
6. 2010 and after (last value, position 5).

The values themselves reflect the confidence of the identification and are derived from the vote scores. They're an integer best viewed as a fraction of 1000, where 1000 represents very high confidence. For example, a score of 100 reflects a 10% confidence in an identification, while a score of 500 reflects a 50% confidence. Achieving a score of 1000 depends on the number of sources I consulted that were published in the relevant time period. In other words, the vote score required to reach 1000 varies depending on the period--I consulted more sources published in 2000-2009 than 1969 and before. If the sources published during the period unanimously had high confidence in an identification, then the value would be 1000.

For a given place, the sum of the `time_values` across all identifications never totals more than 1000 for a particular time period. For example, if a place had three identifications, in the 2000-2009 period you'd never see each one with a value of 600 because then the total score would be 1800.

The `time_values` array reflects the confidences for each decade. In the above JSON example, the score of 1000 in the "1969 and before" period (first value in the array) means that there were no major questions about the identification. Over the decades, the confidence has generally drifted lower.

The `time_best_fits` array reflects a least-squares best-fit line (i.e, [linear regression](https://en.wikipedia.org/wiki/Linear_regression)) applied to `time_values`. The `time_intercept`, `time_r_squared` (a measure of how well the line fits the data), and `time_slope` (a positive value indicates increasing confidence, while a negative value indicates decreasing) values all derive from this line. The y-axis values reflect the end of each time period (i.e., 1969, 1979, 1989, 1999, 2009, and 2020), which is why you'll often see very big or very small intercept values: 0 is far away from 1969.

`time_total` is the same as the last value in `time_values` and, in theory, reflects the confidence of current scholarship. This value is used throughout the dataset as the basis for sorting identifications and resolutions.

If there's no major dispute about the identification, the `time_total` and `time_intercept` will be 1000, the `time_best_fits` and`time_values` arrays will be empty, and the `time_r_squared` and `time_slope` values will be 0. The empty arrays unambiguously identify this situation.

### Resolutions

Resolving the ultimate location(s) of a place isn't trivial, as a place may be identified with another place, which may be identified with another place or several possible locations, and so on. The `resolutions` array follows all these paths, giving you all the possible locations for a place.

Here's an example object in `resolutions`:

```json
"ancient_geometry": "point",
"best_path_score": 44,
"best_time_score": 133,
"description": "<modern id=\"m9f98b7\">Zafar</modern>",
"geojson_roles": {
  "point": {
    "description": "<modern id=\"m9f98b7\">Zafar</modern>",
    "id": "m9f98b7.point"
  }
},
"land_or_water": "land",
"lonlat": "44.402874,14.213898",
"lonlat_type": "point",
"modern_basis_id": "m9f98b7",
"paths": [
  [
    {
      "ancient_id": "a938779",
      "identification_i": 3
    },
    {
      "ancient_id": "a3cae2c",
      "identification_i": 0
    },
    {
      "modern_id": "m9f98b7"
    }
  ]
],
"media": {
  "thumbnail": {
    "credit": "P. Yule",
    "credit_url": "https://commons.wikimedia.org/wiki/File:10~014a_Kopie.jpg",
    "description": "artifact from <modern id=\"m9f98b7\">Zafar</modern>",
    "file": "m9f98b7.ie61056.jpg",
    "image_id": "ie61056"
  },
},
"type": "settlement"
```

Here the object resolves to a modern location, as indicated by the `modern_basis_id`, which provides the id of the modern location that provides the base coordinates (in `lonlat`).

The `paths` array is an array of arrays, with each sub-array containing objects indicating the resolution path. The first item in the sub-array always points to the current item in the `identifications` array. Later objects in the sub-array point to other ancient sources, with the `identification_i` indicating the item in the `identifications` array for the object indicated in `ancient_id`. In the above example, the path has three steps: the current place, the first ("0") object of the `identifications` array of the place with `id` "a938779", and finally the modern location with the id `m9f98b7`.

`best_path_score` shows the score of the path with the highest average `score` (following all the steps in the path). In principle, a higher score means a higher-confidence resolution. A score of 500 or higher indicates very high confidence.

`best_time_score` reflects the highest time-weighted scores for the paths. It only appears if there are intermediate steps in a path (i.e., three or more items). The `best_time_score` is used as a component of the score in the top-level `modern_associations` object.

`class` has a value of "human" or "natural". If the parent identification has one class, the resolution `class` matches the identification's class. Otherwise, the resolution `class` is one of the classes in the parent identification.

`description` is a human-readable description of the resolution, including a `<modern id="...">...</modern>` tag when the resolution isn't `special`.

`geojson_roles` contains an `id` that matches an id in the root object's `geojson_file`, as well as a `description` of the geometry suitable for a label. Its other possible properties are:

* `center`: the center of a circle (used to indicate uncertainty or approximate size); the `geometry` role contains the approximate geometry for the circle
* `geometry`: path or polygon geometry for the feature; when both `geometry` and `precise` appear, `geometry` reflects the creation of data independently from OpenStreetMap
* `local`: when a settlement or other point feature (such as [Megiddo](https://www.openbible.info/geo/ancient/a8554e3/megiddo)) has OpenStreetMap geometry delineating the extent of existing historical remains
* `point`: standard point feature (such as a settlement) that has no polygon or path geometry; a `local` polygon may also exist but can be ignored unless you want to render the extent of visible remains
* `precise`: geometry from OpenStreetMap
* `representative_point`: point inside a region or along a path; if you're displaying path or polygon geometry and not just points, you probably don't want to show this role
* `settlement`: coordinates for the settlement in which the place existed, and no better coordinates are available; typically used for features (such as a gate) located in a particular settlement
* `simplified_geometry`, `simplified_local`, and `simplified_precise` appear when the corresponding standard property has over 100 points. These properties reduce the number of points to approximately 100 for improved rendering performance

`local_geometry_id` appears when the resolution is a point (e.g., a settlement), but OpenStreetMap has polygon geometry. For example, [Amman](https://www.openbible.info/geo/modern/m0c1bc5/amman) has geometry for the archaeological site there.

`lonlat` appears if the place ultimately resolves to a modern location (i.e., it isn't a `special` resolution). It's a comma-separated longitude, latitude pair.

`lonlat_type` describes the `lonlat`. Its possible values are (and match the property in `geojson_roles`):

* center: the coordinates are at the center of a circle bounding where a place could be. Look at `geometry_radius_meters` for the size of the radius (in meters) defining the circle and `geometry_id` for a polygon that approximates the circle
* point: the place is defined by a point
* representative point: the place is a region or path, and the `lonlat` is a point inside the region or along the path. See `geometry_id` or `precise_geometry_id` for a definition of the region or path
* settlement: the place is somewhere inside the settlement located at `lonlat`. For example, several city gates are at unknown locations inside Jerusalem, in which case the provided `lonlat` is for Jerusalem

`precise_geometry_id` appears when exact path or polygon data from OpenStreetMap is available. `geometry_id` appears when less-precise data is available. Both keys can appear in the same object.

`media.thumbnail` has information about the thumbnail image for this resolution if it resolves to a modern location. When deciding what thumbnail to show for a resolution, I recommend looking for a media.thumbnail in the top-level ancient object, then in the identification, and lastly in the resolution. In other words, a thumbnail at a higher level in the object will probably reflect a more-relevant thumbnail.

`in` appears when the `modifier` is `<` and indicates whether the place is in a "settlement" or a "region".

### Special resolutions

Not all places can be resolved to a location. Such `special` resolutions have the following possible values:

* multiple_locations: the place refers to multiple locations, such as the tabernacle during the exodus
* nonspecific_place: indicates a symbolic or prophetic place that may or may not correspond to a real location
* not_a_place: for example, a personal name or a word that some translations treat as a place but may not be
* not_a_proper_name: usually a common noun like "forest" that some translations treat as a proper name
* recursive: the only resolutions for this place point to places that also refer to this same place, creating an endless loop. A "recursive" value appears when it's the only option for a path; if other, valid, non-recursive resolutions are available for a given identification, recursive resolutions are omitted
* unknown_place: the location is unknown, such as the [Garden of Eden](https://www.openbible.info/geo/ancient/af3daeb/eden-1)

## GeoJSON

`geojson_file` points to a GeoJSON file containing a FeatureCollection of the geometry necessary to render all the location possibilities on a map.

Each feature in the FeatureCollection has an `id` property. If a resolution (in `identifications`) has a `geojson_roles` property, the `id` in `geojson_roles` corresponds to the `id` in the FeatureCollection.

## Linked Data (`linked_data`)

This object links the place to available ontologies. Each sub-object contains an `id` or `url`, depending on the source.

The keys match ids in sources.jsonl. The equivalent `friendly_id` of the sources are:

* biblemapper: [Biblemapper.com](https://www.biblemapper.com/)
* dare: [Digital Atlas of the Roman Empire](https://imperium.ahlfeldt.se/)
* factbook: Faithlife Factbook at biblia.com
* openbible_2007: the 2007 version of this dataset
* pleiades: [Pleiades](https://pleiades.stoa.org/)
* tipnr: [Tyndale House StepBible](https://github.com/tyndale/STEPBible-Data)
* wikidata: a Wikidata item
* wikipedia: a Wikipedia article about this place (usually only when a Wikidata item isn't available)
* ubs: United Bible Societies at some point released an XML dataset that disambiguates names in the Bible, though I can't locate it anywhere online

The `review` key indicates whether identification is based on a string match with the source; if it's "automatic" or "uncertain", then I didn't review it by hand.

The `modifier` key reflects a structured tag for the source:

* anchor: the url is part of a page, not the complete page
* main: the place is the main but not exclusive subject of the item
* nonunique_url: this url contains many small articles and shouldn't be used for linked-data applications
* modern: this item is about the modern location rather than the ancient place
* not: this item is not about the place and shouldn't be used for linked-data applications
* partial: this item is partially but not mainly about the ancient place
* partial,redirect: this Wikipedia article redirects to another Wikipedia article, which is partially about the ancient place
* redirect: this Wikipedia article redirects to another one (usually an anchor)

For example:

```json
"factbook": {
  "url": "https://biblia.com/factbook/Adadah",
  "review": "automatic"
},
"openbible_2007": {
  "id": "Adadah"
},
"tipnr": {
  "id": "Adadah@Jos.15.22"
},
"ubs": {
  "id": "ot ID_2137"
},
"wikipedia_article": {
  "modifier": "redirect",
  "url": "https://en.wikipedia.org/wiki/Adadah"
}
```

## Media (`media`)

About 50 regions have a `media` object containing a `thumbnail` object. These regions are defined by a specific location, but the thumbnail for that location doesn't necessarily express the character of the region. For example, [Gilead 1](https://www.openbible.info/geo/ancient/ae73b90/gilead-1) is defined in this dataset as a region around [Tell edh Dhahab esh Sherqiyeh](https://www.openbible.info/geo/modern/m222ca7/tell-edh-dhahab-esh-sherqiyeh), but a thumbnail of hills in the region of Gilead serves as a better thumbnail than one of just the tell.

The structure of this object matches the `media.thumbnail` object found in the modern.jsonl file.

## Modern associations (`modern_associations`)

This object summarizes all the locations enumerated in `identifications` that are associated with the place. Each key is a modern id. `identification_ids` is an array of identification positions, where each item is a tuple. The first value in the tuple refers to the position in the `identifications` array in the current ancient object (with 0 corresponding to the first item in `identifications`). The second value in the tuple refers to the position in the `resolutions` array inside the identification object. For example, a value of `[0, 1]` means that the identification corresponds to the first identification ("0") and the second resolution in that identification ("1").

`name` is the modern name. `url_slug` corresponds to the `url_slug` of the id in modern.jsonl.

`score` reflects an adjusted score to help determine the certainty of the identification, taking into account both the confidence of the identification and the confidence that the location reflects the identification. It uses the `score.time_total` score of the identification multiplied by the resolution's `best_time_score` divided by 1000, if the latter exists. If the identification has a `score.time_total` of 500 and the highest resolution `best_time_score` is 100, then the `score` would be 500 * (100 / 1000) = 50. If the identification has a `score.time_total` of 500 but no resolution `best_time_score`, then the `score` would be 500.

For example:

```json
"m7d8664":  {
  "identification_ids": [[0, 0]],
  "name": "Tel el Beida",
  "score": 349,
  "url_slug": "tel-el-beida"
},
"mec0b4d": {
  "identification_ids": [[1, 1]],
  "name": "Ain Kezbeh",
  "score": 52,
  "url_slug": "ain-kezbeh"
}
```

## Translation name counts (`translation_name_counts`)

This object summarizes the different spellings and translations that appear in the English Bible translations. The values reflect the number of total instances of the spelling across all verses in all translations.

```json
"translation_name_counts": {
  "Chisloth Tabor": 1,
  "Chisloth-tabor": 5,
  "Kislot-Tabor": 1,
  "Kisloth Tabor": 2,
  "Kisloth-tabor": 1
}
```

## String properties

* `comment`: an unstructured comment, potentially containing embedded XML or HTML tags
* `friendly_id`: a human readable name similar to the openbible_2007 dataset id, such as "[Aroer 2](https://www.openbible.info/geo/ancient/af74969/aroer-2)". The `friendly_id` is unique within ancient.jsonl but not necessarily unique in the whole dataset. (For example, there is a "Jerusalem" ancient place and a "Jerusalem" modern location.)
* `id`: a seven-digit string starting with "a" that serves as a unique identifier in the dataset
* `preceding_article`: "the" if the place should be preceded by the word "the" in English sentences (e.g., the Areopagus, the Valley of Hinnom); otherwise it's an empty string
* `type`: describes the kind of place, e.g., "body of water" or "settlement"
* `url_slug`: a non-unique, ASCII, lowercase representation of the name suitable for use in a url

## Verses (`verses`)

This array contains a complete list of the Bible verses where this place occurs.

### Verse identifiers

The `usx` and `osis` keys contain verse identifiers in their respective formats. The `readable` key is consistent with the verse reference from the openbible_2007 dataset.

The `sort` key is an eight-digit string that lexically sorts in canonical order (BBCCCVVV where BB is the book number--01 for Genesis through 66 for Revelation--CCC is the three-digit chapter number, and VVV is the three-digit verse number). The `verses` array is sorted by `sort`.

`alternate_verses` appears when the versification differs from the ESV. For example, most translations place the word "Jerusalem" in [Acts 4:5](https://www.biblegateway.com/passage/?search=Acts+4%3A5-6&version=NIV;KJV), but two place it in Acts 4:6 instead. Each key is a translation, and the value is the target verse reference in OSIS format.

```json
"alternate_verses": {
  "kjv": "Acts.4.6",
  "nkjv": "Acts.4.6"
}
```

### Translations

`translations` is an array of translations that contain a reference to the place where the `instance_type` is one of:  name, combined, partial, people_group. If the name appears as a pronoun in a particular translation, for example, the translation doesn't appear in this list.

The possible translations are "csb", "esv", "kjv", "leb", "nasb", "net", "niv", "nkjv", "nlt", and "nrsv".

`alternate_roots` is an object that describes alternate names that refer to a different place from other translations. For example, [Ezekiel 27:16](https://www.biblegateway.com/passage/?search=Ezekiel+27%3A16&version=ESV;NRSV) has "Syria" or "Aram" (which are synonyms) in most translations and "Edom" (which is a different place) in three translations; the `alternate_roots` looks like the following (where "a2735ff" is the ID for Edom and "3" is the number of translations that include Edom in the text):

```json
"alternate_roots": {
  "a2735ff": 3
}
```

The possible keys for `instance_types` are the following. The value is the number of translations that have an instance matching the type.

* name: a proper name
* combined: combines two names that appear separately in some translations; for example, in [Numbers 27:14](https://www.biblegateway.com/passage/?search=Numbers+27%3A14&version=NIV;ESV), the NIV translates "Meribah Kadesh" while the ESV translates "Meribah of Kadesh." There are two entries in the dataset, one for "Meribah" and one for "Kadesh"; the NIV "Meribah Kadesh" appears in both with a "combined" `type`
* common_noun: not a proper noun; for example, "forest"
* helper: a non-noun such as a pronoun
* no_translation: the place does not appear in the translation
* partial: part of a phrase that is partially a proper name. For example, [Joshua 16:1](https://www.biblegateway.com/passage/?search=Joshua+16%3A1&version=NIV;CSB) NIV contains the phrase "springs of Jericho", where only Jericho is a proper name. In such cases, an `alternate_roots` object (described below) may appear that points to the proper name. In the CSB, this phrase is translated as "Waters of Jericho", which is why the whole phrase appears in the dataset rather than just "Jericho"
* people_group: usually the inhabitants of a place. For example, [Acts 6:9](https://www.biblegateway.com/passage/?search=Acts+6%3A9&version=NIV;ESV) in the NIV has the place name "Alexandria", while the ESV has the people group "Alexandrians" 
* person: this translation treats the place name as a person name. For example, in [2 Samuel 21:16](https://www.biblegateway.com/passage/?search=2+Samuel+21%3A16&version=NIV;LEB), the LEB has "Nob" as a place, while the NIV has "Ishbi-Benob" as a person

# Modern locations (modern.jsonl)

## Accuracy claims (`accuracy_claims`) and precision claims (`precision_claims`)

These two properties are an array of strings containing my notes on difficulties or supporting evidence for identifying the general coordinates (`accuracy_claims`) or the precise point (`precision_claims`) of a modern location. While `accuracy_claims` may have multiple claims, `precision_claims` always only has one claim.

Here's an example accuracy claim:

> `<source id="s876c69" article="Baal-perazim">`International Standard Bible Encyclopedia (1979) (Baal-perazim)`</source>`: along the "valley running southwest between `<ancient id="a15257a">`Jerusalem`</ancient>` and `<modern id="m6beb29">`Mar Elias`</modern>`"

Here's an example precision claim:

> these coordinates match the Ain Sareh on `<a data-source="s8b7b31" href="https://palopenmaps.org/view/-/@31.541919,35.098819">`Palestine Open Maps`</a>` rather than the other Ein Sara just to the north

You can see embedded XML (`<ancient>`, `<modern>`, and `<source>`) and HTML (in this case, `<a>`) tags.

The `@id` attribute of the XML tags corresponds to the ancient, modern, or source id in this dataset. A `<source>` tag may have optional `@article`, `@map`, `@page`, `@table`, or `@url` attributes, with `@url` pointing to a specific part of the source (such as a certain page on Google Books) rather than to the url of the source as a whole.

In HTML, a `@data-source` attribute corresponds to the source id in this dataset. You can always just remove the XML tags for display in an HTML environment, or you could transform them into `<a>` links. Aside from these XML tags, the strings represent a valid HTML fragment.


## Ancient associations (`ancient_associations`)

This object summarizes all the places that possibly correspond to the location. Each key is an ancient id. `name` is the ancient name. `score` reflects a score to help determine the certainty of the identification.

For example:

```json
"a3498b5": {
  "name": "Mezahab",
  "score": 168
},
"afcb77d": {
  "name": "Dizahab",
  "score": 232
}
```

## Coordinates sources (`coordinates_source`)

This object describes the source of the coordinates.

```json
"geometry_id": "gec68e3",
"id": "Q246590",
"type": "wikidata",
"url": "https://www.wikidata.org/wiki/Q246590"
```

For example, this object is saying the the coordinates derive from [Wikidata item Q246590](https://www.wikidata.org/wiki/Q246590). A `url` appears if one is available.

## Media (`media`)

Every location has a `media` object containing at least a `thumbnail` object with a photo related to the location. About 5/8 of locations have a photo taken by a person (from Wikimedia Commons), with the remaining having a 10-meter-per-pixel satellite photo.

The `thumbnail` object has:

* `credit`: attribution for the source of the image
* `credit_url`: a url to use for attribution; for Wikimedia images, the url is the page on Wikimedia Commons
* `description`: a description of the image suitable for use in an `<img alt="">` attribute once any `<modern>` or `<ancient>` inline tags are removed
* `file`: the 512x512-pixel filename
* `image_id`: the id in image.jsonl that corresponds to this image, containing more metadata
* `quality` may appear with a value of "low" if, in my subjective opinion, the image doesn't represent the subject well (e.g., the image is primarily of something else with the subject in the background, or the image is of an artifact found at the location)
* `role` appears with a value of "satellite" if the image is a satellite photo of the site

In attributing the image in interactive environments, I recommend using `credit` and wrapping it in a link to `credit_url` if the latter exists.

There may also be an `alternate`, `google`, or `near` array containing additional image objects. `alternate` contains additional free images considered as thumbnails; these images are of varying quality. `google` generally has a Google Streetview image of the location, and occasionally a Google Place. `near` contains images taken near, but not of, the location that may be useful in establishing the general character of the area. Only `thumbnail` has a file as part of this project.

In an object in the `near` array, `proximity_meters` indicates approximately how close the camera is to the location.

```json
"alternate": [
   {
     "description": "panorama of <modern id=\"mcd7d18\">Tel Zeton</modern>",
     "image_id": "i5df073"
   }
],
"google": [
    {
      "description": "Google Streetview of <modern id=\"mcd7d18\">Tel Zeton</modern>",
      "image_id": "ibdb01f",
      "role": "google_streetview"
    }
],
"thumbnail": {
  "credit": "Dr. Avishai Teicher",
  "credit_url": "https://commons.wikimedia.org/wiki/File:PikiWiki_Israel_35304_Olive_hill_park_Tel_Abu_Zeitun_Bnei_Brak.JPG",
  "description": "panorama of <modern id=\"mcd7d18\">Tel Zeton</modern>",
  "file": "mcd7d18.i9f7bf5.jpg",
  "image_id": "i9f7bf5"
}

```

## Names (`names`)

This array contains objects describing some possible names of the location. The first name in the array is the same as `friendly_id`. The other values are not necessarily unique in the dataset. The `type` indicates whether the name is an "ancient" name for the location or a "modern" one. The `typo_for` indicates that the name is a typo but appears in a print book (in print, you'd indicate it with "sic"). Most variants simply reflect alternate modern spellings. In the below example, Alsi is an ancient name, and Alsiya is a modern one. The `url_slug` is a representation of the name suitable for use in a url. A `sentence_name`, if it appears, includes appropriate hyphenation and capitalization for use in the middle of a sentence.

```json
{
  "name": "Alsi",
  "type": "ancient",
  "url_slug": "alsi"
},
{
  "name": "Alsiya",
  "type": "modern",
  "url_slug": "alsiya"
}
```

## Precision (`precision`)

This object describes my estimate of how close the coordinates are to the intended location. The `description` is the raw expression, with `type` and `meters` programmatically derived from it. The below object indicates that my sources identified an ancient place with a modern settlement, and I have somewhat arbitrarily decided that in such cases the point is within 250 meters of the original place.

```json
"meters": 250,
"type": "settlement",
"description": "point in modern settlement"
```

## Root (`root`)

A few locations are defined in relation to another location, either when I'm not sure that two locations are identical or in defining a region. This `id` points to the source. The `modifier` indicates whether it's a region (if it's ">").

```json
"id": "m0ef9e3",
"modifier": ">",
"source": "modern"
```

## Secondary sources (`secondary_sources`)

This array contains objects in the same format as `coordinates_source`. They provide additional support for the location, though secondary sources don't necessarily represent exactly the same location. They may also come with their own geometry; particularly for rivers you'll find "osm" or "osm_group" types that expand the geometry beyond a point. An "osm" type links to a way or relation at [OpenStreetMap.org](https://www.openstreetmap.org/) and doesn't necessarily have a `geometry_id`, while an "osm_group" is a combination of several OSM ways--essentially a custom relation just for this dataset--and does have a `geometry_id`.

Other properties that may exist in a source are:

* `article`: the article name in the book
* `book_id`: an identifier when the `type` is a "known_book"
* `comment`: a free-text comment
* `data_url`: a url containing a structured representation of the data
* `georeference_id`: the id of a georeferenced map at an external source
* `georeference_url` the url of a georeferenced map at an external source
* `group`: when the `type` is "osm_group", this array contains objects that enumerate the OpenStreetMap urls that compose the group
* `id`: the id used by the source to uniquely identify this location
* `label`: the actual text that appears in the source
* `local_geometry_id`: the geometry id containing local geometry (for a settlement as opposed to a region); typically this geometry reflects the boundary of an archaeological site
* `map`: an identifier to locate the map in the source (generally something like "1-1")
* `osm_version`: the version of the OpenStreetMap node, way, or relation that the data is based on (OSM may have updated the geometry for this object or even deleted it; by adding "/history" to the end of the `url`, you can access the version in question)
* `page`: the page number in the source that supports the location
* `plate`: the plate (image) number in the source that supports the location
* `type`: a structured identifier indicating the source of the data
* `until`: an OpenStreetMap node id indicating when to stop the path
* `url`: a supporting url
* `url_id`: MEGAJordan sites have both an internal MEGAJordan number (in `id`) and this id that's addressable in the url
* `wiki_url`: Amud Anan sites have both a `url` for the map and this property for the wiki content in Hebrew
* `x` and `y`: the coordinates in the original geographic reference system, usually the Palestine 1923 Grid ([EPSG:28191](https://epsg.io/28191))

## String properties

* `class`: whether the location is "human" (e.g., a settlement), "natural" (e.g., a river), "probability" (the place is a point in the region, not the region itself), or "region"; the `class` doesn't necessarily match the `class` of the ancient place resolution
* `custom_lonlat`: when the coordinates are derived from a commercial source, this value provides a nearby but independently created longitude, latitude pair
* `epsg_28191`: coordinates expressed in terms of [EPSG:28191](https://epsg.io/28191), the Palestine 1923 Grid (or Map Reference) typically used in Bible atlases. Every coordinate is a pair of six-digit numbers (first the x coordinate and then the y); coordinates below 0 are expressed as negative numbers, unlike Bible atlases, which often subtract from 1,000,000 (e.g., 120000/-050000 in this dataset would be expressed in a Bible atlas as 120000/950000 or 120/950 depending on the precision). Only points that fall within the bounds of this coordinate system have this property. These coordinates are a straight conversion from the latitude and longitude and don't take into account the `precision` property
* `friendly_id`: unique within this file (but may also exist in ancient.jsonl), it's the same as the first item in `names`
* `geometry`: whether the geometry defining the location is a "path", "point", or "polygon"
* `geometry_id`: a geometry id for the location
* `id`: a seven-digit string starting with "m" that's unique in this dataset
* `local_geometry_id`: a geometry id when a point type (e.g., a settlement) has polygon geometry available from OpenStreetMap (for example, [Tel Beer Sheva](https://www.openbible.info/geo/modern/m392188/tel-beer-sheva) has a polygon indicating the extent of the archaeological site)
* `lonlat`: a comma-separated string indicating a longitude, latitude coordinate
* `preceding_article`: "the" if the location should be preceded by the word "the" in English sentences (e.g., the Jordan River, the Mediterranean Sea); otherwise it's an empty string
* `precise_geometry_id`: a geometry id with data from OpenStreetMap
* `type`: self-explanatory aside from "probability center n-s" and "probability center radial". In the first, the highest probability of finding the location is along the centerline of the polygon from the north to the south, with decreasing probability as you move away from the centerline. In the second, the highest probability is near the center of the region and diminishes as you approach the boundary of the polygon

# Geometry (geometry.jsonl)

This file enumerates non-point geometry.

## Isobands (`/isobands_geojson_file`)

For some regions, the exact boundaries aren't known. I've collected labels and boundaries from dozens of reference works and plotted them, with the idea that you can use this aggregate information to decide where you want to put your label.

`isobands_geojson_file` points to a file (in the "data/geometry" file path) that contains a GeoJSON MultiPolygon of up to 9 overlapping polygons that indicate confidence that the region contains the polygon. The `min_confidence` and `max_confidence` reflect the percentage confidence range for the polygons, with the first polygon in the MultiPolygon having the `min_confidence` and the last having the `max_confidence`. The value of `max_confidence` caps at 90, which reflects a 90% confidence that the biblical region contains the polygon. A 90% confidence indicates that at least 10 independent sources support that polygon. A lower-confidence polygon always completely encloses a higher-confidence polygon.

## Coordinates

When the `type` is "path" or "polygon", `geojson_file` and `simplified_geojson_file` may appear (they won't if `isobands_geojson_file` exists). `geojson_file` contains the full geometry; if the data is from OpenStreetMap, the number of points can stretch into the thousands, in which case `simplified_geojson_file` simplifies the number of points to about 100 to make display easier. All files appear in the "data/geometry" path in the repository.

* `geojson_file`: a GeoJSON "Feature" containing coordinates for the complete "Polygon" or "LineString"
* `simplified_geojson_file`: a GeoJSON "Feature" containing coordinates for the simplified "Polygon" or "LineString"

If the `type` is "polygon", the starting and ending value is identical, and the point order is counterclockwise. OSM data only includes the "outside" or "perimeter" relation, which means that the data doesn't have any holes--for example, if there are islands in a body of water, `geojson_file` encloses them completely. You may want to investigate the original data at OSM if you're looking to represent a feature with holes.

## Suggested (`/suggested`)

The geometry object may also contain a `suggested` object with up to three keys:

* `rough_boundary` has my subjective interpretation of possible (approximate) boundaries; for modern features (such as the island of [Crete](https://www.openbible.info/geo/modern/mf93341/crete)), the boundary reflects a bounding area in which you would likely place a label
* `label_line` consists of two points forming a line segment along which you could potentially place a label; it follows the contours of the land and is often horizontal
* `label_line_horizontal` is like `label_line` but is horizontal (same starting and ending latitude) rather than following the land

## Geometry types

The `geometry` key can have one of the following values:

* `rough_boundary`: an approximate boundary for the region
* `isobands`: a collection of polygons indicating the confidence that each polygon is in the region
* `path`: points defining a series of line segments
* `polygon`: a closed polygon (the last point is identical to the first point)
* `probability`: not a region, but the place in the Bible likely was somewhere in the polygon provided; the `modifier` identifies the most-likely point in the region
* `rough_path`: an approximate path

# Images (image.jsonl)

Every location has a 512x512-pixel thumbnail image available under a permissive license (usually [Creative Commons](https://creativecommons.org/) or similar). About 1,000 have a thumbnail sourced from photographers ([Wikimedia Commons](https://commons.wikimedia.org/wiki/Main_Page)); the remaining have satellite photos of the area at a 10-meter-per-pixel resolution (i.e., each image covers 5,120x5,120 meters).

The satellite photos are derived from the European [Sentinel-2](https://en.wikipedia.org/wiki/Sentinel-2) satellite program and have a permissive license--essentially an attribution license.

This file also contains metadata on images not included in the project, such as copyrighted images that you could potentially license for your own project. It has metadata for images from Google Streetview and Google Places and a few from [Bible Places](https://www.bibleplaces.com/) and [Holy Land Photos](https://holylandphotos.org/).

This object can contain the following non-object properties:

* `author`: the author of the image for attribution
* `color`: whether the original image is in "color", "black_and_white", or "colorized" (originally black and white but with color added before uploading to Wikimedia Commons)
* `credit`: a credit string to use for attribution. This string is usually identical to the `author`. It doesn't contain any html
* `credit_url`: the url to use for attribution. This url is typically the overview page for the image on Wikimedia Commons
* `file_url`: the url of the original image on Wikimedia Commons
* `height`: the pixel height of the original image
* `id`: the image id referenced in the modern and ancient files
* `meters_per_pixel`: for satellite images, the resolution of the image; a value of 10 means that each pixel represents approximately 10x10 meters
* `thumbnail_url_pattern`: replace the "####" with the desired width to have Wikimedia Commons produce a thumbnail
* `url`: identical to the `credit_url`
* `width`: the pixel width of the original image

The `license` attribute indicates the type of license for the image; for the most part, the licenses are Creative Commons. The non-Creative-Commons values are:

* attribution: requires attribution on Wikimedia Commons
* copyright: the image is copyrighted and can't be used without permission from the rightsholder
* FAL: the [Free Art License](https://en.wikipedia.org/wiki/Free_Art_License)
* GFDL: the [GNU Free Documentation License](https://en.wikipedia.org/wiki/GNU_Free_Documentation_License)
* GPL: the [GNU General Public License](https://en.wikipedia.org/wiki/GNU_General_Public_License), which is an unusual license to use for an image
* OGL: the [Open Government License](https://en.wikipedia.org/wiki/Open_Government_Licence)
* PD: public domain
* sentinel: the [Sentinel license](https://sentinel.esa.int/documents/247904/690755/Sentinel_Data_Legal_Notice)

The `descriptions` object has a modern or ancient id as the key and a string for a description when used as a thumbnail. Because the same image can serve for multiple locations, the description can vary.

```json
"descriptions": {
  "m4c3dce": "streetscape of <modern id=\"m4c3dce\">Ismailia</modern>",
  "maaba23": "streetscape of Ismalia in the region <modern id=\"maaba23\">between Maghfar and Lake Timsah</modern>"
},
```

## Thumbnails (`/thumbnails`)

The `thumbnails` object, like `descriptions`, has a modern or ancient id as the key. Each item here reflects a file I've processed into a 512x512-pixel image for consistency and to be legible at small sizes. The processed thumbnail images are available at https://a.openbible.info/geo/thumbnails.zip (180 MB zip file).

The `file` key indicates the file name in the zip file.

The `edits` array summarizes the edits I made to the original file:

* color: adjusted colors in Photoshop, generally brightness, contrast, levels, saturation, tone, and vibrance. Some images have minimal editing; some have major (e.g., changing a night scene to a day scene)
* colorize: applied Photoshop's colorize filter to a black-and-white image and adjusted the resulting output to look somewhat natural
* content-aware fill: used Photoshop's content-aware fill feature to remove elements (such as people, vehicles, trash, or power lines) or to extend the background to provide a more-suitable composition
* crop: cropped the image (or a subset of it) into a square
* rotate: rotated the image so that the horizon is more level
* super-resolution: applied Photoshop's super-resolution algorithm to enlarge the image before other edits

The `placeholder` key provides a hex representation of a CSS vertical linear-gradient background color that reflects the image, usable for placeholders. For example, the value "#85b7ec,#aaaa90,#746e56" could be used with: `background: linear-gradient(#85b7ec,#aaaa90,#746e56)`.

A `description` key may exist if the thumbnail requires a different description from the original image. For example, the image object description for the below example is: "panorama including `<modern id="m549398">Louaize</modern>`, which is the smaller settlement beyond the gorge at the center". Because the thumbnail focuses on the area of interest in the larger image, it has a different description.

```json
"thumbnails": {
  "m549398": {
    "description": "panorama including <modern id=\"m549398\">Louaize</modern>, which is at center",
    "edits": [
      "color",
      "crop"
    ],
    "file": "m549398.i887af0.jpg"
    }
  }
```

# Sources (source.jsonl)

This file documents sources used in this project. The purpose is to enable you to track the source of a claim.

## URL properties

In many cases, multiple editions of a source make finding a "canonical" id difficult. In some cases, the link points to only part of a multi-volume set. The different sources often deliberately point to different editions of the same underlying text to make the dataset more robust and resistant to decay.

* `amazon_id` and `amazon_url` provide a link to the source on [Amazon](https://www.amazon.com/)
* `best_commentaries_book_id`, `best_commentaries_series_id`, and `best_commentaries_url` provide a link to a source on [Best Commentaries](https://www.bestcommentaries.com/). A source with these properties will have either a book id or a series id, not both.
* `google_books_id` and `google_books_url` provide a link to the source on [Google Books](https://books.google.com/)
* `logos_id`, `logos_resource_id`, and `logos_url` provide a link to the source on [Logos](https://www.logos.com/). The `logos_id` appears in the url, and the `logos_resource_id` is used in the app
* `olivetree_id` and `olivetree_url` provide a link to the source on [Olive Tree](https://www.olivetree.com/)
* `url` provides a link to the source
* `web_archive_url` provides a link to the source at the [Wayback Machine](https://web.archive.org/)
* `worldcat_id` and `worldcat_url` provide a link to the source on [WorldCat](https://www.worldcat.org/)

## Other properties

* `abbreviation`: an abbreviation (acronym) that you could use for longer names (e.g., "DAAHL" for "Digital Archaeological Atlas of the Holy Land")
* `contributors`: an array of contributor names, generally an author or an editor
* `display_name`: a friendly name, including a publication year where applicable
* `id`: an "s" followed by a six-digit hexadecimal number that's unique in the dataset
* `publisher`: sources with a `vote_count` have this property to indicate the publisher
* `type`: the source type (book, article, etc.)
* `vote_count`: a number, range 1-100, indicating the number of votes in the ancient data from this source; 100 means 100 or more votes
* `year`: the year of the source's publication. For a series, it's generally the publication date of the last book in the series
