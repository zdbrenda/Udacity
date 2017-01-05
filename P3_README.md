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

### Inconsistent street names
When wrangling with street names, I noticed that the street names in the "addr:street" tag of the osm file were inconsistent with our standard form.
Some samples are as follows:
- East 2nd (East 2nd Avenue)
- Denmanstreet (Denman Street)
- Shaughnessy St (Shaughnessy Street)
- E. Broadway (East Broadway)
- Boundary Rd.(Boundary Road)
- Howe St. Vancouver (Howe Street)
- Granville St #216 (Granville Street)
- Broadway W (West Broadway)

To deal with such inconsistency, I first printed out all the street types that are not in expected (a set that contains all expected street types ["Street", "Avenue", "Boulevard", "Drive", "Court", "Place", "Square", "Lane", "Road", "Trail", "Parkway", "Commons"]). It turned out there are many street types that are not in the set. Some such samples are listed as follows:
- Sennok Crescent
- Grant McConachie Way
- Kingsway
- West Mall
- Greer Venue
- Health Sciences Mall
- West Broadway
- Foreshore Walk
- Railspur Alley
- Lougheed Highway

Since these street names comply our expectations, I decided to keep them as they were. As I mentioned above, the inconsistent street names were inconsistent with our expected form in various ways. Therefore, I dealt with them separately in my update_street_name function:

```
def update_street_name(name, mapping):
    better_name=""
    
    # remove "#"
    if "#" in name:
        
        
        name=name.rsplit("#",1)[0][:-1]
	
     # change Broughton into Broughton Street   
    elif name=="Broughton":
        # A street name is missing "street"
        name="Broughton Street"
    elif "Steet" in name:
        # A spelling mistake
        name=name.replace("Steet","Street")
    elif "Mall" in name:
        name=name
    elif "South" in name:
        name=name
	#Keep Broadway
    elif "Broadway" in name:
        name=name
	#Change Denmanstreet into Denman Street
    elif name=="Denmanstreet":
        name="Denman Street"
    elif " " in name:
       # Vancouver is in street name
        if name.split(" ")[-1]=="Vancouver":
            name=name.rsplit(" ",1)[0]
            
        elif name.split(" ")[1]=="2nd":
            name=name+" Avenue"
     	    
    if name.endswith("W"): # name ends with W
        name="West "+name.rsplit(" ",1)[0]
    elif name.endswith("E"):  #ends with E
        name="East "+name.rsplit(" ",1)[0]
	#starts with W
    if name.startswith("W ") or name.startswith("W. "):
        name="West "+name.rsplit(" ",1)[1]
	#starts with E
    elif name.startswith("E ") or name.startswith("E. "):
        name="East "+name.rsplit(" ",1)[1]
	#Change Jervis into Jervis Street
    if name=="Jervis":
        name=name+ " Street"
	
	
    # if the street name ends with one of the keys in the mapping
    if name[-3:] in mapping.keys(): 
        better_name=mapping[name[-3:]]
        name=name[:-3]+better_name
    # if the street name ends with one of the keys in the mapping
    elif name[-2:] in mapping.keys():
        better_name=mapping[name[-2:]]
        name=name[:-2]+better_name
    elif name.endswith("East"): #put "East" at the beginning of the street names
        name="East "+name.rsplit(" ",1)[0] 
    elif name.endswith("West"):
        name="West "+name.rsplit(" ",1)[0]

    return name

```
