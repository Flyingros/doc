Implement a MSF Node
=======================

If your needs are not in the given nodes, you can implement yours. This example is to implement a dual position node integration.

Please check [AlexisTM/ethzasl_msf](https://github.com/AlexisTM/ethzasl_msf/tree/master/msf_updates/src/dual_position_msf) for more nodes

Data
----------

* IMU (from the Pixhawk, a bluebird or others)
* Position (x,y,z)
* Second position (x,y,z) 

Involved files and actions
----------

* `msf_updates/CMakeLists.txt` - Add the compilation of the ConfigServer file, add a directory to build
* `msf_updates/cfg/DualPositionSensor.cfg` - Create a Config Server File
* `msf_updates/src/dual_position_msf/CMakeLists.txt` - Create your build options
* `msf_updates/src/dual_position_msf/main.cpp` - Pretty basic, simply instanciate your msf class
* `msf_updates/src/dual_position_msf/msf_statedef.hpp` - Define the states of MSF
* `msf_updates/src/dual_position_msf/dual_position_sensormanager.h` - Implement your class

Lazy start 
-----------

Copy paste `src/position_pose_msf` to `src/dual_position_msf`

`msf_updates/CMakeLists.txt`
-----------

Modify `msf_updates/CMakeLists.txt` modifying `generate_dynamic_reconfigure_options` from 

```
generate_dynamic_reconfigure_options(
    cfg/PosePressureSensor.cfg
    cfg/PositionPoseSensor.cfg
    cfg/SinglePoseSensor.cfg
    cfg/SinglePositionSensor.cfg
    cfg/SphericalPositionSensor.cfg
)
```

to 

```
generate_dynamic_reconfigure_options(
    cfg/PosePressureSensor.cfg
    cfg/PositionPoseSensor.cfg
    cfg/SinglePoseSensor.cfg
    cfg/SinglePositionSensor.cfg
    cfg/SphericalPositionSensor.cfg
    cfg/DualPositionSensor.cfg
)
```

And add a subdirectory to compile 

```
add_subdirectory(src/dual_position_msf)
```

`msf_updates/src/dual_position_msf/CMakeLists.txt`
-----------

```
add_executable(dual_position_sensor main.cpp)

target_link_libraries(dual_position_sensor pose_distorter ${catkin_LIBRARIES})

add_dependencies(dual_position_sensor ${${PROJECT_NAME}_EXPORTED_TARGETS})
```

`msf_updates/cfg/DualPositionSensor.cfg`
-----------

Add the parameters for the second position input and change the name of the config (last line).

**DO NOT FORGET `chmod +x DualPositionSensor.cfg`** else, it won't build.

```python
#! /usr/bin/env python

# Copyright (C) 2012-2013 Simon Lynen, ASL, ETH Zurich, Switzerland
# You can contact the author at <slynen at ethz dot ch>
# Copyright (C) 2011-2012 Stephan Weiss, ASL, ETH Zurich, Switzerland
# You can contact the author at <stephan dot weiss at ieee dot org>
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

PACKAGE='msf_updates'
import roslib; roslib.load_manifest(PACKAGE)

#from driver_base.msg import SensorLevels
from dynamic_reconfigure.parameter_generator_catkin import *

gen = ParameterGenerator()

# @todo Think about levels. Setting most of these to STOP to guarantee atomicity.
# @todo Double check these ranges, defaults

INIT_FILTER     = gen.const("INIT_FILTER",              int_t, 0x00000001, "init_filter")
MISC            = gen.const("MISC",                     int_t, 0x00000002, "misc")

#       Name                 Type      Reconfiguration level                Description   Default   Min   Max
gen.add("core_init_filter",       bool_t,   INIT_FILTER["value"],                    "call filter init using defined scale",                    False)

gen.add("position_noise_meas",      double_t,   MISC["value"],                  "noise for measurement sensor (std. dev)",         0.01,          0,       10)
gen.add("position_yaw_init",    double_t,   MISC["value"],                  "initial value of yaw",         0,          -180,       180)
gen.add("position_delay",         double_t,   MISC["value"],                  "fix delay in seconds",               0.02,       -2.0,     2.0)
gen.add("position_noise_p_ip",      double_t, MISC["value"],                    "propagation: noise p_ip (std. dev)",           0.0,        0,          10.0)
gen.add("position_fixed_p_ip",    bool_t,   MISC["value"],          "fix calibration state position", False)

gen.add("position2_noise_meas",      double_t,   MISC["value"],                  "noise for measurement sensor (std. dev)",         0.01,          0,       10)
gen.add("position2_yaw_init",    double_t,   MISC["value"],                  "initial value of yaw",         0,          -180,       180)
gen.add("position2_delay",         double_t,   MISC["value"],                  "fix delay in seconds",               0.02,       -2.0,     2.0)
gen.add("position2_noise_p_ip",      double_t, MISC["value"],                    "propagation: noise p_ip (std. dev)",           0.0,        0,          10.0)
gen.add("position2_fixed_p_ip",    bool_t,   MISC["value"],          "fix calibration state position", False)

exit(gen.generate(PACKAGE, "Config", "DualPositionSensor"))
```

`msf_updates/src/dual_position_msf/main.cpp`
------------------------

Change the include, the node name and the manager class. 

```c++
/*
 * Copyright (C) 2012-2013 Simon Lynen, ASL, ETH Zurich, Switzerland
 * You can contact the author at <slynen at ethz dot ch>
 * Copyright (C) 2011-2012 Stephan Weiss, ASL, ETH Zurich, Switzerland
 * You can contact the author at <stephan dot weiss at ieee dot org>
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
#include "dual_position_sensormanager.h"

int main(int argc, char** argv) {
  ros::init(argc, argv, "msf_dual_position_sensor");

  msf_updates::DualPositionSensorManager manager;

  ros::spin();

  return 0;
}
```

`msf_updates/src/dual_position_msf/msf_statedef.hpp`
-------------------------------

In the enum, add a second p_ip parameter.

```
enum StateDefinition {  // Must not manually set the enum values!
  p,
  v,
  q,
  b_w,
  b_a,
  p_ip,
  p_ip2 // here
};
```

and a variable assotiated to p_ip2 

```c++
typedef boost::fusion::vector<
    // States varying during propagation - must not change the ordering here for
    // now, CalcQ has the ordering hardcoded.
    msf_core::StateVar_T<Eigen::Matrix<double, 3, 1>, p,
        msf_core::CoreStateWithPropagation>,  ///< Translation from the world frame to the IMU frame expressed in the world frame.
    msf_core::StateVar_T<Eigen::Matrix<double, 3, 1>, v,
        msf_core::CoreStateWithPropagation>,  ///< Velocity of the IMU frame expressed in the world frame.
    msf_core::StateVar_T<Eigen::Quaternion<double>, q,
        msf_core::CoreStateWithPropagation>,  ///< Rotation from the world frame to the IMU frame expressed in the world frame.
    msf_core::StateVar_T<Eigen::Matrix<double, 3, 1>, b_w,
        msf_core::CoreStateWithoutPropagation>,  ///< Gyro biases.                      
    msf_core::StateVar_T<Eigen::Matrix<double, 3, 1>, b_a,
        msf_core::CoreStateWithoutPropagation>,  ///< Acceleration biases.              

    // States not varying during propagation.
    msf_core::StateVar_T<Eigen::Matrix<double, 3, 1>, p_ip>,  ///< Translation from the IMU frame to the position sensor frame expressed in the IMU frame.
    msf_core::StateVar_T<Eigen::Matrix<double, 3, 1>, p_ip2>  ///< here
> fullState_T;
```

Checklist for the sensor manager 
------------

* DEFINE renamed from `POSITION_MEASUREMENTMANAGER_H` to `DUAL_POSITION_MEASUREMENTMANAGER_H` 
* Included all the sensor files (note; for the pressure sensor, you only have to include the sensorhandler as it already include the measurement file)
* Rename the Config file from `SinglePositionSensorConfig` to `DualPositionSensorConfig`
* Rename the class file from `PositionSensorManager` `DualPositionSensorManager`
* Create a typedef and a friend class for each sensor (not for pressure sensor)
* Add the sensor handler 
* Instanciate a sensor handler in private part 
* Add the config for the second position sensor (in Config())
* In the Init function, instanciate all variables from msf_statedef.hpp 
* Add the node param input 

Final sensor manager file `msf_updates/src/dual_position_msf/dual_position_sensormanager.h`
-------------------------------

```c++
#ifndef DUAL_POSITION_MEASUREMENTMANAGER_H
#define DUAL_POSITION_MEASUREMENTMANAGER_H

#include <ros/ros.h>

#include <msf_core/msf_core.h>
#include <msf_core/msf_sensormanagerROS.h>
#include <msf_core/msf_IMUHandler_ROS.h>
#include "msf_statedef.hpp"
#include <msf_updates/position_sensor_handler/position_sensorhandler.h>
#include <msf_updates/position_sensor_handler/position_measurement.h>
#include <msf_updates/DualPositionSensorConfig.h>

namespace msf_position_sensor {

typedef msf_updates::DualPositionSensorConfig Config_T;
typedef dynamic_reconfigure::Server<Config_T> ReconfigureServer;
typedef shared_ptr<ReconfigureServer> ReconfigureServerPtr;

class DualPositionSensorManager : public msf_core::MSF_SensorManagerROS<
    msf_updates::EKFState> {
  typedef PositionSensorHandler<
      msf_updates::position_measurement::PositionMeasurement,
      DualPositionSensorManager> PositionSensorHandler_T;
  friend class PositionSensorHandler<
      msf_updates::position_measurement::PositionMeasurement,
      DualPositionSensorManager> ;
 public:
  typedef msf_updates::EKFState EKFState_T;
  typedef EKFState_T::StateSequence_T StateSequence_T;
  typedef EKFState_T::StateDefinition_T StateDefinition_T;

  DualPositionSensorManager(
      ros::NodeHandle pnh = ros::NodeHandle("~/position_sensor")) {
    imu_handler_.reset(
        new msf_core::IMUHandler_ROS<msf_updates::EKFState>(*this, "msf_core",
                                                            "imu_handler"));

    position_handler_.reset(
        new PositionSensorHandler_T(*this, "", "position_sensor"));
    AddHandler(position_handler_);

    position_handler2_.reset(
        new PositionSensorHandler_T(*this, "", "position_sensor2"));
    AddHandler(position_handler2_);


    reconf_server_.reset(new ReconfigureServer(pnh));
    ReconfigureServer::CallbackType f = boost::bind(
        &DualPositionSensorManager::Config, this, _1, _2);
    reconf_server_->setCallback(f);
  }
  virtual ~DualPositionSensorManager() {
  }

  virtual const Config_T& Getcfg() {
    return config_;
  }

 private:
  shared_ptr<msf_core::IMUHandler_ROS<msf_updates::EKFState> > imu_handler_;
  shared_ptr<PositionSensorHandler_T> position_handler2_;
  shared_ptr<PositionSensorHandler_T> position_handler_;

  Config_T config_;
  ReconfigureServerPtr reconf_server_;

  /**
   * \brief Dynamic reconfigure callback.
   */
  virtual void Config(Config_T &config, uint32_t level) {
    config_ = config;
    position_handler_->SetNoises(config.position_noise_meas);
    position_handler_->SetDelay(config.position_delay);
    position_handler2_->SetNoises(config.position2_noise_meas);
    position_handler2_->SetDelay(config.position2_delay);
    if ((level & msf_updates::DualPositionSensor_INIT_FILTER)
        && config.core_init_filter == true) {
      Init(1.0);
      config.core_init_filter = false;
    }
  }

  void Init(double scale) const {
    if (scale < 0.001) {
      MSF_WARN_STREAM("init scale is "<<scale<<" correcting to 1");
      scale = 1;
    }

    Eigen::Matrix<double, 3, 1> p, v, b_w, b_a, g, w_m, a_m, p_ip, p_ip2, p_vc, p_vc2;
    Eigen::Quaternion<double> q;
    msf_core::MSF_Core<EKFState_T>::ErrorStateCov P;

    // Init values.
    g << 0, 0, 9.81;  /// Gravity.
    b_w << 0, 0, 0;   /// Bias gyroscopes.
    b_a << 0, 0, 0;   /// Bias accelerometer.

    v << 0, 0, 0;     /// Robot velocity (IMU centered).
    w_m << 0, 0, 0;   /// Initial angular velocity.

    // Set the initial yaw alignment of body to world (the frame in which the
    // position sensor measures).
    double yawinit = config_.position_yaw_init / 180 * M_PI;
    Eigen::Quaterniond yawq(cos(yawinit / 2), 0, 0, sin(yawinit / 2));
    yawq.normalize();

    q = yawq;

    P.setZero();  // Error state covariance; if zero, a default initialization
                  // in msf_core is used

    p_vc = position_handler_->GetPositionMeasurement();
    p_vc2 = position_handler2_->GetPositionMeasurement();

    MSF_INFO_STREAM(
        "initial measurement pos:[" << p_vc.transpose() << "] orientation: " << STREAMQUAT(q));
    MSF_INFO_STREAM(
        "initial measurement pos:[" << p_vc2.transpose() << "] orientation: " << STREAMQUAT(q));

    // check if we have already input from the measurement sensor
    if (!position_handler_->ReceivedFirstMeasurement())
      MSF_WARN_STREAM(
          "No measurements received yet from 1 to initialize position - using [0 0 0]");

    if (!position_handler2_->ReceivedFirstMeasurement())
      MSF_WARN_STREAM(
          "No measurements received yet from 2 to initialize position - using [0 0 0]");


    ros::NodeHandle pnh("~");
    pnh.param("position_sensor/init/p_ip/x", p_ip[0], 0.0);
    pnh.param("position_sensor/init/p_ip/y", p_ip[1], 0.0);
    pnh.param("position_sensor/init/p_ip/z", p_ip[2], 0.0);

    pnh.param("position_sensor/init/p_ip2/x", p_ip2[0], 0.0);
    pnh.param("position_sensor/init/p_ip2/y", p_ip2[1], 0.0);
    pnh.param("position_sensor/init/p_ip2/z", p_ip2[2], 0.0);

    // Calculate initial attitude and position based on sensor measurements.
    p = ((p_vc - q.toRotationMatrix() * p_ip) + (p_vc2 - q.toRotationMatrix() * p_ip2))/2;

    a_m = q.inverse() * g;          /// Initial acceleration.

    //prepare init "measurement"
    // True means that we will also set the initialsensor readings.
    shared_ptr < msf_core::MSF_InitMeasurement<EKFState_T>
        > meas(new msf_core::MSF_InitMeasurement<EKFState_T>(true));

    meas->SetStateInitValue < StateDefinition_T::p > (p);
    meas->SetStateInitValue < StateDefinition_T::v > (v);
    meas->SetStateInitValue < StateDefinition_T::q > (q);
    meas->SetStateInitValue < StateDefinition_T::b_w > (b_w);
    meas->SetStateInitValue < StateDefinition_T::b_a > (b_a);
    meas->SetStateInitValue < StateDefinition_T::p_ip > (p_ip);
    meas->SetStateInitValue < StateDefinition_T::p_ip2 > (p_ip2);

    SetStateCovariance(meas->GetStateCovariance());  // Call my set P function.
    meas->Getw_m() = w_m;
    meas->Geta_m() = a_m;
    meas->time = ros::Time::now().toSec();

    // Call initialization in core.
    msf_core_->Init(meas);
  }

  // Prior to this call, all states are initialized to zero/identity.
  virtual void ResetState(EKFState_T& state) const {
    UNUSED(state);
  }
  virtual void InitState(EKFState_T& state) const {
    UNUSED(state);
  }

  virtual void CalculateQAuxiliaryStates(EKFState_T& state, double dt) const {
    const msf_core::Vector3 npipv = msf_core::Vector3::Constant(
        config_.position_noise_p_ip);
    const msf_core::Vector3 npipv2 = msf_core::Vector3::Constant(
        config_.position2_noise_p_ip);

    // Compute the blockwise Q values and store them with the states,
    //these then get copied by the core to the correct places in Qd.
    state.GetQBlock<StateDefinition_T::p_ip>() =
        (dt * npipv.cwiseProduct(npipv)).asDiagonal();
    state.GetQBlock<StateDefinition_T::p_ip2>() =
        (dt * npipv2.cwiseProduct(npipv2)).asDiagonal();
  }

  virtual void SetStateCovariance(
      Eigen::Matrix<double, EKFState_T::nErrorStatesAtCompileTime,
          EKFState_T::nErrorStatesAtCompileTime>& P) const {
    UNUSED(P);
    // Nothing, we only use the simulated cov for the core plus diagonal for the
    // rest.
  }

  virtual void AugmentCorrectionVector(
      Eigen::Matrix<double, EKFState_T::nErrorStatesAtCompileTime, 1>& correction) const {
    UNUSED(correction);
  }

  virtual void SanityCheckCorrection(
      EKFState_T& delaystate,
      const EKFState_T& buffstate,
      Eigen::Matrix<double, EKFState_T::nErrorStatesAtCompileTime, 1>& correction) const {
    UNUSED(delaystate);
    UNUSED(buffstate);
    UNUSED(correction);
  }
};
}
#endif  // DUAL_POSITION_MEASUREMENTMANAGER_H
```