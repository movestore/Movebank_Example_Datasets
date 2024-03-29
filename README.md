# Movebank_Example_Datasets

We provide here a list of studies for testing MoveApps Apps and other applications using data from Movebank. Our intent is to illustrate potential undesired results to watch out for with certain dataset characteristics, so you can ensure your code will accurately read and process data. All examples refer to publicly-available studies.

## Working with deployed data.
**Recommendation:** Analysis should rely on the [individual local identifier](http://vocab.nerc.ac.uk/collection/MVB/current/MVB000016/) (Animal ID) rather than [tag local identifier](http://vocab.nerc.ac.uk/collection/MVB/current/MVB000181/) (Tag ID) or internal database identifiers.

**Why this matters**  
* Tags can be redeployed on multiple animals, and animals can have multiple tags deployed on them. Therefore, relying on Tag IDs can lead to incorrect sample sizes. 
* Events assigned to a Tag ID but not an Animal ID represent data that the owner has not linked to an animal, typically because they were collected before or after the tag was deployed on an animal. These should not be interpreted as animal movements or behavior!
* It will be easier to communicate with data owners using these animal IDs, which they have assigned to the data.

**Examples**  
Study [MPIAB Argos white stork tracking (1991-2017)](https://www.movebank.org/cms/webapp?gwt_fragment=page=studies,path=study7431347) ([Berthold et al., 2022](https://www.doi.org/10.5441/001/1.k29d81dh))  
To check: This study contains 126 tags, 214 deployments and 209 animals. After reading data, make sure the number of animals is 209! Make sure any output clearly displays the animal IDs. If you suspect that the Animal IDs have not been correctly defined, see instructions to [add](https://www.movebank.org/cms/movebank-content/upload-qc#add_deployments) or [fix](https://www.movebank.org/cms/movebank-content/upload-qc#fix_incorrect_deployments) the deployments in Movebank, or request that the owner do so.

Study [Deconstructing the flight paths of hippocampal-lesioned homing pigeons (data from Gagliardo et al. 2022)](https://www.movebank.org/cms/webapp?gwt_fragment=page%3Dstudies%2Cpath%3Dstudy2258814165), Animal [C048971](https://www.movebank.org/cms/webapp?gwt_fragment=page=studies,path=study2258814165+individual2258906357) ([Gagliardo et al., 2022](https://www.doi.org/10.5441/001/1.fs61hr7v))  
To check: This study contains results of homing experiments, and contains nine deployments for this animal. As shown here, these deployments represent multiple releases from different sites, involving the same animal and tag. Make sure the application understands this is 1 animal! This can also be used to test code intending to assess deployments or track segments separately.
GagliardoEtAl2022_study2258814165_animalC048971.png

## Working with locations estimated using multiple methods.
**Recommendation:** Consider how to treat data collected using different methods, indicated by the [sensor type](http://vocab.nerc.ac.uk/collection/MVB/current/MVB000170/) of the event.  

**Why this matters**  
* Within a dataset, tracking data may be collected using multiple sensor types, such as radio-telemetry and GPS. Individual animals within the dataset may have been tracked using one or more sensor types, consecutively or over the same time period.  
* These different sensor types commonly represent different sampling rates and location accuracy.  
* Filtering data to a single sensor type (e.g., GPS only) can allow consistent treatment of the data, but can lead to reduced sample sizes or data gaps. Depending on the use case, it may be preferable to apply different procedures to different sensor types, and/or to include data from a lower-accuracy sensor type only for periods with gaps in the higher-accuracy sensor type.

**Examples**  
Study [MPIAB Argos white stork tracking (1991-2017)](https://www.movebank.org/cms/webapp?gwt_fragment=page=studies,path=study7431347) ([Berthold et al., 2022](https://www.doi.org/10.5441/001/1.k29d81dh))  
To check: This study contains a combination of GPS and Argos data, some birds with just one and some with both.

## Working with milliseconds.
**Recommendation:** Code should retain milliseconds stored in [timestamps](http://vocab.nerc.ac.uk/collection/MVB/current/MVB000200/) when reading and processing data. If reduced precision is desired or required, consider carefully how to treat milliseconds.

**Why this matters**  
* Millisecond precision can be important for those datasets that contain it.
* Reduced precision can lead to different results depending on whether values are truncated or rounded, and on how duplicate timestamp values are treated.
* Linking processed data back to another version of the same dataset may rely on the timestamps remaining identical. 

**Examples**  
Study [Straw-colored fruit bats (Eidolon helvum) in Africa 2009-2014](https://www.movebank.org/cms/webapp?gwt_fragment=page=studies,path=study404939825) ([Scharf et al., 2019](https://www.doi.org/10.5441/001/1.k8n02jn8))  
To check: This study contains milliseconds, which should be present after reading in the data. If intentionally reducing precision, this dataset contains lots of ".998" values that should ideally be rounded up.

## Reading identifiers.
**Recommendation:** Do not read event, animal and tag identifiers as numeric values. Instead they can be read as factors, character strings or (for long values) integer64.  

**Why this matters**  
* Each event record in Movebank has an [event id](http://vocab.nerc.ac.uk/collection/MVB/current/MVB000103/) which may be used as a key to link data across tables and files. If these values are read as numeric, they will sometimes be converted to exponents, which can break subsequent code or attempts to join data.  
* User-assigned Animal and Tag IDs (see [above](#working-with-deployed-data)) are all numeric for some studies, but non-numeric characters will be present in others. Code that assumes values are numeric will break when it encounters these values.  

## Supporting live data streams.
**Recommendation:** For applications intended to work with data as they are being collected, provide options to select recent data.  

**Examples**
See Study [LifeTrack White Stork SW Germany](https://www.movebank.org/cms/webapp?gwt_fragment=page=studies,path=study21231406) ([Fiedler et al., 2019](https://www.doi.org/10.5441/001/1.ck04mn78)

This example has incoming data as of March 2023. *Note it is very large!* In most cases, applications working with incoming data will want to limit requests to newer data. This can be done by filtering using relevant fields in the [REST API](https://github.com/movebank/movebank-api-doc/blob/master/movebank-api.md) including timestamp or (when present) update_ts, provider_update_ts, argos_transmission_timestamp and transmission_timestamp.

## Working with non-location data.
**Recommendation:** Know how to identify data from non-location sensors, and how to process these data, if it is within the scope of your application. Be aware that there are completed studies without any location data.

**Why this matters** 
* Most data in Movebank is not location coordinates, but measurements from other bio-logging sensors such as acceleration.
* These sensor measurements may be indicated by the [sensor type](http://vocab.nerc.ac.uk/collection/MVB/current/MVB000170/), or may be included in the same events with location estimates. For example, if temperature is sampled on the same schedule as GPS fixes, researchers will typically store these values together. If acceleration is sampled on a different schedule from GPS, or if the tags are not collecting location estimates, they can be stored in separate tables.

**Examples**  
Study [LifeTrack White Stork Poland ECG](https://www.movebank.org/cms/webapp?gwt_fragment=page=studies,path=study25166516)  
To check: This study contains sensor types GPS, acceleration and heart-rate. Make sure your application should identify or access the acceleration and heart rate data, if they are of interest.

Study [Eastern whip-poor-will migrations (data from English et al. 2017)](https://www.movebank.org/cms/webapp?gwt_fragment=page=studies,path=study18072398) (English et al., 2017)  
To check: This study contains the sensor type solar-geolocator-raw (light-level recordings that can be analyzed with time to estimate location), but no location estimates or location sensors. Applications looking only for locations and location sensors could be configured to return a message that no location data exist for the study.

## Working with duplicate and near-duplicate data records.
**Recommendation:** Check for and address duplicates, considering your attributes of interest and potential quality differences in near-duplicate records. (In the [Movebank App](https://www.moveapps.org/apps/browser/8eeafaad-410e-440b-a105-94f6ff4109d8) duplicates must be addressed when accessing data, and [multiple options](https://github.com/movestore/movebank-download#documentation) are offered.) If you are the data owner, you can [exclude](https://www.movebank.org/cms/movebank-content/import-custom-tabular-data#finishing_the_data_import) duplicates when importing data, or [filter](https://www.movebank.org/cms/movebank-content/general-data-filters#duplicate_filter) duplicates present in your study.

**Why this matters**  
* Duplicates can be introduced to a dataset if the same data are added through multiple sources.
* What is a "duplicate" depends on your data attributes of interest. For data owners, it can be important to retain near-duplicate records: for example, a reduced or lower-quality data record may be transmitted via a live feed, and a more extended or higher-quality data record can sometimes be obtained later if the tag is retrieved from the field. This later version may contain more or different values from the original, and is added to the study later. In this case, the data owner typically does not want to exclude these records from the import because it contains the better-quality version of the record.
* In order to prevent data loss, Movebank does not overwrite or delete existing data records, although this can be done by the data owner.

**Examples**  
To do.

## Working with outliers.
**Recommendation:** Check for and remove outliers from your analysis, considering the accuracy necessary for your use of the data.

**Why this matters** 
* Outliers can be sent automatically through data feeds or included in raw data uploaded by users.
* Data owners commonly want to retain their full dataset, including outliers, as it can indicate behaviors or equipment reliability.
* While data owners can flag outliers in their studies, there are often lower-quality records that may be useful for some cases (like assessing large-scale migration) but inappropriate for others (like assessing local foraging behavior).

**Examples**  
To do.

## Working with partial data access.
**Recommendation:** Data owners control access to their studies in Movebank. If they have not provided permissions for a Movebank account or the public to download data, the account won't be able to download data for use in other applications. It is also possible for owners to use an embargo to allow the public to prohibit public access to more recent locations.  

**Why this matters** 
* Study and animal-level statistics provided by Movebank will typically represent the full study, but may not reflect what a user is allowed to download.

**Examples**  
To do.

## References
Berthold P, Kaatz C, Kaatz M, Querner U, van den Bossche W, Chernetsov N, Fiedler W, Wikelski M. 2022. Data from: Study "MPIAB Argos white stork tracking (1991-2022)". Movebank Data Repository. https://www.doi.org/10.5441/001/1.k29d81dh

English PA, Mills AM, Cadman MD, Heagy AE, Rand GJ, Green DJ, Nocera JJ. 2017. Data from: Tracking the migration of a nocturnal aerial insectivore in the Americas. Movebank Data Repository. https://www.doi.org/10.5441/001/1.66jq0844

Fiedler W, Flack A, Schäfle W, Keeves B, Quetting M, Eid B, Schmid H, Wikelski M. 2019. Data from: Study "LifeTrack White Stork SW Germany" (2013-2019). Movebank Data Repository. https://www.doi.org/10.5441/001/1.ck04mn78

Gagliardo A, Cioccarelli S, Giunchi D, Pollonara E, Colombo S, Casini G, Bingman VP. 2022. Data from: Deconstructing the flight paths of hippocampal-lesioned homing pigeons as they navigate near home offers insight into spatial perception and memory without a hippocampus. Movebank Data Repository. https://www.doi.org/10.5441/001/1.fs61hr7v

Scharf AK, Fahr J, Abedi-Lartey M, Safi K, Dechmann DKN, Wikelski M, O'Mara MT. 2019. Data from: Overall dynamic body acceleration in straw-colored fruit bats increases in headwinds but not with airspeed. Movebank Data Repository. https://www.doi.org/10.5441/001/1.k8n02jn8
