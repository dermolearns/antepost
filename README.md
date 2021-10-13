import pandas as pd 
import numpy as np
import time
from selenium import webdriver
import datetime as dt

chromedriver_path = r"C:\chromedriver.exe"
antepost_url = "https://www.attheraces.com/antepost"

options = webdriver.ChromeOptions()
options.add_argument("--start-maximized")

driver = webdriver.Chrome(chromedriver_path, chrome_options=options)

driver.get(antepost_url)
time.sleep(3)


def gdpr_agreement_click():
    
    xpath1 = """/html/body/div/div/div/div/div[2]/div/button[text()='AGREE']"""

    gdpr_agreement_click = driver.find_element_by_xpath(xpath1)
    gdpr_agreement_click.click()
    
    return gdpr_agreement_click

gdpr_agreement_click = gdpr_agreement_click()

def expand_all_races():
    # expand all runners in every race    
    i = 0
      # locate all elements
    elements = driver.find_elements_by_xpath('//span[@class="reveal reveal--inline"]')
    while len(elements)/2 > i:
        elements[i].click() # click on the i-th element in the list
        i += 1 # increment i
        time.sleep(0.5) # wait until list will be updated
        #continue

expand_all_races = expand_all_races()        
    

def all_scrapes():
    
    jumps = driver.find_elements_by_xpath('//div[@class="antepost-filter-all antepost-filter-jumps"]//a')

    jumps_list  = []

    for entry in range(len(jumps)//2):
        jumps_list.append(jumps[entry].text)
        
    cheltenham = driver.find_elements_by_xpath('//div[@class="antepost-filter-all antepost-filter-cheltenham antepost-filter-jumps"]//a')

    cheltenham_list  = []

    for entry in range(len(cheltenham)//2):
        cheltenham_list.append(cheltenham[entry].text)

        
    flat = driver.find_elements_by_xpath('//div[@class="antepost-filter-all antepost-filter-flat"]//a')

    flat_list  = []

    for entry in range(len(flat)//2):
        flat_list.append(flat[entry].text)    
   
     
    international = driver.find_elements_by_xpath('//div[@class="antepost-filter-all antepost-filter-international"]//a')

    international_list  = []

    for entry in range(len(international)//2):
        international_list.append(international[entry].text)

    ascot = driver.find_elements_by_xpath('//div[@class="antepost-filter-all antepost-filter-ascot"]//a')

    ascot_list  = []

    for entry in range(len(ascot)//2):
        ascot_list.append(ascot[entry].text)
        
    def all_todays_races():
        
        todays_url = "https://www.attheraces.com/runners"
        driver.get(todays_url)
        selecting_all = driver.find_element_by_xpath('//select[@id="courses"]/option[1]')
        selecting_all.click()
        
        
        go = driver.find_element_by_xpath('//*[@id="runner-form-go2"]')
        go.click()
        
        return selecting_all, go
    
    selecting_all, go = all_todays_races()
    
    
    scrape_todays = driver.find_elements_by_xpath('//div[@id="site-content"]/div/main/div/div[1]/div/div/div[2]/table/tbody/tr')

    scrape_todays_list  = []

    for entry in range(len(scrape_todays)):
        scrape_todays_list.append(scrape_todays[entry].text)
        
    driver.close()    

    return jumps_list,flat_list, scrape_todays_list,international_list, ascot_list,cheltenham_list

jumps_list,flat_list, scrape_todays_list,international_list, ascot_list,cheltenham_list = all_scrapes()


def antepost_jumps(jumps_list):
        
    df_jumps = pd.DataFrame(jumps_list, columns = ['Data'])
    
    df_jump_races = df_jumps[df_jumps['Data'].str.contains(':')]
    df_jump_races['race_index_val'] = df_jump_races.index
    df_jump_races = df_jump_races.reset_index(drop=True)
    
    df_jump_runners = df_jumps[~df_jumps['Data'].str.contains(':')]
    df_jump_runners['horse_index_val'] = df_jump_runners.index
    
    
    df_jump_runners_last = df_jump_runners[~df_jump_runners['Data'].str.contains(',')]
    df_jump_runners_last = df_jump_runners_last.rename(columns={'Data': 'Horse','horse_index_val':'last_horse_index_val'})
    df_jump_runners_last = df_jump_runners_last.reset_index(drop=True)
    
    
    df_jumps_index = pd.concat([df_jump_runners_last, df_jump_races], axis=1)
    df_jumps_index = df_jumps_index.rename(columns={'Data': 'Race'})
    df_jumps_index = df_jumps_index[['Race','race_index_val','last_horse_index_val']]
    
    
    horse_index = df_jump_runners.horse_index_val.values
    race_index = df_jumps_index.race_index_val.values
    last_horse_index = df_jumps_index.last_horse_index_val.values
    
    i, j = np.where((horse_index[:, None] >= race_index) & (horse_index[:, None] <= last_horse_index))
    
    df_jumps_horse_race = pd.DataFrame(
        np.column_stack([df_jump_runners.values[i], df_jumps_index.values[j]]),
        columns=df_jump_runners.columns.append(df_jumps_index.columns))
    
    return df_jumps_horse_race

df_jumps_horse_race = antepost_jumps(jumps_list)

def antepost_cheltenham(cheltenham_list):
        
    df = pd.DataFrame(cheltenham_list, columns = ['Data'])
    
    df_races = df[df['Data'].str.contains(':')]
    df_races['race_index_val'] = df_races.index
    df_races = df_races.reset_index(drop=True)
    
    df_runners = df[~df['Data'].str.contains(':')]
    df_runners['horse_index_val'] = df_runners.index
    
    
    df_runners_last = df_runners[~df_runners['Data'].str.contains(',')]
    df_runners_last = df_runners_last.rename(columns={'Data': 'Horse','horse_index_val':'last_horse_index_val'})
    df_runners_last = df_runners_last.reset_index(drop=True)
    
    
    df_index = pd.concat([df_runners_last, df_races], axis=1)
    df_index = df_index.rename(columns={'Data': 'Race'})
    df_index = df_index[['Race','race_index_val','last_horse_index_val']]
    
    
    horse_index = df_runners.horse_index_val.values
    race_index = df_index.race_index_val.values
    last_horse_index = df_index.last_horse_index_val.values
    
    i, j = np.where((horse_index[:, None] >= race_index) & (horse_index[:, None] <= last_horse_index))
    
    df_cheltenham_horse_race = pd.DataFrame(
        np.column_stack([df_runners.values[i], df_index.values[j]]),
        columns=df_runners.columns.append(df_index.columns))
    
    return df_cheltenham_horse_race

df_cheltenham_horse_race = antepost_cheltenham(cheltenham_list)

def antepost_flat(flat_list):
    
    df_flat = pd.DataFrame(flat_list, columns = ['Data'])
    
    df_flat_races = df_flat[df_flat['Data'].str.contains(':')]
    df_flat_races['race_index_val'] = df_flat_races.index
    df_flat_races = df_flat_races.reset_index(drop=True)
    
    df_flat_runners = df_flat[~df_flat['Data'].str.contains(':')]
    df_flat_runners['horse_index_val'] = df_flat_runners.index
    
    df_flat_runners_last = df_flat_runners[~df_flat_runners['Data'].str.contains(',')]
    df_flat_runners_last = df_flat_runners_last.rename(columns={'Data': 'Horse','horse_index_val':'last_horse_index_val'})
    df_flat_runners_last = df_flat_runners_last.reset_index(drop=True)
    
    df_flats_index = pd.concat([df_flat_runners_last, df_flat_races], axis=1)
    df_flats_index = df_flats_index.rename(columns={'Data': 'Race'})
    df_flats_index = df_flats_index[['Race','race_index_val','last_horse_index_val']]
    
    
    horse_index = df_flat_runners.horse_index_val.values
    race_index = df_flats_index.race_index_val.values
    last_horse_index = df_flats_index.last_horse_index_val.values
    
    i, j = np.where((horse_index[:, None] >= race_index) & (horse_index[:, None] <= last_horse_index))
    
    df_flats_horse_race = pd.DataFrame(
        np.column_stack([df_flat_runners.values[i], df_flats_index.values[j]]),
        columns=df_flat_runners.columns.append(df_flats_index.columns))
    
    return df_flats_horse_race

df_flats_horse_race = antepost_flat(flat_list)

def antepost_ascot(ascot_list):
    
    df_ascot = pd.DataFrame(ascot_list, columns = ['Data'])
    
    df_ascot_races = df_ascot[df_ascot['Data'].str.contains(':')]
    df_ascot_races['race_index_val'] = df_ascot_races.index
    df_ascot_races = df_ascot_races.reset_index(drop=True)
    
    df_ascot_runners = df_ascot[~df_ascot['Data'].str.contains(':')]
    df_ascot_runners['horse_index_val'] = df_ascot_runners.index
    
    df_ascot_runners_last = df_ascot_runners[~df_ascot_runners['Data'].str.contains(',')]
    df_ascot_runners_last = df_ascot_runners_last.rename(columns={'Data': 'Horse','horse_index_val':'last_horse_index_val'})
    df_ascot_runners_last = df_ascot_runners_last.reset_index(drop=True)
    
    df_ascot_index = pd.concat([df_ascot_runners_last, df_ascot_races], axis=1)
    df_ascot_index = df_ascot_index.rename(columns={'Data': 'Race'})
    df_ascot_index = df_ascot_index[['Race','race_index_val','last_horse_index_val']]
    
    
    horse_index = df_ascot_runners.horse_index_val.values
    race_index = df_ascot_index.race_index_val.values
    last_horse_index = df_ascot_index.last_horse_index_val.values
    
    i, j = np.where((horse_index[:, None] >= race_index) & (horse_index[:, None] <= last_horse_index))
    
    df_ascot_horse_race = pd.DataFrame(
        np.column_stack([df_ascot_runners.values[i], df_ascot_index.values[j]]),
        columns=df_ascot_runners.columns.append(df_ascot_index.columns))
    
    return df_ascot_horse_race

df_ascot_horse_race = antepost_ascot(ascot_list)


def antepost_international(international_list):
    
    df_international = pd.DataFrame(international_list, columns = ['Data'])
    
    df_international_races = df_international[df_international['Data'].str.contains(':')]
    df_international_races['race_index_val'] = df_international_races.index
    df_international_races = df_international_races.reset_index(drop=True)
    
    
    df_international_runners = df_international[~df_international['Data'].str.contains(':')]
    df_international_runners['horse_index_val'] = df_international_runners.index
    
    df_international_runners_last = df_international_runners[~df_international_runners['Data'].str.contains(',')]
    df_international_runners_last = df_international_runners_last.rename(columns={'Data': 'Horse','horse_index_val':'last_horse_index_val'})
    df_international_runners_last = df_international_runners_last.reset_index(drop=True)
    
    df_international_index = pd.concat([df_international_runners_last, df_international_races], axis=1)
    df_international_index = df_international_index.rename(columns={'Data': 'Race'})
    df_international_index = df_international_index[['Race','race_index_val','last_horse_index_val']]
    
    horse_index = df_international_runners.horse_index_val.values
    race_index = df_international_index.race_index_val.values
    last_horse_index = df_international_index.last_horse_index_val.values
    
    i, j = np.where((horse_index[:, None] >= race_index) & (horse_index[:, None] <= last_horse_index))
    
    df_international_horse_race = pd.DataFrame(
        np.column_stack([df_international_runners.values[i], df_international_index.values[j]]),
        columns=df_international_runners.columns.append(df_international_index.columns))
    
    
    return df_international_horse_race

df_international_horse_race = antepost_international(international_list)


def antepost_fixtures(df_flats_horse_race,df_jumps_horse_race,df_international_horse_race, df_ascot_horse_race,df_cheltenham_horse_race):
    
    antepost_fixtures = pd.concat([df_flats_horse_race,df_jumps_horse_race,df_international_horse_race, df_ascot_horse_race,df_cheltenham_horse_race])
    antepost_fixtures = antepost_fixtures[['Data','Race']]
    antepost_fixtures = antepost_fixtures.rename(columns={'Data': 'Horse','Race':'Antepost Race'})
    antepost_fixtures = antepost_fixtures.replace(',','', regex=True)

    new = antepost_fixtures["Horse"].str.split(" ", n = 1, expand = True) 
  
    # making separate antepost odds column from new data frame 
    antepost_fixtures["Antepost Odds"]= new[0] 
  
    # making separate horse name column from new data frame 
    antepost_fixtures["Horse"]= new[1]
    antepost_fixtures = antepost_fixtures[['Antepost Race','Horse','Antepost Odds']]
    antepost_fixtures.sort_values(by=['Antepost Race','Antepost Odds']).reset_index(drop=True)
    
    # removing | from some horse names
    antepost_fixtures['Horse'] = antepost_fixtures['Horse'].str.replace("|","")
    
    return antepost_fixtures

antepost_fixtures = antepost_fixtures(df_flats_horse_race,df_jumps_horse_race,df_international_horse_race,df_ascot_horse_race,df_cheltenham_horse_race)



def scrape_todays(scrape_todays_list):
    
    df_scrape_todays = pd.DataFrame(scrape_todays_list, columns = ['Data'])
    
    new = df_scrape_todays["Data"].str.split(":", n = 1, expand = True) 
    
    new3 = new[0].str.rsplit(" ", n = 1, expand = True)
    
    new4 = new[1].str.split(" ", n = 1, expand = True)
    
    df_scrape_todays["Today's Race"]= new3[1] + ":" + new4[0] + " @ " + new3[0]
   
    df_scrape_todays["Horse"]= new4[1]
    
    df_scrape_todays = df_scrape_todays[['Horse',"Today's Race"]]
    To_remove_lst = ['WS5 ','WS4 ','WS3 ', 'WS2 ','WS ',\
                     ' t1',' v1',' b1',' h1', ' p1',\
                         ' t',' v',' b',' h',' p']
    
    df_scrape_todays['Horse'] = df_scrape_todays['Horse'].str.replace('|'.join(To_remove_lst), '')
    
    return df_scrape_todays

df_scrape_todays = scrape_todays(scrape_todays_list)


def todays_future_anteposts(df_scrape_todays,antepost_fixtures):
    
    todays_future_anteposts = pd.merge(df_scrape_todays,antepost_fixtures, how = 'inner', left_on = 'Horse', right_on= 'Horse')

    todays_future_anteposts = todays_future_anteposts[["Horse","Today's Race","Antepost Race","Antepost Odds"]]

    return todays_future_anteposts

todays_future_anteposts = todays_future_anteposts(df_scrape_todays,antepost_fixtures)




todays_date = dt.date.today().strftime('%d_%m_%Y')


todays_date = dt.date.today().strftime('%d_%m_%Y')
    

