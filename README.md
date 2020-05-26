# docker_dope

A docker image of the deep object pose estimation (DOPE) method.

The code, docker image, and the instructions are adapted from `https://github.com/NVlabs/Deep_Object_Pose`.

## Start your camera node on the loacal machine
The command depends on what camera you are using.
For example, `roslaunch zed_wrapper zedm.launch`.

The camera must publish a correct `camera_info` topic to enable DOPE to compute the correct poses.
Basically all ROS drivers have a `camera_info_url` parameter
where you can set the calibration info (but most ROS drivers include a reasonable default).

## Edit config info in `~/catkin_ws/src/dope/config/config_pose.yaml`
* `topic_camera`: RGB topic to listen to
* `topic_camera_info`: camera info topic to listen to
* `topic_publishing`: topic namespace for publishing
* `input_is_rectified`: Whether the input images are rectified. It is strongly suggested to use a rectified input topic.
* `downscale_height`: If the input image is larger than this, scale it down to this pixel height. Very large input images eat up all the GPU memory and slow down inference. Also, DOPE works best when the object size (in pixels) has appeared in the training data (which is downscaled to 400 px). For these reasons, downscaling large input images to something reasonable (e.g., 400-500 px) improves memory consumption, inference speed *and* recognition results.
* `weights`: dictionary of object names and there weights path name, **comment out any line to disable detection/estimation of that object**
* `dimensions`: dictionary of dimensions for the objects  (key values must match the `weights` names)
* `class_ids`: dictionary of class ids to be used in the messages published on the `/dope/detected_objects` topic (key values must match the `weights` names)
* `draw_colors`: dictionary of object colors (key values must match the `weights` names)
* `model_transforms`: dictionary of transforms that are applied to the pose before publishing (key values must match the `weights` names)
* `meshes`: dictionary of mesh filenames for visualization (key values must match the `weights` names)
* `mesh_scales`: dictionary of scaling factors for the visualization meshes (key values must match the `weights` names)
* `thresh_angle`: undocumented
* `thresh_map`: undocumented
* `sigma`: undocumented
* `thresh_points`: Thresholding the confidence for object detection; increase this value if you see too many false positives, reduce it if  objects are not detected.

## Start DOPE node
Download [the weights](https://drive.google.com/open?id=1DfoA3m_Bm0fW8tOWXGVxi4ETlLEAgmcg) and save them to the `weights` folder, i.e., `~/catkin_ws/src/dope/weights/`.
```roslaunch dope dope.launch [config:=/path/to/my_config.yaml]  # Config file is optional; default is `config_pose.yaml` ```

## Debugging

* The following ROS topics are published (assuming `topic_publishing == 'dope'`):
    ```
    /dope/webcam_rgb_raw       # RGB images from camera
    /dope/dimension_[obj_name] # dimensions of object
    /dope/pose_[obj_name]      # timestamped pose of object
    /dope/rgb_points           # RGB images with detected cuboids overlaid
    /dope/detected_objects     # vision_msgs/Detection3DArray of all detected objects
    /dope/markers              # RViz visualization markers for all objects
    ```
    *Note:* `[obj_name]` is in {cracker, gelatin, meat, mustard, soup, sugar}

* To debug in RViz, run `rviz`, then add one or more of the following displays:
    * `Add > Image` to view the raw RGB image or the image with cuboids overlaid
    * `Add > Pose` to view the object coordinate frame in 3D.
    * `Add > MarkerArray` to view the cuboids, meshes etc. in 3D.
    * `Add > Camera` to view the RGB Image with the poses and markers from above.

    If you do not have a coordinate frame set up, you can run this static transformation: `rosrun tf2_ros static_transform_publisher 0 0 0 0.7071 0 0 -0.7071 world <camera_frame_id>`, where `<camera_frame_id>` is the `frame_id` of your input camera messages.  Make sure that in RViz's `Global Options`, the `Fixed Frame` is set to `world`. Alternatively, you can skip the `static_transform_publisher` step and directly set the `Fixed Frame` to your `<camera_frame_id>`.

* If `rosrun` does not find the package (`[rospack] Error: package 'dope' not found`), be sure that you called `source devel/setup.bash` as mentioned above.  To find the package, run `rospack find dope`.
