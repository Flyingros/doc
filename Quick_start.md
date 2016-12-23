# Quick start with total station

## Pixhawk configuration

Documentation : [http://dev.px4.io](http://dev.px4.io)

Calibration des senseurs : [https://www.youtube.com/watch?v=91VGmdSlbo4](https://www.youtube.com/watch?v=91VGmdSlbo4)

## Working with a total station

Steps : 

* Learn how to build a project with Catkin, tutorial : [http://wiki.ros.org/catkin/Tutorials/create\_a\_workspace](http://wiki.ros.org/catkin/Tutorials/create_a_workspace)
* Learn how to publish data : [http://wiki.ros.org/ROS/Tutorials/WritingPublisherSubscriber\(c%2B%2B\)](https://www.gitbook.com/book/flyingros/flyingros-doc/edit#)
* Get the data from the total station
* Send the data to the Pixhawk by sending it via the topic `/mavros/vision_position/pose` \(at 7Hz!\) with the data type `geometry_msgs::PoseStamped`
* Check the `ATT_EXT_HDG_M` param on the Pixhawk and set it to 0. \(No external heading\(yaw\) estimation\)
* Fly thtough `control_thread.py` : [https://github.com/AlexisTM/flyingros/blob/master/flyingros\_nav/nodes/control\_thread.py](https://github.com/AlexisTM/flyingros/blob/master/flyingros_nav/nodes/control_thread.py)





