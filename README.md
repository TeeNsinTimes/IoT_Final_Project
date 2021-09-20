# Smart Gardening Which Can Be Controlled and Monitored via Android App
**Demo Video Link:** https://youtu.be/rZxE7m1jKjA

**Objective:** Simulate the Smart Gardening system, which users can monitor and control it remotely via Android App.


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


Flowchart:
![](https://user-images.githubusercontent.com/79388911/134055092-9c23263a-1fa1-477a-92b8-20e012c8a9c1.png)
![](https://user-images.githubusercontent.com/79388911/134055158-9833ca5c-01eb-47b2-b73b-017132a403ab.png)
