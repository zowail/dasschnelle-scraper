# Welcome to dasschnelle.at scraper project

## The full source code is private since it's a paid software, contact me for more details

### In this page i will talk about some parts of the code including a sample from the source code.

**Starting from this url** : https://www.dasschnelle.at/ergebnisse?what=john&where=&distance=0&search_button=SUCHEN

The following code is written in **python 3.6.5** and **Selenium** which is a module for scraping and automating.

![Image of search results page](https://raw.githubusercontent.com/zowail/dasschnelle-scraper/master/dasschnelle%20search%20results%20page.PNG?token=AMfGvjJgWfWdTtmG3_NMxZGAiwpcpw3Sks5a7k7gwA%3D%3D)

**Basically the full script is reading input search queries from csv file and then visits the website then start searching for all names and clicks search and finally scrapes all names and phones under some conditions**

**But in this page i will present only the part of scraping one search result page and not all of the website**

The script will open google chrome using the chrome driver then visits a search results page for any name e.g. "John"
and then will extract any contact details "Names and Phone Numbers" but if the name string has more than 2 words then it will skip it and also if there is no phone number it will be skipped as well.

**First import the needed modules**
```python
from time import sleep
from selenium.webdriver.common.keys import Keys
from selenium import webdriver
from parsel import Selector
import csv
import os.path
from collections import defaultdict
import re
```

**Then create the function that extracts the name and phone from all contacts then append them to two lists which are the return variables**
```python
def get_names_and_phones_from_search_results():
    
    # 2 lists that will contain all names and phones from the page
    names_list = []
    phones_list = []

    # get all parent web elements of all names and phones in the page
    name_and_phone_parent_web_elements = driver.find_elements_by_css_selector("article[typeof = 'Organization']")

    # for every parent web element in the page get the name and the phone web elements and then get the text
    for parent_web_element in name_and_phone_parent_web_elements:
        
        # if the contact doesn't have a name then skip this contact
        try:
            name_web_element = parent_web_element.find_element_by_css_selector("h3[property='legalName']")
            name = str(name_web_element.text).strip()

            # count number of words in the name, if more than 2 skip the appending part
            number_of_spaces_in_name = name.count(" ")
            if number_of_spaces_in_name > 1:
                continue
            else:
                # if the phone is not found skip this contact
                try:
                    phone_web_element = parent_web_element.find_element_by_css_selector("span[property = 'telephone']")
                    phone = str(phone_web_element.text).strip()

                    # if the phone string is empty then skip
                    if (phone == ""):
                        continue

                    # if the contact's name has 2 words or less and the phone is found then append to the list
                    else:
                        names_list.append( name )
                        phones_list.append( str(phone) )
                except:
                    pass
                    continue
        except:
            pass
            continue
    return names_list, phones_list
```

**Another function that takes 2 lists as arguments then write them to a csv file**
```python
def write_names_and_phones_to_csv(names,phones):
    

    ####################### writing to csv part #########################
    file_exists = os.path.isfile("Contact_Details.csv")
    with open ("Contact_Details.csv", "a", newline='',encoding='utf-8') as resultFile:  # open input file for reading

        #writer = csv.writer(resultFile)
        writer = csv.DictWriter(resultFile,
                            fieldnames=["Name"
                            ,"Phone"])
        if not file_exists:
            writer.writeheader()  # file doesn't exist yet, write a header
        
        spamwriter = csv.writer(resultFile, delimiter=',',
                                            quotechar='|', quoting=csv.QUOTE_MINIMAL)

        writer.writerows({"Name": var1
                        ,"Phone": var2
                        } for
                        var1
                        ,var2
                        in zip( names
                                ,phones) )
    resultFile.close()
```

**Now let's communicate with Google Chrome using the chrome driver and go to the search results page**
```python
driver = webdriver.Chrome('C:\chromedriver\chromedriver.exe')


driver.get("https://www.dasschnelle.at/ergebnisse?what=david&where=&distance=0&search_button=SUCHEN")

sleep(0.5)
```

**Finally lets parse the page then print the results and also write them to a csv file**
```python
names, phones = get_names_and_phones_from_search_results()

for i in range(len(names)):
    print(names[i], " ", phones[i])

# write names and phones to csv
write_names_and_phones_to_csv(names,phones)
```

![Image of CSV Output](https://raw.githubusercontent.com/zowail/dasschnelle-scraper/master/csv%20results.PNG?token=AMfGvvg0TvqbQMrmcymnm_3ijAMt57sXks5a7k9awA%3D%3D)

![Image of VS Code Output](https://raw.githubusercontent.com/zowail/dasschnelle-scraper/master/vs%20code%20output.PNG?token=AMfGvr7BmAkYB8yQY0-zIvG_GqjAvkpRks5a7k9jwA%3D%3D)
