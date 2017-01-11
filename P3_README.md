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

### Missing "amenity:school" tags

As mentioned in section Problems Encountered in the Map, several schools in the dataset have "schools" in their names, but they are missing "amenity:school" tags which are normally included for other schools. To deal with this, I decided to add "amenity:school" tags to such schools. I added the following code to add such tags:

```
 found_school=False
            # add "amenity:school" tags if it's missing from schools
            for i in range(len(tags)):
                tag_set.add(tags[i]["key"])
                if "name" in tags[i]["key"] and "School" in tags[i]["value"]:  
                    found_school=True
            if "amenity" not in tag_set and found_school:      
                new_tag={}
                new_tag["key"]="amenity"
                new_tag["value"]="school"
                new_tag["type"]="regular"
                new_tag["id"]=tags[0]["id"]
                tags.append(new_tag)
		
```

## Query result by count, descending

### Top 10 appearing amenities:
```
select value, count(*) as num 
from nodes_tags 
where key="amenity" 
group by value order by num 
desc limit 10;
```

The query result is:

```
bench,711
restaurant,676
bicycle_parking,385
cafe,375
fast_food,318
post_box,225
bank,131
toilets,98
bicycle_rental,90
drinking_water,83
```
### Top 10 appearing postcodes:

```
select value, count(value) 
from( select * from  nodes_tags union all select * from ways_tags)  
where type="addr" and key="postcode" 
group by value 
order by count(value) 
desc limit 10;
```
The query result is:

```
"V6S 0A9",72
V7L2A4,38
"V5M 0C4",26
"V5V 3A4",21
V6M1V1,18
V6M3A5,18
"V5R 5K6",17
"V6A 3T8",15
"V5R 5G9",14
"V6T 1Z4",14
```

### Top 10 Appearing Street names

```
select value, count(value) 
from( select * from  nodes_tags union all select * from ways_tags)  
where type="addr" and key="street" 
group by value 
order by count(value) 
desc limit 10;
```

The query result is: 
```
"East 4th Avenue",637
"West 4th Avenue",611
"East 5th Avenue",556
"Granville Street",499
"Davie Street",469
"East 6th Avenue",467
"East Keith Road",417
"East 3rd Avenue",368
"West Hastings Street",368
"Richards Street",331
```

### Top 10 contributing users
```
SELECT e.user, COUNT(*) as num
   ...> FROM (SELECT user FROM nodes UNION ALL SELECT user FROM ways) e
   ...> GROUP BY e.user
   ...> ORDER BY num DESC
   ...> LIMIT 10;
   
``` 
Query result:
```
keithonearth,656650
michael_moovelmaps,250738
still-a-worm,215433
treeniti2,164891
pdunn,88081
muratc3,81614
WBSKI,66345
rbrtwhite,47412
Siegbaert,47070
pnorman,43319
```

Top 20 popular cuisine

```
SELECT nodes_tags.value, COUNT(*) as num
   ...> FROM nodes_tags 
   ...>     JOIN (SELECT DISTINCT(id) FROM nodes_tags WHERE value='restaurant') i
   ...>     ON nodes_tags.id=i.id
   ...> WHERE nodes_tags.key='cuisine'
   ...> GROUP BY nodes_tags.value
   ...> ORDER BY num DESC limit 20;
   
 ```
 Query result:
``` 
japanese,100
chinese,88
sushi,62
vietnamese,40
italian,28
pizza,28
indian,26
mexican,22
asian,20
burger,18
greek,18
thai,18
american,16
malaysian,16
regional,16
french,14
breakfast,12
international,10
seafood,10
sandwich,8
```
Religion
```
SELECT nodes_tags.value, COUNT(*) as num
   ...> FROM nodes_tags 
   ...>     JOIN (SELECT DISTINCT(id) FROM nodes_tags WHERE value='place_of_worship') i
   ...>     ON nodes_tags.id=i.id
   ...> WHERE nodes_tags.key='religion'
   ...> GROUP BY nodes_tags.value
   ...> ORDER BY num DESC;
 ```
 Query result:
``` 
christian,36
buddhist,2
```
