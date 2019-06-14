# Project: Building an Estimator

## Required Steps for a Passing Submission:
1. Sensor Noise GPS and Accelerometer (scenario 6)
2. Attitude Estimation, the Complementary Filter (scenario 7)
3. Prediction Step (scenario 8,9)
4. Magnetometer Update (scenario 10)
5. Closed Loop + GPS Update (scenario 11)

## [Rubric](https://review.udacity.com/#!/rubrics/1643/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.


## 1. Sensor Noise GPS and Accelerometer (scenario 6)
This task involves the following:

1. Record data for GPS X position and Accelerometer X position
2. Calculate standart deviation for both, and set parameters in the config file

|               |  Quad.GPS.X   |  Quad.IMU.AX    |
|:-------------:|:-------------:|:---------------:|
|# measurements | 99            | 2000            |
|max(time)      |  9.905373     |   10.000411     |               
|Mean           |  0.0004833434 |   -0.0155863395 |             
|stdev.p        |  0.7091878283 |    0.4873281339 |              
|stdev          |  0.71279695   |    0.4874500117 |               


## 2. Attitude Estimation,Improve the Complementary filter (scenario 7)
### 2.1. UpdateFromIMU

As described in the section 7.1.2 of Estimation for Quadrotors, the pitch, roll and yaw angles were converted to a quaternion, qt.

Then, the IntegrateBodyRate function was called with the gyro and dtIMU parameters. The gyro provides the body rates pqr, and the dtIMU provides the delta time.

## 3. Prediction Step (scenario 8,9)

### 3.1. PredictState
This is the transition step. The function PredictState() calculates the new values for the elements of the state vector, except the yaw.

x = x + x_dot * dt
y = y + y_dot * dt
z = z + z_dot * dt
x_dot = x_dot + acc.x * dt
y_dot = y_dot + acc.y * dt
z_dot = z_dot + acc.z * dt

The acc must be in the world frame, and the input, accel is in the body frame. The function converts estimated roll, pitch and yaw to quaternion, then using the Rotate_BtoI function of the quaternion, it converts the accel in tho the world frame. 

### 3.2. GetRbgPrime
Returns the derivative of the Rbg as described in the section 7.2. of the Estimation for Quadrotors.

### 3.3. Predict
Calculates the gPrime matrix, starting with the identity matrix. The elements (1,4), (2,5), (3, 6) are changed to dt. And the elements (4,7), (5,7), (6, 7) are set to the elements of the vector obtained by multiplying the RgbPrime and the control input, u.


### 3.4. Parameters
To have covariance that grows as much as the data, the parameters were selected as follows 
* QPosXYStd = .1
* QVelXYStd = .25


## 4. Magnetometer Update (scenario 10)
### 4.1. QYawStd was set to 0.1 

### 4.2. UpdateFromMag
This function defines the parameters for the Update function. z and R_Mag was already defined. hPrime is [0, 0, 0, 0, 0, 0, 1] as given in the section 7.3.2. of the Estimation for Quadrotors.

zFromX could be obtained from ekfState(6). However, the difference between zFromX and z was important. If the absolute difference was greater than PI, then it was required to subtract 2PI from or add 2PI to zFromX.

### 4.3. QYawStd was set to 0.09

## 5. Closed Loop + GPS Update (scenario 11)
### 5.1. Parameters
in config/11_GPSUpdate.txt file;
* Quad.UseIdealEstimator was set to 0 
* #SimIMU.AccelStd = 0,0,0 (comment out)
* #SimIMU.GyroStd = 0,0,0 (comment out)

### 5.2. UpdateFromGPS
Similar to the UpdateFromMag function. 
hPrime is a 6x7 matrix all elements are zero, except (1, 1), (2, 2), (3, 3), (4, 4), (5, 5), (6, 6) elements. And they are 1. According to the section 7.3.1. of the Estimation for Quadrotors.

zFromX could be obtained from the ekfState vector, the first six elements.

z and R_GPS was already defined.
