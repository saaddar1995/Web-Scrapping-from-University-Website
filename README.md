# Web-Scrapping-from-University-Website
Retrieving Data from University Website

Scrapping data from any websites:

from bs4 import BeautifulSoup
import requests
import urllib.request as urllib2
import datetime
import time
# Print Timestamp At time of crawl
datePosted = str(datetime.date.today())
print("Time of Crawl:" + datePosted)
#these below are libraries which are used for below code
import json
import os
import nltk
import re,string
nltk.download("punkt")
from nltk.tokenize import word_tokenize
from nltk.stem import PorterStemmer
nltk.download("stopwords")
from nltk.corpus import stopwords

# Create a RobotFileParser object for the website and this will check robot.txt file rule which are allow or not.
rp = urllib.robotparser.RobotFileParser()

# Set the URL of the website's robots.txt file
rp.set_url("https://pureportal.coventry.ac.uk" + "/robots.txt")
url = "https://pureportal.coventry.ac.uk"
#
rp.read()
if rp.can_fetch("*", url): #check * in file so if in file it will run 
  print("Crawling allowed")

  def scrape_website():
    lst = []

    for x in range(0,6): # header code used for the user agent string for HTTP requests
          headers = {'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36'}

          r= requests.get(f"https://pureportal.coventry.ac.uk/en/organisations/research-centre-for-computational-science-and-mathematical-modell/publications/?page={x}",headers=headers)
          soup = BeautifulSoup(r.text,'html.parser')
          
      #this will get h3 tag from html
          for h3_tag in soup.find_all("h3"):
            a_tag = h3_tag.find('a')
          
            lst.append(a_tag.attrs['href'])
        
            
    print("links of title : ",lst)     
    print(len(lst))
    print("Total Number of Link Crawled : ",len(lst))
	


    Final_Dictionary = {}

    count = 0
    
    ###################### BELOW CODE IS FOR CRAWLER ############################
       ##################                            ##################
    for link in lst:
        
        request = requests.get(link)
        
        soup = BeautifulSoup(request.text,'html.parser')
        
        try:
            check_li_tag = soup.find_all('li',{'class' : 'researchgroup'})
        except:
          continue
        
        organization_list = []
        research_string = 'Research Centre for Computational Science and Mathematical Modelling'
          
        for x in check_li_tag:
          if x.find('span').string == research_string:
            print(f"Fetching Link from page {count//50+1} :\n ", link )
                  
            a_tags = soup.find_all('a',{'class' : 'link person'})
            authors = {}
            for y in a_tags:
              author_name = y.find('span').string
              author_link = y.get('href')
              
              current_author = {author_name:author_link} 
              authors=authors|current_author
              # | sign use for merging operators
            c= soup.find('div',{'class' : 'rendering'}).find('h1').find('span').string
            title_link = soup.find('div',{'class' : 'rendering'}).find('h1').find('span').string.lower()

            

            # below line is to delete  non-ASCII ch
            document_Test = re.sub(r'[^\x00-\x7F]+', ' ', title_link)
            # Remove Mentions,user tags as they are not relevant for ranking or retrieval 
            document_Test = re.sub(r'@\w+', '', document_Test)
            #this line below will delete Punctuations
            document_Test=re.sub(r'[%s]'% re.escape(string.punctuation),' ',document_Test)
            # if any number in string it will be delted
            document_Test = re.sub(r'[0-9]', '', document_Test)
            # below line will Remove the doubled space
            document_Test = re.sub(r'\s{2,}', ' ', document_Test)

            sw = stopwords.words('english')
            ps = PorterStemmer()
            #filtered_docs = []
           # for doc in check_list:
            tokens = word_tokenize(document_Test)
            tmp = ""
            for w in tokens:
               if w not in sw:
                  tmp += ps.stem(w) + " "
       
            title = tmp
            results = {
                      'Title':title_link,
                      'url':link,
                      'authors':authors,
                      'year':soup.find('span',{'class' : 'date'}).string[-4:] }

            result_Dictionary = {title : results}
            Final_Dictionary=  Final_Dictionary |result_Dictionary
        count= count+1
	print(results) 

    print(Final_Dictionary)
            
    ############## Below Code is used for writing data in Json File #############
    with open('Final_Dictionary.json','w')as json_file:
        contents = ''
    with open('Final_Dictionary.json','w') as json_file:
        contents += json.dumps(Final_Dictionary, indent=2)
        json_file.write(contents)

interval = 604800  # 7 Days and numbers are in seconds

# do crawling indefinitely
while True:
  scrape_website()
          
# Wait for the defined interval before running the crawler again
  time.sleep(interval)
else:
  print("Crawling not allowed")

import pandas as pd
with open("/content/Final_Dictionary.json","r") as informations: ####Reading Json File
      info=json.load(informations)
h= info.keys()
print(h)
Key_List= []

for x in info.keys(): #looping keys one by one in x variable
  Key_List.append(x) # appending in key_list

print(Key_List)
Query = input("Search Something :")

 # below line is to delete  non-ASCII ch
document_Test_check = re.sub(r'[^\x00-\x7F]+', ' ', Query)
# Remove Mentions,user tags as they are not relevant for ranking or retrieval 
document_Test_check = re.sub(r'@\w+', '', document_Test_check)
#this line below will delete Punctuations
document_Test_check=re.sub(r'[%s]'% re.escape(string.punctuation),' ',document_Test_check)
# if any number in string it will be delted
document_Test_check = re.sub(r'[0-9]', '', document_Test_check)
# below line will Remove the doubled space
document_Test_check = re.sub(r'\s{2,}', ' ', document_Test_check)


sw = stopwords.words('english')
ps = PorterStemmer()

tokens = word_tokenize(document_Test_check)
tmp = ""
for w in tokens:
    if w not in sw:
      tmp += ps.stem(w) + " "

Filtered_Query = tmp


search_list= Filtered_Query.split()

matching_titles = []
for Title in Key_List:
  split_word = Title.split()
  matching_word_list = []
  
  for words in split_word:
    if words in search_list:
      matching_word_list.append(words)

  if matching_word_list: # checking list is empty or not
    matching_titles.append(Title)
 
if not matching_titles:
  print("----No Result Found----")

for x in matching_titles:
  print(info[x]["Title"])
  print(info[x]["url"])
  print(info[x]["authors"])
  print(info[x]["year"])
  print("\n")
  
  /images/11.jpg
