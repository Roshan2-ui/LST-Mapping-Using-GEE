# LST-Mapping-Using-GEE
The is a Google Earth Engine code generates LST with two different consideration:

i) v2 (em2): This version simplifies the emissivity calculation by using a constant value (0.986) for areas with vegetation. This approach may not account for variations in vegetation density as effectively as the normal version.

ii) (em): This version uses a specific emissivity calculation based on the proportion of vegetation (Pv) and the NDVI values. It adjusts the emissivity based on the vegetation cover, which can lead to more accurate temperature readings in areas with varying vegetation.
 
consdering different surface types such as water, ground (Soil), vegetation using sentinal images.

Steps for Executing Code in GEE

Step 1: While using this piece of code please add your ROI to GEE reprository and rename it to study.

Step 2: Select appropriate Sentinal dataset for your ROI.

Step 3: Provide your google drive folder link to export the generated LST map.

Step 4: Execute Code
