---
title: "Real-time Vehicle Detection and Speed Estimation using YOLO and Deepsort : Part II"
date: 2023-01-16T18:57:46+05:00
draft: false
---


#### New features added 

![web-dashboard](https://raw.githubusercontent.com/Dharisd/dharisd.github.io/main/assets/images/part_two.png)
* pushing image detections to db 

#### How?

The Detection object already has a list of features for the detections, So what needs to be done is for the Detection object to also have a list image crops of the detections taken using the Bounding box Data from the model. 

This is done by adding a function to crop the detections and add to the Detections object 

```python
    def _get_im_crops(self, bbox_xywh, ori_img):
        im_crops = []
        for box in bbox_xywh:
            x1,y1,x2,y2 = self._xywh_to_xyxy(box)
            im = ori_img[y1:y2,x1:x2]
            im_crops.append(im)
        return im_crops
```


Then when the vehicle has passed, and is added to the list of tracks to be deleted, 
before deleting , The image crop at the middle index of the img_crop is taken ,

#### Why mid?
* The highest chance of getting cleanest and largest image  

Although it might be possible to calculate the largest bounding box and then choose it, I decided to keep it simple .The array for the crop is converted to a JPEG image, base64 encoded and send to the API,The image data is kept in MongoDB itself

#### Why?
* Easy and the detecions are small in size 
Small change added to API to only send the img data when the img parameter is set to true 
to make the dashboard loads faster, Since only the Detailed Detection page only needs it 

```python
        if orderby == "speed":
            pass_output = list(db["vehiclepasses"].find({},{"img_data":0},limit=50).sort("speed",-1))
        if img:
            pass_output = list(db["vehiclepasses"].find(limit=50).sort("timestamp",-1))
        else:
            pass_output = list(db["vehiclepasses"].find({},{"img_data":0},limit=50).sort("timestamp",-1))
```