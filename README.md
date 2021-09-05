import cred, json, time #import our cred file which has all the credentials, json and time.
from datetime import datetime, timedelta  #import dateime and timedelta library for fetching the current time.
from boltiot import Sms, Bolt #Fetch the data stored in Bolt Cloud and then based on their value send SMS.
mybolt = Bolt(cred.API_KEY, cred.DEVICE_ID) #fetch the data that is APL key and Device ID from Bolt Cloud, we will create an object of the same.
sms = Sms(cred.SSID, cred.AUTH_TOKEN, cred.TO_NUMBER, cred.FROM_NUMBER) #fetch the data from Twilio Dashboard, we will create an object of the same.
button_state  = 0 #for switching the states; acknowlegment button of taking medicine

def set_time(alarm_hr,alarm_min): 
        global button_state #Declaring button_state as Global Variable
        door_response = mybolt.digitalRead('0') #Medical Box sensor fixed at pin0; Taking response from pin0 
        door_data = json.loads(door_response)  #Coversion of any string in to json(object notation)
        button_response = mybolt.digitalRead('1') #Button fixed at pin1; Taking response from pin1 
        button_data = json.loads(button_response) #Coversion of any string in to json(object notation)
        current_time =  datetime.now().hour*60 +datetime.now().minute  #Converting Current time in to Minutes
        alarm_time=int(alarm_hr)*60 + int(alarm_min) #Converting alarm time in to minutes
        try:
            #Converting in to integer value.
            door = int(door_data['value']) 
            button = int(button_data['value']) 
            if (button ==1 and door==0): #Checking if the buttton is press or not when the door is open.
                button_state=1 
            if current_time == alarm_time:
                mybolt.digitalWrite('2','HIGH' )
                mybolt.digitalWrite('3','HIGH' )
                time.sleep(60)
                mybolt.digitalWrite('2','LOW')
                mybolt.digitalWrite('3','LOW')
            if ((current_time == alarm_time+2)and button_state==0):
                mybolt.digitalWrite('2','HIGH' )
                mybolt.digitalWrite('3','HIGH' )
                time.sleep(60)
                mybolt.digitalWrite('2','LOW')
                mybolt.digitalWrite('3','LOW')
            elif ((current_time==alarm_time+10) and button_state == 0): #Sending SMS after 10 minutes of pill taking
                response=sms.send_sms("Alert! your patient has not take the medicine. Please remind him/her")
                time.sleep(60)
            elif current_time == alarm_time+30:
                button_state = 0
            if (button ==1 and door==0):
                button_state=1
        except Exception as e:
            print("Error",e)
        time.sleep(10)
while True:
    set_time(10,3)
