# Scraping behind password

Scraping data from web pages that requires login and password represents addional challange to stay authorized during the process. This is where `requests.session()` is extremely handy.

### Background

Objective for this project was to collect data from web site with huge price list of prudcts for health. While some information is available without authorization, some important data is revealed only if you stay connected after login.

### Solution

Start with importing only 3 libraries:
```
import requests
import time
from bs4 import BeautifulSoup
```
Python's `request` does all heavy job of connecting, loging and keep session alive while `BeautifulSoup` libraries helps to collect data for navigation and scraping. 

First step before loging is to get token value from hidden input of the form. Then you need to provide **headers** and **cookies** data for `requests.session()`. And do not forget to provide **data** with login and password:
```
# start the session
s = requests.Session()

# post to page to login
s.post('https://www.fab-ent.com/manage-account/', headers=headers, cookies=cookies, data=data)
time.sleep(10)

# check if login succesfull
response = s.get('https://www.fab-ent.com/manage-account/', headers=headers)
time.sleep(10)
soup = BeautifulSoup(response.content, 'html.parser')
print(f"logged-in as {soup.find_all('a', {'class':'aboveHeaderLink'})[1].text}")
```
Once you get connected, you need to collect navigation data from the site, and then follow the links to scrape data as needed from every page:
```
# collect subcategories
for cat_num in range(len(categories)):
  url = base_url + categories_links[cat_num]
  print(f'collecting subcategories from {url}..')
  response = s.get(url, headers=headers)
  time.sleep(10)
  soup = BeautifulSoup(response.content, 'html.parser')
  side_bar = soup.find('aside', {'id' : 'catSidebar'})
  # collect subcategories
  sub_categories = [link.text for link in side_bar.find('a', {'class':'more'}).next_sibling.find_all('a')]
  sub_categories_links = [link['href'] for link in side_bar.find('a', {'class':'more'}).next_sibling.find_all('a')]
```
Once you get to the page with data needed you just use table tags for scraping. 

![scraping process](https://github.com/deal2k/scraping_behind_password/blob/main/scraping.png?raw=true)

Finall step is to clean data with `pandas` and export to CSV format.

![clean and export data](https://github.com/deal2k/scraping_behind_password/blob/main/export.png?raw=true)
