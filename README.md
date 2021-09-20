# Smart Gardening Which Can Be Controlled and Monitored via Android App
**Demo Video Link:** https://youtu.be/rZxE7m1jKjA

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Objective:** Simulate the Smart Gardening system, which users can monitor and control it remotely via Android App.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

![](https://user-images.githubusercontent.com/79388911/134055092-9c23263a-1fa1-477a-92b8-20e012c8a9c1.png)

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**System Design:**

Pi 4-

(1) Receives data from sensors and posts them to the cloud.

(2) Gets data from the cloud and control the actuators.

Mobile Phone-

(1) Receives data from cloud.

(2) Posts users’ settings to cloud.

[Section 1] Measure temperature, humidity, lightness, & soil moisture.

[Section 2] Adjust lightness according to environment/settings.

[Section 3] Turn on/off water sprayer according to environment/settings.

Section On Phone-

if Refresh Clicked -> Get the newest value from cloud.

if Change Settings Clicked -> Post current settings to cloud.

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Flowchart:**

![](https://user-images.githubusercontent.com/79388911/134055158-9833ca5c-01eb-47b2-b73b-017132a403ab.png)

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**Code Explanation:** (Since most of things run on Raspberry Pi are same as the 3rd lab, I’ll cover the differences, and explain more about how I developed the App.)

**1. Compress Setting Data:**
I want to let users to choose whether they want to control LEDs and sprayer by themselves or by environment.
Also, they can assign the lightness & humidifier for every different hour.

So, we have 2 + 24 + 24 attributes. Can we build 50 data channels for them?

No! MCS only offers 10 data channels for free.

My solution for this problem is to compress all 50 data into one string.

This is the format:

“[assignlight]\n[assignhumid]\n[light0-1]\n…[light23-24]\n…[humid0-1]\n...[humid23-24]”

On Raspberry Pi, we have to parse the settings string:

```
def get_settings():
    host = "http://api.mediatek.com"
    endpoint = "/mcs/v2/devices/" + deviceId + "/datachannels/time/datapoints"
    url = host + endpoint
    headers = {"Content-type": "application/json", "deviceKey": deviceKey}
    r = requests.get(url,headers=headers)
    value = (r.json()["dataChannels"][0]["dataPoints"][0]["values"]["value"])
    return value

settings = get_settings().split('\n')  #split the string by ‘\n’, the result is a list

now = datetime.now()
current_time = now.strftime("%H:%M:%S").split(':')  #now.strftime gets current time in format “hh:mm:ss”, so if we split it by ‘:’, will get list = [hh, mm, ss]

if settings[0] == "True":
    val = int(settings[2+int(current_time[0])])
    pwm.ChangeDutyCycle(val)
    
#according to the compress format mentioned above, settings[0] means whether to use light settings
#current_time[0] = “hh”   it’s a string  -> use int() to convert to int
#int(settings[2 + int(current_time[0])]) is light setting value for hour = current hour
```

(+2 because settings[0], settings[1] are checkbox value)

(codes for “if settings[1] == “True”” work for same reasons)

**2. Details in Building Android App:**
(I use Kivy + Python + Buildozer to create my Android App.)

(1) Connect to Internet:

The Internet function works well when I test it on Ubuntu. However, when I try to transfer it to APK and install on my phone, it crashes.

That’s because lacking of requirements and permissions, so in buildozer.spec add:
```
requirements = python3,kivy,requests,urllib3,chardet,idna

android.permissions = INTERNET
```
Once added the lines, the GET/POST functions work successfully on Android systems.

(2) Require Storage:

Just as most APPs, when we finished the program and later restart it, we may want to retrieve past data.

The way to do this is to ask for a storage, store values, and retrieve later.

In buildozer.spec:
```
android.permissions = INTERNET, WRITE_EXTERNAL_STORAGE
```

In main_final_2.py:
```
store = JsonStore('hello.json')  #ask for storage
store.put('act1', name='Set_Time_Val', Vals=value)  #store value

assign_light=True

if (store.exists('act1')):
    assign_light = store.get('act1')['Vals']  #get value if it exists
```
(3) Scrollview & Sliders:

Since we have so many sliders (24 in a row), the mobile screen is not wide enough to show all, so we need to put all sliders (+ label and textInput with it) into a scrollview.

ex.
```
layout1 = BoxLayout(orientation = "vertical")
label1 = MyLabel(text='', size_hint=(args[0], args[1]))
layout1.add_widget(label1)
bar_1 = Slider()
bar_1.bind(value = self.on_off1)
layout1.add_widget(bar_1)
    
self.ids.container_x.add_widget(layout1)
self.ids.container_x.add_widget(layout2, layout3…)
```
Special design for sliders - Since users may not want to assign values for every hour (maybe they want to set values for every 2 or 3 hours), so I designed that the right sliders will follow the value of left sliders. Method:

```
def on_value1(self, instance, val):
    bar2.value = val
    
def on_value2(self, instance, val):
    bar3.value = val
    
def on_value3(self, instance, val):
    bar4.value = val
    
#always change the value of next bar when self’s value changed!
```

