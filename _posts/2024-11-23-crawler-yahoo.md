---
layout: post
title: Crawler Finance Yahoo
subtitle: Extracting available stocks of a desired region
tags: [crawler, python, selenium, logging]
author: Sarah Silva
--- 

The main goal of this project was to extract the available Stocks of a desired region. The website used was the [Yahoo Finance](https://finance.yahoo.com/screener/new). 

# Preparing the environment 

## Selenium
In order to extract the stocks, I used the Selenium framework, the most used one when we talk about web scrapping. Selenium is a free framework designed for automated testing of web applications via the browser. In addition to its use as a testing tool, it is widely used for collecting web data. It basically simulates user behavior on a web page.

The webdriver class is what allows us to simulate this behavior, as it is an interface to write instruction sets that can be run interchangeably in many browsers. The Options class can be used with the webdriver one to set specific configurations like log level, headless, cookies and so on.

The By class is a mechanism used to locate elements within a document. We can locate the elements by class name, CSS selector, ID, Name and so on. To view all the options, check the documentation [here](https://www.selenium.dev/selenium/docs/api/java/org/openqa/selenium/By.html).

The WebDriverWait class is a mechanism we use when we want to wait until something happens, usually when the element becomes visible, or finishes loading. The documentation is located [here](https://www.selenium.dev/selenium/docs/api/java/org/openqa/selenium/support/ui/WebDriverWait.html). The EC class is usually used with the WebDriverWait, to allow the webdriver to wait until an expected condition happens. All of the options you can check [here](https://www.selenium.dev/selenium/docs/api/java/org/openqa/selenium/support/ui/ExpectedConditions.html).


```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.chrome.options import Options
```

## pandas, time, logging, warning and sys

Some other frameworks I used were Pandas (to create and export the final file), time (to stop the code execution until the page loads specific elements), logging (to control and save the log along the execution), warning (to remove some warning messages), sys (to get the line number that gave error).

```python
import pandas as pd
import time, logging, warnings, sys
```

# Crawler

The first filter I had to do on the website, was to input the desired region. If the region didn't exist on the website, the code ended and the log was saved.

```python
WebDriverWait(self.driver, 20).until(EC.element_to_be_clickable((
    By.XPATH, "//*[@id='screener-criteria']/div[2]/div[1]/div[1]/div[1]/div/div[2]/ul/li[1]/button"))).click()
WebDriverWait(self.driver, 20).until(EC.element_to_be_clickable((
    By.XPATH, "//*[@id='screener-criteria']/div[2]/div[1]/div[1]/div[1]/div/div[2]/ul/li[1]/button"))).click()

WebDriverWait(self.driver, 20).until(EC.element_to_be_clickable((
    By.XPATH, "//*[@id='dropdown-menu']/div[1]/div[1]/div[1]/input"))).clear()
WebDriverWait(self.driver, 20).until(EC.element_to_be_clickable((
    By.XPATH, "//*[@id='dropdown-menu']/div[1]/div[1]/div[1]/input"))).send_keys(self.region)

try: 
    WebDriverWait(self.driver, 20).until(EC.element_to_be_clickable((
        By.XPATH, "//*[@id='dropdown-menu']/div[1]/div[2]/ul/li/label/input"))).click()
except:
    logging.error('Region does not exist, please verify')
    return
```

Depending on the region, there can be more than 1 page of Stocks, so we have to go through each page to collect each available value.

```python
table = self.driver.find_element("xpath","//*[@id='screener-results']/div[1]/div[2]/div[1]/table").get_attribute("outerHTML")
df_final = pd.read_html(table)[0]
df_final = df_final.iloc[:, :3]

while True:
    try:
        WebDriverWait(self.driver, 5).until(EC.element_to_be_clickable((
            By.XPATH, "//*[@id='screener-results']/div[1]/div[2]/div[2]/button[3]"))).click()
    except:
        logging.info('Stocks finished!')
        break
    else:
        time.sleep(2)
        table = self.driver.find_element("xpath","//*[@id='screener-results']/div[1]/div[2]/div[1]/table").get_attribute("outerHTML")
        df = pd.read_html(table)[0]
        df = df.iloc[:, :3]
        df_final = pd.concat([df_final, df])
```

After we scan all the pages, the final dataframe is exported to a csv file.