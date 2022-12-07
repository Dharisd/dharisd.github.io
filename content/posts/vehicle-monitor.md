---
title: "Real-time Vehicle Detection and Speed Estimation using YOLO and Deepsort"
date: 2022-11-29T11:26:00+05:00
draft: false
---







Having a good view of the road infront and extra android phone laying around  

I wondered 

```
can i log very passing vehicle?
can i get their types (car,motorbike,truck)?
can i get their speed?
can i visualize how busy the
```

and generate live reports based on the logged data? if so that would look darn cool


This could technically be acheived through

* live stream of the road 
* server to process the stream and run object detection and tracking
* backend to save the data
* Frontend to do cool visuals

Final dashboard
![web-dashboard](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/images/Screenshot%202022-11-14%20at%2000.30.58.png)

## But how ?

#### camera for live stream

For this i just used my old android device and ran one of the mulitple ip camera apps available to get an RTSP stream from the device on to my laptop

blog_v3/assets/images/Screenshot 2022-12-07 at 09.21.04.png


#### How do i detect from this?

Technically it would be possible to get a 

* count of number of vehicles on road 
* classes of detections 

If I just run an object recognition model such as YOLO periodically on frames of the livestream. This method cannot track or identify detections across multiple frames. This prevents me from getting important data such as getting the speed of passing vehicles

#### so?
Enter [Deepsort](https://arxiv.org/abs/1703.07402). Deepsort allows to track detections across frames. This means that we can track a car and get its path from when it enters the frame to when it disappears from frame, as in in we can get the rough speed of the vehicle's pass, with code as simple as

![[Screenshot 2022-11-18 at 18.16.33.png]]

Fortunately [this]([https://github.com/HowieMa/DeepSORT_YOLOv5_Pytorch]) exists, i just had to make a few modifications to make it spit out the tracks of vehicles right before it gets deleted


![Deepsort-runnnig](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/images/Screenshot%202022-12-07%20at%2010.23.37.png)

**Easy and done right? nope**

Saving the data to the backend would require me to do a blocking HTTP API call, and if we are to run this in realtime this must be done with in the detection loop , obviously not good?

So how do we not block the detection loop but still make it output data while its running?


## We use Python Queues
start a worker in another thread 

```python 
import queue

q = queue.Queue()

  
def worker():

	while True:

		data = q.get()
		requests.post(f"http://{backend_url}/store-pass", json=data)
		print(f'Working on {data}')
		print(f'Finished {data}')
		q.task_done()

  
#start the thread
#less go


```

Then start an instance of this worker in another thread 
```python
threading.Thread(target=worker, daemon=True).start()
```

And then we can simply add to this queue with
```python 
q.put()
```
Whats next is to simply store the data and show it in interesting way

### Backend 
Backend is quite simple and is bulit using FastAPI and mongoDB to save data 


### Frontend 


For frontend i just modified [this very cool dashboard](https://github.com/themesberg/tailwind-dashboard-windster/) and just added the relevent fields, and some simple JS for fetching (needs improvement).





