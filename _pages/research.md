---
permalink: /research/
title: ""
author_profile: true
redirect_from: 
  - /research/
  - /research.html
---

# Automatic Annotation for Semantic Segmentation in Indoor Scenes

- Expensive to get image semantic labels manually, so it's necessary to generate image semantic annotation automatically without any ground truth labels. Using together with human annotation, we hope we can get a better segmentation (e.g. FCN) model
- Structural scene understanding separates an image to 2 parts rst: foreground and background. We use
  Mask RCNN to detect foreground objects and 3D layout segmentation to recognize background information separately.
- We gather this two information together using a well-dened energy function to nd best annotation.
- Now we are trying to nd a way 'ne tuning' our Mask RCNN detector without access to any ground truth
  labels to improve the result



# Few Shot Image Classication with Multiple Image Views

- Idea: instead of seeing thousands of chair images, we think humans learn to recognize a chair by seeing a chair from multiple views and generating a 3D chair model in their memory
- Based on Geometry-Aware Recurrent Network [link], our model uses RGB input to generate a 3D feature tensor and updates a 3D GRU memory at each image view
- Experiments based on CORE50 Dataset which only contains multiple view images for 50 objects, 10 classes in all and 5 for each class



# Unsupervised Object Detection using VAE

- Idea: structural image understanding sees one object at a time and joins them together to get a big picture.
- Try to do object detection on special raw images (e.g. MINST) in a unsupervised way without ground truth.
- Based on AIR [link], our model uses an RNN to encode one object at a time and learns to gure out how many objects there are in this image so that we can get variable-length latent representation for different images
- Use VAE to re-generate the image to get training signal



# Link Prediction on Weighted Signed Social Network (WSN)
- Focus on link prediction problems in Weighted Signed Social Network (WSN)
- Come out an algorithm called MFLG, a new network embedding algorithm based on matrix factorization. We embed each node with two vecotrs, one subjective one objective
- We use random work to get related node pairs which aren't connected directly to combine global social network features with local features (adjacent node pairs)
- We add regularization to get correct sentiment results when two predictions have same square error (e.g. when answer is 1, choose 3 instead -1)
- Use different algorithms' prediction results as features to train a linear regression model and get a more robust model
- Summarize our work into a paper as the third author (waiting for notification)



# P2P chatroom on LAN

- It's a basic P2P online chatroom based on OMCS framework. It can achieve communication through text, audio and video.