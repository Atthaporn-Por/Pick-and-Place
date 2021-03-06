# Kinematics Pick & Place

[//]: # (Image References)

[image1]: ./misc_images/misc1.png
[image2]: ./misc_images/misc3.png
[image3]: ./misc_images/misc2.png
[3DTheta123]: https://github.com/BrunoEduardoCSantos/Pick-and-Place/blob/master/misc_images/theta13D.png
[law of cos sin]: http://www2.clarku.edu/~djoyce/trig/laws.html
[2D perspective]: https://github.com/BrunoEduardoCSantos/Pick-and-Place/blob/master/misc_images/drawRobotic.png
[angle and parallel lines rule]: https://www.mathplanet.com/education/pre-algebra/introducing-geometry/angles-and-parallel-lines

![robotic arm](https://github.com/BrunoEduardoCSantos/Pick-and-Place/blob/master/misc_images/SteadyCase.PNG) 

### Kinematic Analysis
#### 1. Evaluation kr210.urdf.xacro file to perform kinematic analysis of Kuka KR210 robot and derive its DH parameters.
From URDF file it is obtained the following offsets between joints:

Joint Name | Parent Link |Child Link | x(m) | y(m) | z(m) | roll | pitch | yaw
--- | --- | --- | --- | ---| ---|---|---|---
Joint_1| base_link |link_1  | 0     |0  |0.33   |0 |0 |0  
Joint_2| link_1    | link_2 |  0.35 | 0 |  0.42 |0 |0 |0 
Joint_3| link_2    | link_3 | 0     | 0 | 1.25  |0 |0 |0 
Joint_4|  link_3   | link_4 |0.96   | 0 | -0.054|0 |0 |0 
Joint_5| link_4    | link_5 | 0.54  |0  | 0     |0 |0 |0 
Joint_6| link_5    | link_6 | 0.193 | 0 | 0     |0 |0 |0 
gripper_joint |link_6| gripper_link|0.11|0 | 0     |0 |0 |0   

Using the previous URDF parameters it is possible to obtain DH parameter table:

Links | alpha(i-1) | a(i-1) | d(i-1) | theta(i)
--- | --- | --- | --- | ---
0->1 | 0 | 0 | 0.75 | q1
1->2 | - pi/2 | 0.35 | 0 | -pi/2 + q2
2->3 | 0 | 1.25 | 0 | q3
3->4 |  -pi/2 | -0.054 | 1.50 | q4
4->5 | pi/2 | 0 | 0 | q5
5->6 | -pi/2 | 0 | 0 | q6
6->EE | pi/2 | 0 | 0.303 | 0


#### 2. Derivation of individual transformation matrices about each joint using DH parameters

The general expression to each joint transformation matrix is:
```
T = [[        cos(θ),       -sin(θ),       0,         a],
      [ sin(θ)*cos(α), cos(θ)*cos(α), -sin(α), -sin(α)*d],
      [ sin(θ)*sin(α), cos(θ)*sin(α),  cos(α),  cos(α)*d],
      [             0,             0,       0,         1]]
```

Using the transformation matrix formula above, here are the joint transformation matrices for the arm:

```
T_0_1 = [[ cos(θ1), -sin(θ1),  0,     0],
          [ sin(θ1),  cos(θ1),  0,     0],
          [       0,        0,  1,  0.75],
          [       0,        0,  0,     1]]
```

```
T_1_2 =  [[ sin(θ2),  cos(θ2),  0,  0.35],
          [       0,        0,  1,     0],
          [ cos(θ2), -sin(θ2),  0,     0],
          [       0,        0,  0,     1]]
```

```
T_2_3 =  [[ cos(θ3), -sin(θ3),  0,  1.25],
          [ sin(θ3),  cos(θ3),  0,     0],
          [       0,        0,  1,     0],
          [       0,        0,  0,     1]]
```

```
T_3_4 = [[ cos(θ4), -sin(θ4),  0, -0.054],
          [       0,        0,  1,    1.5],
          [-sin(θ4), -cos(θ4),  0,      0],
          [       0,        0,  0,      1]]
```

```
T_4_5 = [[ cos(θ5), -sin(θ5),  0,      0],
          [       0,        0, -1,      0],
          [ sin(θ5),  cos(θ5),  0,      0],
          [       0,        0,  0,      1]]
```

```
T_5_6  = [[ cos(θ6), -sin(θ6),  0,      0],
          [       0,        0,  1,      0],
          [-sin(θ6), -cos(θ6),  0,      0],
          [       0,        0,  0,      1]]
```
 

#### 3 Generalized homogeneous transform between base_link and gripper_link using only end-effector
The transformation matrix between base_link and end-effector is: 
```
R_rpy = [[1.0*(sin(phi2)*cos(phi3) - sin(phi3))*cos(phi1), 1.0*(-sin(phi2)*cos(phi3) + sin(phi3))*cos(phi1),- sin(phi3))*sin(phi1) + 1.0*cos(phi2)*cos(phi3)],
[1.0*(sin(phi2)*sin(phi3) + cos(phi3))*cos(phi1), -1.0*(sin(phi2)*sin(phi3) + cos(phi3))*cos(phi1), cos(phi3))*sin(phi1) + 1.0*sin(phi3)*cos(phi2)],
[1.0*sin(phi1)*cos(phi2),-1.0*sin(phi1)*cos(phi2),-1.0*sin(phi2)]]

```
where phi1 , phi2 and phi3 are respectively roll, pitch and yaw on base_link reference frame. 

#### 4. Derivation of the equations to calculate all individual joint angles

We aim to compute 6 joint angles corresponding to 6 DoF (theta_i, where i= {1,2,3,4,5,6}). For the purpose of computing the first three thetas we are using to use a geometric approach. 
For computing theta_1 let's use the following figure:

![alt_text][3DTheta123] 


Firstly, we need to compute the coordinates of wrist center (WC) from the following expression:

```
w_x = px - (d7*R_rpy[0,2])
w_y = py - (d7*R_rpy[1,2])
w_z = pz - (d7*R_rpy[2,2])

```
where d7 comes from DH table in section 1 , **R_rpy** its the transform matrix from base_link to end-effector obtained in previous sectio and **(px,py,pz)** is the end-effector position.
Using the WC coordinates, the expression for theta1 follows:

```
theta1 = atan2(w_y,w_x)
```
For the computation of theta2, we will need to derive distances A/B/C as well angles a/b. Regarding the distances A and B it follows easily from observing figure [3DTheta123] :

```
A= 1.501
B = sqrt(pow(w_z - d1,2) + pow(sqrt(w_x*w_x+w_y*w_y) - a1,2))
C = 1.25

```

where **d1** and **a1** are DH parameters from section1 ( d1 = 0.75m, a1= 0.35).
The following step is applying the [law of cos sin] and we obtain angles a/b : 

```
a = acos((pow(B,2) + pow(C,2) - pow(A,2))/(2*B*C))
b = acos((pow(C,2) + pow(A,2) - pow(B,2))/(2*C*A))

```

After obtaining **A/B/C** and **a/b** we can obtain a general expression to theta2 from the following the analysis of figure  [3DTheta123] and the following one:

![alt_text][2D perspective] 

As a result, we can deduce that **theta2** using a joint 2 frame , it will be given by:

```
theta2= pi/2 - a - offset
```
where from  figure [2D perspective],  the offset is obtained by:


```
offset = atan2(w_z - d1,sqrt(w_x*w_x+w_y*w_y) - a1)

```

Finally, using [angle and parallel lines rule] and regarding figure [2D perspective] , it follows that:

```
pi/2 - theta3 = b+x 

```
where **theta3** has symmetric value since it is counterclockwise. So, it resuls that,

```
theta3 = pi/2 - (b+x) 

```
where **x** is an offset resulting from robotic arm model design:

```
x = atan2(0.054, 1.5)
```
At this point, we obtained the first three joint angles **theta1,theta2,theta3** which lead us to the orientation of WC (i.e, the rotation matrix transforming from base_link to WC),
```
R_0_3 = T_0_1[0:3,0:3]*T_1_2[0:3,0:3]*T_2_3[0:3,0:3]
```
replacing **theta1**, **theta2** and **theta3** obtained previously it follows :
```
R_0_3 = [[sin(theta2 + theta3)*cos(theta1), cos(theta1)*cos(theta2 + theta3), -sin(theta1)],
[sin(theta1)*sin(theta2 + theta3), sin(theta1)*cos(theta2 + theta3),  cos(theta1)],
[        cos(theta2 + theta3),        -sin(theta2 + theta3),        0]]

```
Following this result, the next step is obtaining the symbolic rotation matrix for **theta4**, **theta5** and **theta6** angles (i.e., orientation of end-effector on WC frame ) :
```
R_3_6 = T_3_4[0:3,0:3] * T_4_5[0:3,0:3] * T_5_6[0:3,0:3] 
= [[-sin(theta4)*sin(theta6) + cos(theta4)*cos(theta5)*cos(theta6), -sin(theta4)*cos(theta6) - sin(theta6)*cos(theta4)*cos(theta5), -sin(theta5)*cos(theta4)], 
[sin(theta5)*cos(theta6), -sin(theta5)*sin(theta6), cos(theta5)]
,[-sin(theta4)*cos(theta5)*cos(theta6) - sin(theta6)*cos(theta4), sin(theta4)*sin(theta6)*cos(theta5) - cos(theta4)*cos(theta6), sin(theta4)*sin(theta5)]]
```
On the other hand, we also have **R_3_6** values using **R_0_3** and **R_corr* :
```
R_3_6 = R_0_3.T*R_rpy =
[[  0.878176399737216, -0.878176399737217, -0.411396675699123],
[-0.0508002512060586, 0.0508002512060586,  0.705249532493766],
[-0.0888965057443953, 0.0888965057443953,   0.57738710770248]]

```
From algebric simplification, we simpify the following expressions to obtain **theta4**, **theta5** and **theta6** ,
```
tan(theta4) = R_3_6[2,2]/ (-R_3_6[0,2])
```

```
tan(theta5)= sqrt(pow(R_3_6[2,2],2) + pow(R_3_6[0,2],2))/ R_3_6[1,2]
```

```
tan(theta6) =  (-R_3_6[1,1])/R_3_6[1,0]
```
which resuls that:
```
theta4 = atan2(R_3_6[2,2], -R_3_6[0,2])
theta5 = atan2( sqrt(pow(R_3_6[2,2],2) + pow(R_3_6[0,2],2)) , R_3_6[1,2])
theta6 =  atan2(-R_3_6[1,1], R_3_6[1,0])
```

### Project Implementation

#### 1. Implementation of inverse kinematics on 'IK_server.py'

The first steps were the implementation of the following computations:

- forward kinematics 
- correction matrix from URDF to DH 
- rotation matrix transforming from base_link to end-effector reference frame
- rotation matrix transforming from base_link to WC reference frame

The previous computations were made outside the loop over robotic positions in order to save computation time. 

Inside the end-effector positions loop the following calculation were evaluated:

- WC coordinates
- theta1, theta2, theta3, theta4, theta5, theta5

The average time computation per end-effector position was less thant one second.

In order to reduce the end-effector position error the non-singular cases computing **theta4**, **theta5**, **theta6**. For instance, when **sin(theta5)>0** or **sin(theta5)<0** we will two different solutions for the previous joint angles. 
On the other hand, when **theta5~0**,i.e., we will have **r13 = r23 = r31 = r32 = 0** and it will result in a infinite number of solutions. 
From the two previous cases we would need to define conditions to cover these cases.

![robotic arm2](https://github.com/BrunoEduardoCSantos/Pick-and-Place/blob/master/misc_images/pickobject.PNG) 


