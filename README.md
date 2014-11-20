Code-for-image-table
====================


import glob
import multiprocessing
import time
import MySQLdb
import ast
from datetime import datetime
import os
import sys
import req_proxy
import logging
import time
import multiprocessing
import os
from threading import Thread
import requests
from bs4 import BeautifulSoup
from lxml import html
from Queue import Queue

logging.basicConfig(level=logging.DEBUG, format='[%(levelname)s] (%(threadName)-10s) %(message)s', )
num_fetch_process = 5
enclosure_queue = Queue()

def my_strip(x):
    try:
       
        x = str(x)
    except:
      
        x = str(x.encode("ascii", "ignore"))
    return x




def main(link,id1):
      
    page = req_proxy.main(link)
    soup = BeautifulSoup(page,"html.parser")

    filename =("/home/desktop/anit/zivame/image_files.csv")
    
    try:
        os.remove(filename)
    except OSError:
        pass

    image_file = open(filename,"a+")

    pimage_list=[]
    
    try:
        p1 = soup.find("ul",attrs={"id":"links"}).find_all("a")
    #print p1
        for p in p1:
            p2 = p.get("alt")
            pimage_list.append(p2)
   
        for p in pimage_list:
            p1 =p
            info = [id1,p]
            info2 =map(my_strip,info)
            image_file.write(str(info2)+"\n")

        print ".................inserted................"
         
    except:
        pass

    image_file.close()

def store_in_csv(i , q):
    
    for id1, link in iter(q.get, None):
        try:
            link = link.strip()
            main(link,id1)
            
            logging.debug(".......inserted....")
            
        except:
            pass

            time.sleep(2)
            q.task_done()

    q.task_done()

def supermain():
    

    db = MySQLdb.connect("192.99.13.229","root","6Tresxcvbhy","zivame")
    cursor = db.cursor()

    sql_select = """select product_url,product_id  from zivame_data"""
    cursor.execute(sql_select)

    results = cursor.fetchall()
    result =map(list,results)
 #   for url in result:
 #       try:
  #          url1 = url[0]
   #         id1 = url[1]
    #        main(url1,id1)
       
     #   except:
      #      pass

    proc = []
     
    for i in range(num_fetch_process):
        proc.append(Thread(name = str(i), target= store_in_csv, args=(i, enclosure_queue,)))
        proc[-1].start()
    
    for link in result:
        url1 = link[0]
        id1 =link[1]

        enclosure_queue.put((id1, url1))

    enclosure_queue.join()

    for p in proc:
        enclosure_queue.put(None) 
    
    enclosure_queue.join()

    for p in proc:
        p.join(120)

    print "Done............" 
         
    filename =("/home/desktop/anit/zivame/image_files.csv")

    try:
        os.remove(filename)
    except OSError:
        pass




if __name__ =="__main__":
    supermain()
#    main("http://www.zivame.com/nightwear/sleep-tees/coucou-gentle-cotton-short-sleeves-sleep-tee.html",1)
