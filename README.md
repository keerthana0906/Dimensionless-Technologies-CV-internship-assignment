# Dimensionless-Technologies-CV-internship-assignment
This repository contains the assignment question given by Dimensionless Technologies as a part of selection process for the Computer Vision/Image Processing Internship

The problem statement is present in the pdf document  

background_images directory       : has pictures of X-ray baggage images given   
threat_images directory           : has pictures of the threat objects given  
sample_output_images directory    : has pictures of the sample output provided 
cropped_threats directory         : has the pictures of the cropped threat objects and the masks obtained for them  
final_output directory            : contains the final output images obtained by me  

Logic used for cropping and obtaining masks for threat objects :    
	The threat images have been read and converted into binary images using cv2 library. The threashold for doing this has been put 245 because all the background of threat images is having white pixels while some pixels values on the threat object were so low. So the threshold has been kept so high so that the white backgound will anyway be converted into dark as its pixel values will lie above 250 and the light pixels on objects will get detected. After converting it into binary image , the contours for this binary image were found using the findContours() inbuilt function. Now out of all these countours found , the contour with the largest arear is found. The largest contour will be the one that will bound entire threat object as it will be the largest object present in that entire image.Now I obtained the bounding Rectangle for this largest contour and cropped that bounding Rectangle out of the image and stored it into cropped_images folder.
 
 Logic used to obtain mask for the cropped threat objects :  
 	For each cropped threat image, binary image has been obtained with same threshold as said before (245). Now this binary image has been subjected to Morphological opening operation so that it will get rid of white pixel noises present in the boundaries of the threat object. Sample is shown below. The first picture shows the binary image before applying opening operation and second shows the image after applying opening operation
	
![g](https://user-images.githubusercontent.com/76999699/162389169-d9303c68-b79a-48c7-b4d0-d70863a2ecb4.jpg)
![t3_threat_mask](https://user-images.githubusercontent.com/76999699/162387015-193366d2-06cd-458c-bdce-b97a30be668d.png)  
  
  After applying the morphological opening , the contours have been found for the image and the contour with the largest arear has been found same as before. The bounding rectangle for the largest contour has been obtained . For each point in the bounding rectangle , it was verified whether that point lies inside of the largest contour or outside using the inbuilt pointPolygonTest(). If the point lies inside of the contour and it is in black , then I converted it into white pixel. This is done inorder to achieve perfect mask so that there are no gaps left inside threat object while we paste it on background images. Dilation operations could have been perofrmed to achieve this , but the dilation takes more than 1 iteration and will lead to loss in perfect boundary shape of the threat object . So this method was implemented instead of dialtion to make mask more precise. A sample image has been shown for the mask before this operatino and after this operation
  
![h](https://user-images.githubusercontent.com/76999699/162391202-15878b49-e67a-4cec-b021-3c9b05270a01.jpg)
![t4_threat_mask](https://user-images.githubusercontent.com/76999699/162391271-262a8219-3fd7-4899-aa95-c93083c3137f.png)  

Now the mask for each threat object is ready and it is saved into the cropped_threats folder  

Logic for finding scale factor and position where threat object has to be pasted :  
	The background_images which have the X-ray bag images were convereted into binary image and the largest contour was found similary as done before. Now the largest contour gives the baggage contour. We have found the bounding rectangle for this baggage. So now we have the pixel locations where the baggage will be present.  
	Now we have both the area of the bag and area of threat object. We were asked to scale down threat objects and rotate them by 45 degrees. So for finding the scale factor by which we need to scale down the threat , I have made a condition that the area of the threat object must be less than 1/6th of the bag area (just a common intution). If the threat object area is not less that 1/6th of bag area the we need to scale it down by (bag_area)/(6*threat_area). SO in this case the scale_factor is set to (bag_area)/(6*threat_area). But if the area of threat is less than 1/6th of bag area, then I set default scle_factor to be 0.7 as we were specifiaclly asked to scale down the object. Along with the object their respective masks also were scaled down by the same factor. Now both the object and mask were rotated by 45 degrees.
	Now we need to know which location on background image to paste this threat. For this , we already know the bounding box for the baggage that we obtained previously.So if we paste the threat at the center of this bounding box , the chances for it to come out of the baggage boundary are very less. As we have x,y,width and height of the bounding box of baggage (y+y+height/2 , x+x+width/2)  will give center point of the the bounding box for the baggage.Again here we are using Image.paste method to keep the threat on baggage, inside trans_blend function. The Image.paste() method will paste the 2nd image from top,left corner i.e (0,0) cordinate of second image on the the given input location of first image . So if we try pasting threat object on baggage by passing cordinates as (y+y+height/2 , x+x+width/2) , there might be a possibility that it will come out of baggage from bottom edge. SO to avoid this I tried pasting center of threat object on center of baggage bounding box which will reduce the possibilities of threat popping out of the bag to a great extent. So inorder to achieve this we need to translate the (y+y+height/2 , x+x+width/2) cordinate by (-height_of_threat /2 , -width_of_threat /2 ). This will give us the final cordinate of 
	((y+y+h)/2-(height_of_threat/2),(x+x+w)/2-(width_of_threat/2))
	This scaled down and rotated threat along with its mask ,background image and cordinates to paste are sent into trans_blend function to obtain final output.  

Logic used for trans_blend function : 
	Image_paste method directly pastes the threat onto baggae without adding any translucency. So inorder to bring in translucency , first a trsnsparent image of size of the background image (baggage image) has been made and the threat object has been pasted on to this transparent image using it's mask. Image.blend() option combines two equal sized images with a transparency factor which we can control. But the problem with this is that it makes both the images transparent to some extent based on the input parameter we give . If we give 0.7 then the first image will be shown with 70 % of its brightness and second image will be blended with 30% of its brightness.So inorder to keep the background images more bright and opaque , I made two images , one having the blend of baggage and threat object with 65% threat object and other having blend of threat and bag with 100% baggage and then did alpha_composite of both to lay threat on baggage. Example has been shown below

![i](https://user-images.githubusercontent.com/76999699/162408530-5e52b944-360c-4ff1-a82e-037ad91b755f.png)
![j](https://user-images.githubusercontent.com/76999699/162408570-e240e407-cd51-4327-9c25-1b9c9c7688de.png) 
![k](https://user-images.githubusercontent.com/76999699/162408611-571088c9-249f-4c5b-9573-e842434c6962.png)  



	
