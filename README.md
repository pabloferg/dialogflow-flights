# dialogflow-flights

![Screenshot](timecomparison.png)

# Get content: destination images and similar destinations

The main database we will use is stored in Firestore. We will load a [reference table](/aIrport_codes_200.csv) for 200 destinations, containing.

Each destination in the database has an image and a list of similar destinations. The image is an url where theimage is hosted. 

You could build this database with your own images and calculate the similarity based on users behaviour on your website, geographic proximity, themes (beach, ski, nature...) , etc. In this example we will use an already built database from [Nomadlist.com](https://nomadlist.com), a nice website with lots of info about places to visit. 

| airportCode | airportName | cityName | countryName | similar | url | 
| :---: | :---: | :---: | :---: | :---: | :---: | 
| Seconds | 301 | 283 | 290 | 286 | 289 |


![Screenshot](scrapper.png)

```python

all_similar_array = []
all_images_array  = []

for city in list(df['cityName']): # loop for every city in column 'cityName'
    url = "https://nomadlist.com/similar/" + city.lower().replace(" ","-") # create url, i.e. 'New York' ->  "https://nomadlist.com/similar/new-york"
    
    if similarDestinations(url) != []: # url is valid for that city name
        all_similar_array.append(similarDestinations(url)) # append array of similar destinations
        all_images_array.append(imageDestinations(url))    # append image url
        
    else:
        all_similar_array.append(similarDestinations(url)) # [] empty array will be appended
        all_images_array.append("http://logok.org/wp-content/uploads/2014/04/British-Airways-logo-ribbon-logo.png")   # default image    
        
df['similar'] = all_similar_array  # add column   
df['url'] = all_images_array       # add column   

df.to_csv('<path>/airport_codes_200.csv') # save csv
    
#### Function definitions

def similarDestinations(url):
    # input:    string 'https://nomadlist.com/similar/<destination>'
    # returns:  array of strings ['madrid', 'lisbon', 'paris']
    response = requests.get(url)
    soup = BeautifulSoup(response.text, "html.parser")
    s = ''.join(str(tag) for tag in soup.findAll("h3", {"class": "itemName"}))
    similar_destinations_array = re.findall(r'href="/(.*?)"', s)
    return similar_destinations_array

def imageDestinations(url):
    # input:    string 'https://nomadlist.com/similar/<destination>'
    # returns:  string 'https://nomadlist.com/assets/img/cities/abu-dhabi-united-arab-emirates-500px.jpg'
    response = requests.get(url)
    soup = BeautifulSoup(response.text, "html.parser")
    image_url = "https://nomadlist.com" + soup.findAll("img", {"class": "bg-modal"})[0]['src']
    return image_url
    
```
