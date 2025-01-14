# Cloud Hosted MongoDB Database

We are migrating our Ingame Data to JSON formatted data. To Host and access this data effectively we can use MongoDB.
- <https://cloud.mongodb.com/>

## Python Example

Python integrates very well with mongodb. Feels like you are directly accessing the objects

```python
import pymongo
import dns # required for connecting with SRV
import pprint

myuser="rudi"
mypassword="mypassword"

#, server_api=ServerApi('1')
myclient = pymongo.MongoClient("mongodb+srv://"+myuser+":"+mypassword+"@chia-stuff.g7ivxgh.mongodb.net/?retryWrites=true&w=majority")
db = myclient.test
print(db)

mydb = myclient.mydatabase
mycol = mydb.customers
mydict = { "name": "Herbert", "address": "Trafalgar Square" }
result = mycol.insert_one(mydict)

doc=mycol.find_one({"name": "John"})

pprint.pprint(doc)
```

## HTTP API Example

Powershell MongDB driver doesn't work for me. But with MongDB Atlas i can easily access via HTTP API

### Powershell

```powershell

$h_data=@{
    "dataSource"="chia-stuff"
    "database"="mydatabase"
    "collection"="customers"
    "projection"= @{
        "name"="John"
    }
}

$h_headers=@{
#    "Content-Type"="application/json"
    "Access-Control-Request-Headers"="*"
    "api-key"="yourApiKey"
}

$result=Invoke-RestMethod -Method Post -Uri "https://data.mongodb-api.com/app/data-tmopi/endpoint/data/v1/action/findOne" -Headers $h_headers -Body (ConvertTo-Json $h_data) -ContentType "application/json"
$result.document
```

### curl

Even with Bash Scripts we can directly access with curl

```bash
curl --location --request POST 'https://data.mongodb-api.com/app/data-tmopi/endpoint/data/v1/action/findOne' \
--header 'Content-Type: application/json' \
--header 'Access-Control-Request-Headers: *' \
--header 'api-key: yourApiKey' \
--data-raw '{
    "collection":"customers",
    "database":"mydatabase",
    "dataSource":"chia-stuff",
    "projection": {"name": "John"}
}'
```

## Importing Chiania Data

Is easy as fuck

```python
import pymongo
import dns # required for connecting with SRV
import pprint
import config.secrets
import json

# Your Internet IP must be added to MongoDB Atlas Network Access IPs

myuser=config.secrets.dbuser
mypassword=config.secrets.dbpass

#, server_api=ServerApi('1')
dbclient = pymongo.MongoClient("mongodb+srv://"+myuser+":"+mypassword+"@chia-stuff.g7ivxgh.mongodb.net/?retryWrites=true&w=majority")

#db is "chiania"
db = dbclient.chiania
#collection is locations
col = db.locations

#db and collection are created as needed

location_list = []

#Loading Locations JSON File
with open("src/data/locations.json","r") as file:
    location_data=json.loads(file.read())

#Each Key in Location is one "document"
for key in location_data:
    #Adding Key as "name" in document
    location_data[key]["name"]=key
    #Inserting one document for one location key
    col.insert_one(location_data[key])
```

## Selecting Data

Then when you want to select data it works like this

```python
dbclient = pymongo.MongoClient("mongodb+srv://"+myuser+":"+mypassword+"@chia-stuff.g7ivxgh.mongodb.net/?retryWrites=true&w=majority")
#db is "chiania"
db = dbclient.chiania
#collection is locations
col = db.locations

location=col.find_one({"name":"Kingdom Street"})

pprint.pprint(location)
```


## Powershell Examples

### Powershell on Linux

#### .Net Driver

This is what i did on **Ubuntu 20.04**. It **may** not necessary to install these mono (.Net Libraries) because in the end i decided to give the powershell Module [MDBC](https://github.com/nightroman/Mdbc) another Try

```bash
#.Net and Nuget for Linux
sudo apt install nuget
```

I need current version of "mono" (.Net Library on Linux)

- <https://www.mono-project.com/download/stable/>

```bash
sudo apt install gnupg ca-certificates
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
echo "deb https://download.mono-project.com/repo/ubuntu stable-focal main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list
sudo apt update

sudo apt install mono-complete 

#Somehow Nuget had a broken Config. Could be regenerated by removing the current config
cd ~/.config
mv NuGet NuGet.old
```

Then also nuget should be current version (instead of way too old Ubuntu integrated version)

```text
rudi@hpws-ubuntu:~/.local/share/powershell/Modules$ nuget
NuGet Version: 6.2.1.2
```

Then Finally i was able to install MogoDB Driver for .Net

```bash
nuget install MongoDB.Driver
```

#### MDBC

Its way easier with MDBC

```powershell
Install-Module MDBC
Import-Module MDBC
```

```powershell
#$Global:ChiaShell
#$srcDir="~/git/chiania/src"
$configDir="~/git/chiania/config"
$itemsDir="~/git/chiania/docs/items"
$logPath="~/Documents/chiania_items.log"

$scriptConfigFile=$configDir + "/" + "scriptConfig.json"
$h_config=Get-Content -Path $scriptConfigFile -Encoding UTF8 | ConvertFrom-Json -Depth 10


$Server=$h_config.mongodb.server
$Database=$h_config.mongodb.database
$User=$h_config.mongodb.user
$Password=$h_config.mongodb.password

$connectionString="mongodb+srv://" + $User + ":" + $Password + "@" + $Server + "/" + $Database + "?retryWrites=true&w=majority"

Connect-Mdbc -ConnectionString $connectionString -DatabaseName "chia-stuff" -CollectionName "test"

@{_id = 1; value = 42}, @{_id = 2; value = 3.14} | Add-MdbcData

Get-MdbcData -As PS | Format-Table
```