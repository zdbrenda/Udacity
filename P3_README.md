# P3 Report

## OpenStreetMap Data Case Study


### Map Area
Vancouver, British Columbia, Canada
https://www.openstreetmap.org/relation/1852574
This is the city where I am living right now. I have been living in Vancouver for over eight months so far, and I came to fall in love with it, so I’d like to know the city better by doing the project.


### Problems Encountered in the Map

After spending time looking at a small sample size of the Vancouver area, I noticed several problems with the data:
- Inconsistent street names in “addr:street” tags (such as: Yew St, Howe St. Vancouver, Seymour St., Boundary Rd.)
- Missing detailed address information about many places spotted in the dataset,  including street name and house number, only longitude and latitude information was provided.
- Some places are named as “school”s, but they didn’t have an “amenity: school” tag as many other schools are. 

```XML
<node id="3328040320" lat="49.2613559" lon="-123.2463657" version="1" timestamp="2015-02-03T07:33:28Z" changeset="28583278" uid="503905" user="eone">
		<tag k="name" v="Eaton Arrowsmith School"/>
		<tag k="building" v="school"/>
		<tag k="addr:city" v="Vancouver"/>
		<tag k="addr:street" v="Agronomy Road"/>
		<tag k="addr:postcode" v="V6T1Z3"/>
		<tag k="addr:province" v="BC"/>
		<tag k="addr:housenumber" v="6190"/>
		
		
		</node>
```
