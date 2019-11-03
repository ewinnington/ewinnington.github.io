Title: Python - CSV handling
Published: 24/04/2012
Tags: [Migrated, Python] 
---

If you are going to process CSV's in Python, you must really use the csv module. It makes life very easy on you and has extensive possibilities. By default, It creates a dictionary containing the header rows as key (eg: "Header 1"), which makes processing a charm.
```python
import csv

coordinates\_file = open("data.csv", "rU")
coordinates\_data = csv.DictReader(coordinates\_file)

for coord in coordinates\_data:
    print (coord\["Header 1"\])

coordinates\_file.close()
```
In the open statement, the "rU" is for universal read, meaning python will figure out line endings. (\\r\\n or \\n or something other).

Another little trick, is once you have loaded a csv file, you can for example make a indexed array by indexing the input with one of the information in the row (assume here patrols is loaded with all the patrols, either from a DB or a CSV file):
```
patrols\_dict = {} #Init to empty
for patrol in patrols:
     patrols\_dict\[patrol\["PatrolID"\]\] = patrol

#And now we can access a patrol directly by it's patrol ID
#eg. patrols\_dict\["1000"\] to access patrol 1000.
```