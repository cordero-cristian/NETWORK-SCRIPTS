#Start Imports

 

from piapi import PIAPI

import pandas as pd

import numpy as np

import pprint

from pathlib import Path

from datetime import datetime

from openpyxl import load_workbook

import platform

import os

import time

import smtplib

import mimetypes

from email.message import EmailMessage

from email.mime.multipart import MIMEMultipart

from email.mime.base import MIMEBase

from email.mime.text import MIMEText

from email.mime.image import MIMEImage

from email import encoders

import matplotlib.pyplot as plt

 

#End Imports

#start of Globals

 

#globals need for datetime

#set time zone to EST

os.environ["TZ"] = "America/New_York"

time.tzset()

today = datetime.now()

today = str(today.strftime("%Y-%m-%d"))

dateFormatCsv = "%B-%d-%Y %H"

 

#globals need for path

fileExtension = ".xlsx"

fileExtensionCsv = ".csv"

fileExtensionPng = ".png"

currentDir = Path.cwd()

resultsDir = Path(str(currentDir) + "/Results")

fullFilePath = Path(str(resultsDir) + r”name of your report_" + today + fileExtension )

fullFilePathCsv = Path(str(resultsDir) + r”name of your report_" + today + fileExtensionCsv)

fullFilePathPng = Path(str(resultsDir) + r”name of your report_" + today + fileExtensionPng)

 

#globals need for Pandas

idx = pd.date_range(today, periods=24,freq='H')[::-1]

columnNames = ['Total Aps','Total User Count','Most Used AP Name','Amount of Users on Most Used AP']

#can add 'change to total ups and downs', 'chagne to user count', 2nd most used ap,

columnNamesUpDown = ['Total Aps UP','Total Aps 802.11a/n/ac UP','Total Aps 802.11b/g/n UP','Total Aps DOWN','Total Aps 802.11a/n/ac DOWN','Total Aps 802.11b/g/n DOWN','Total APs']

 

totalApsDownList = list()

totalApsUp = int()

totalApsDown = int()

totalAps802AUp = int()

totalAps802BUp = int()

totalAps802ADown = int()

totalAps802BDown = int()

TotalAps = int()

 

#globals needed for Prime API

username = input('Please Enter API Username' + "\n")

password = input('please enter API PW'+ "\n")

 

accessPointWithTheMostClients = dict()

accessPointWithTheMostClientsName = ''

accessPointWithTheMostClientsCount = 0

accessPointWithTheMostClients = {accessPointWithTheMostClientsName : accessPointWithTheMostClientsCount}

#End of Globals

 

def mostUsedAccessPoint(aListOfAllAccessPointsDetails):

    #loops thru the api call /webacs/api/v1/data/AccessPoints.json

    #returns the AP with the 2nd most clients in a Dict: {APNAME : COUNT}

    #the AP with the most clients can be called with accessPointWithTheMostClients.keys()

   

    for accessPointDetail in aListOfAllAccessPointsDetails:

       

        apName = accessPointDetail['accessPointDetailsDTO']['name']

        apClientCount = int(accessPointDetail['accessPointDetailsDTO']['clientCount'])

       

        accessPointWithTheMostClientsCountTemp = list(accessPointWithTheMostClients.values())

        accessPointWithTheMostClientsCountTemp = int(accessPointWithTheMostClientsCountTemp[0])

       

        accessPointWithTheMostClientsNameTemp = list(accessPointWithTheMostClients.keys())

        accessPointWithTheMostClientsNameTemp = str(accessPointWithTheMostClientsNameTemp[0])

       

        if apClientCount > accessPointWithTheMostClientsCountTemp:

           

            oldAccessPointWithTheMostClientsCount = accessPointWithTheMostClients.pop(accessPointWithTheMostClientsNameTemp)

            updateDict = {apName : apClientCount}

            accessPointWithTheMostClients.update(updateDict)

           

            oldAccessPointWithTheMostClients = {accessPointWithTheMostClientsNameTemp : oldAccessPointWithTheMostClientsCount}

           

            return oldAccessPointWithTheMostClients

 

def totalClientCount(aListOfAllAccessPointsDetails):

   

    allClients = int()

 

    for accessPointDetail in aListOfAllAccessPointsDetails:

        apClientCount = int(accessPointDetail['accessPointDetailsDTO']['clientCount'])

        allClients = allClients + apClientCount

   

    return allClients

 

def runPrimeReport(reportName):

    reports = api.request("report", params = {'reportTitle':reportName})

   

    dfTemp = pd.DataFrame(columns = columnNamesUpDown, index=idx)

    

    if reportName == 'Test':

       

        listOfEntries = reports['mgmtResponse']['reportDataDTO'][0]['dataRows']['dataRow']

       

        listToDfName = list()

        listToDf802A = list()

        listToDf802B = list()

       

        for item in listOfEntries:

   

            tempName = item['entries']['entry'][0]['dataValue']

            temp802AStatus = item['entries']['entry'][9]['dataValue']

            temp802BStatus = item['entries']['entry'][10]['dataValue']

            listToDfName.append(tempName)

            listToDf802A.append(temp802AStatus)

            listToDf802B.append(temp802BStatus)

            if temp802AStatus == 'Down' and  temp802BStatus == 'Down':

                totalApsDownList.append(tempName)

       

        seriesName = pd.Series(listToDfName,name='AP Name')

        series802A = pd.Series(listToDf802A,name='802.11a/n/ac Status')

        series802B = pd.Series(listToDf802B,name='802.11b/g/n Status')

       

        columnNamesTemp = ['AP Name','802.11a/n/ac Status','802.11b/g/n Status']

        inxTemp = pd.RangeIndex(stop=len(seriesName))

        dfTemp1 = pd.DataFrame(columns=columnNamesTemp, index=inxTemp)

       

        dfName = pd.DataFrame(seriesName)

        df802A = pd.DataFrame(series802A)

        df802B = pd.DataFrame(series802B)

 

        dfTemp1.update(dfName)

        dfTemp1.update(df802A)

        dfTemp1.update(df802B)

       

        totalApsUp = len(dfTemp1[(dfTemp1['802.11a/n/ac Status']=='Up') & (dfTemp1['802.11b/g/n Status']=='Up')].index)

        totalApsDown = len(dfTemp1[(dfTemp1['802.11a/n/ac Status']=='Down') & (dfTemp1['802.11b/g/n Status']=='Down')].index)

 

        totalAps802AUp = len(dfTemp1[(dfTemp1['802.11a/n/ac Status']=='Up')].index)

        totalAps802ADown = len(dfTemp1[(dfTemp1['802.11a/n/ac Status']=='Down')].index)

 

        totalAps802BUp = len(dfTemp1[(dfTemp1['802.11b/g/n Status']=='Up')].index)

        totalAps802BDown = len(dfTemp1[(dfTemp1['802.11b/g/n Status']=='Down')].index)

        TotalAps = len(dfTemp1)

        pprint.pprint(TotalAps)

        myhour=getHour()

 

        dfTemp.at[myhour,'Total Aps UP'] = int(totalApsUp)

        dfTemp.at[myhour,'Total Aps DOWN'] = int(totalApsDown)

 

        dfTemp.at[myhour,'Total Aps 802.11a/n/ac UP'] = int(totalAps802AUp)

        dfTemp.at[myhour,'Total Aps 802.11b/g/n UP'] = int(totalAps802BUp)

 

        dfTemp.at[myhour,'Total Aps 802.11a/n/ac DOWN'] = int(totalAps802ADown)

        dfTemp.at[myhour,'Total Aps 802.11b/g/n DOWN'] = int(totalAps802BDown)

        dfTemp.at[myhour,'Total Aps'] = int(TotalAps)

       

        with open(fullFilePathCsv,'w') as file:

            dfTemp.to_csv(file,index =False)

       

        axes = dfTemp[['Total Aps UP','Total Aps DOWN','Total Aps']].plot(sort_columns=True,subplots=False,kind='barh',width=.5)

        for rect in axes.patches:   

            # Get X and Y placement of label from rect.

            x_value = rect.get_width()

            y_value = rect.get_y() + rect.get_height() / 2

 

            # Number of points between bar and label. Change to your liking.

            space = 5

            # Vertical alignment for positive values

            ha = 'left'

 

            # If value of bar is negative: Place label left of bar

            if x_value < 0:

                # Invert space to place label to the left

                space *= -1

                # Horizontally align label at right

                ha = 'right'

 

            # Use X value as label and format number with one decimal place

            label = x_value

   

            if label != 0:

            # Create annotation

                axes.annotate(

                    label,                      # Use `label` as label

                    (x_value, y_value),         # Place label at end of the bar

                    xytext=(space, 0),          # Horizontally shift label by `space`

                    textcoords="offset points", # Interpret `xytext` as offset in points

                    va='center',                # Vertically center label

                    ha=ha)

        axes.figure.savefig(fullFilePathPng)

 

def getHour():

    dateToBeReturned = " "

    now = datetime.now()

    hour = now.hour

    today = datetime.now()

    today = str(today.strftime("%Y-%m-%d"))

    if hour < 10:

        dateToBeReturned = today + " 0" + str(hour) + ":00:00"

   

        return dateToBeReturned

   

    else:

        dateToBeReturned = today + " " + str(hour) + ":00:00"

   

    return dateToBeReturned

   

def excelFileWriter(dfTypeObj):

   

    try:

        #get Dataframe from current excel

        dfToMerge = pd.read_excel(fullFilePath,parse_dates=False,index_col=0)

        #start a book instance of the excel file

        #book = load_workbook(fullFilePath)

        #a = book.get_sheet_by_name(today)

        #book.remove_sheet(a)

        #delete the old data since we have it in a DataFrame

        #book.save(fullFilePath)

        writer = pd.ExcelWriter(fullFilePath, engine='openpyxl',datetime_format='mmm d yyyy hh:mm',mode='a')

        #merge old data with up-to date data

        dfTypeObj = dfTypeObj.combine_first(dfToMerge)

        #send to Excelfile

        dfTypeObj.to_excel(writer)

        #save it

        writer.save()

   

    except:

        df = pd.DataFrame()

        writer = pd.ExcelWriter(fullFilePath, engine='openpyxl',mode='w')

        dfTypeObj.to_excel(df)

        writer.save()

       

def csvWriter(dfTypeObj):

    try:

        dfToMerge = pd.read_csv(fullFilePathCsv,parse_dates=True,index_col=0,)

        dfTypeObj = dfTypeObj.combine_first(dfToMerge)

        dfTypeObj.to_csv(fullFilePathCsv,date_format=dateFormatCsv,mode='w')

    except:

        pprint.pprint(dfTypeObj)

        dfTypeObj.to_csv(fullFilePathCsv,date_format=dateFormatCsv,mode='w')   

    

def updateDataFrame(dfTypeObj,columnName,updateValue):

   

    dateindex = getHour()

    dfTypeObj.at[dateindex,columnName] = updateValue

 

 

def sendEmail(fileToSend,fileName):

    fileName = str(fileName)

    #use with open File then add the file object as the fileToSend args. Name the attachment with fileName

    smtpServer =#smtp server

    sender = #sender address

    receiver #receiver address

    subject = #subject

    message = EmailMessage()

    message['Subject'] = subject

    message['From'] = sender

    message['To'] = receiver

    textForBody = #bleh

 

    message.set_content(textForBody)

 

    ctype, encoding = mimetypes.guess_type(fileToSend)

 

    if ctype is None or encoding is not None:

        ctype = "application/octet-stream"

   

    maintype, subtype = ctype.split("/", 1)

 

    if maintype == "text":

   

        fp = open(fileToSend)

        attachment = MIMEText(fp.read(), _subtype=subtype)

        fp.close()

    elif maintype == "image":

        fp = open(fileToSend, "rb")

        attachment = MIMEImage(fp.read(), _subtype=subtype)

        fp.close()

    else:

        fp = open(fileToSend, "rb")

        attachment = MIMEBase(maintype, subtype)

        attachment.set_payload(fp.read())

        fp.close()

        encoders.encode_base64(attachment)

   

    #attachment.add_header("Content-Disposition", "attachment", filename=fileName)

    message.add_attachment(attachment)

    with smtplib.SMTP(smtpServer,25) as mailServer:

        mailServer.sendmail(sender,receiver,message.as_string())

        pprint.pprint('Email Sent')

 

       

quickCheck = resultsDir.is_dir()

 

if quickCheck is False:

    resultsDir.mkdir()

if platform.system() == 'Linux':

    quickCheck = fullFilePathCsv.exists()

 

    if quickCheck is False:

        fullFilePathCsv.touch()

 

else:

    quickCheck = fullFilePath.exists()

 

    if quickCheck is False:

        fullFilePath.touch()

 

df = pd.DataFrame(columns = columnNames, index=idx)

 

api = PIAPI(#address to prime server,username,password, verify = False)

 
runPrimeReport(#report name)

sendEmail(str(fullFilePathPng),str('#nameOfReport'+today))
