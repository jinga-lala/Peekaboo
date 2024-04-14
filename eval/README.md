## Evaluation for Controllable Video Generation

### Datasets
We provide four datasets for evaluating Control in video generation methods. These datasets are - 
- **Something-something v2-Spatio-Temporal (ssv2-ST):** We use Something-Something v2 dataset to obtain the generation prompts and ground truth masks from real action videos. We filter out a set of 295 prompts. The details for this filtering are in the appendix. We then use an off-the-shelf *OWL-ViT-large* open-vocabulary object detector to obtain the bounding box (bbox) annotations of the object in the videos. This set represents bbox and prompt pairs of real-world videos, serving as a test bed for both the quality and control of methods for generating realistic videos with spatio-temporal control.

- **Interactive Motion Control (IMC):** We also curate a set of prompts and bounding boxes which are manually defined. We use GPT-4 to generate prompts and pick a set of 34 prompts of objects in their natural contexts. These prompts are varied in the type of object, size of the object, and the type of motion exhibited. We then annotate 3 sets of bboxes for each prompt, where the location, path taken, speed, and size are varied. This set of 102 prompt-bbox pairs serve as our custom evaluation set for spatial control. Note that since ssv2-ST dataset has a lot of inanimate objects, we bias this dataset to contain more living objects. This dataset represents possible input pairs that real users may generate.

- **LaSOT:** We repurpose a large-scale object tracking dataset --LaSOT-- for evaluating control in video generation. This dataset contains prompt-bbox-video triplets for a large number of classes. The videos have frame level annotations specifying the location of the object in the video. We subsample the videos to 8 FPS and then randomly pick up 2 clips per video from the test set of this dataset. This gives us 450 total clips across 70 different object categories.

- **DAVIS-16:** DAVIS-16 is another video object segmentation dataset that we consider. We take videos from its test set, manually annotating them with prompts. We use the provided segmentation masks to create input bboxes. This gives us 40 prompt-bbox pairs in total, where each video has a different subject.

### File structure
For each of these datasets, we provide a json file, which has multiple records, each record being a dictionary. Each dictionary has the following keys - 
- **id** - The id of the video
- **caption** - The caption for generating this video
- **subject** - The main subject of the video, a subset of the caption
- **bboxes** - A list of 24 values, where each value is (x_1, y_1, x_2, y_2) of the relative co-ordinates of the bounding box. 

Additionaly, there is also other metadata present, including the id of the video in the original dataset.


### Metrics
We use the following metrics for evaluation - 
- **Coverage:** This metric represents the fraction of generated videos where OWL-ViT detects bounding boxes (bboxes) in more than 50% of the frames. It indicates the model's ability to generate videos that include detectable objects across a majority of frames.

- **Intersection-over-Union (mIoU):** After filtering out videos where less than 50% of frames have a detected bbox, we calculate the Intersection-over-Union of the detected bboxes with the input mask for these filtered videos. The mean IoU (mIoU) scores for each method are reported, offering a measure of the generation method's spatio-temporal control by assessing how well the detected objects align with the specified input masks.

- **Centroid Distance (CD):** This metric calculates the distance between the centroid of the generated object and the input mask, normalized to 1. It serves as an indicator of the generation method's control over the location of the generated object within the video.

- **Average Precision@50% (AP50):** The average precision at 50% threshold (AP50) of the detected and input bboxes averaged over all videos is reported. For frames where the object is present, AP50 quantifies the spatial control the method provides. It evaluates how precisely the generated object's location matches the input specification and penalizes frames where the object is not detected accurately.

### Evaluation
To run the evaluation script, run
```
python get_metrics.py --prediction_path /path/to/predictions --gt_path /path/to/ground_truth
```
We expect predictions to be a pickle file containing bbox annotations for your generated videos. The structure of the data is similar to the ground truth json files provided, but `bboxes` need to be numpy arrays of size `(num_frames x h x w)` with masks being denoted by 1 and other values being 0.


To generate bounding box annotations for your videos, you can use the `create_bbox.py` script provided. 
