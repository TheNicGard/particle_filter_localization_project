# Particle Filter Localization Project
### Team members: Nic Gard, Karhan Kayan
#### Implementation Plan:
* The particle cloud will be initialized using uniformly distributed approximately 200-500 points across a field with dimensions 50 x 50. The exact number of points, and the size of the field, are subject to change based on testing, and the initial number of particles initialized will be smaller for the sake of testing. When testing `initialize_particle_cloud`, the output will be checked manually to determine that no large groups of particles form.
* The positions of the particles will be updated by taking the movement information of the robot (i.e. how much the robot is expected to have moved in 1 tick, whether that be a change in position or angle) and finding the new position of each particle using that movement information. `update_particles_with_motion_model` will be tested by creating a list of movements, applying them to the list of particles, and verifying that their new positions is correct. When updating the movements, Gaussian noise will be added. 
* Importance weights: `update_particle_weights_with_measurement_model` will be implemented using a likelihood field algorithm to estimate hypothetical particle measurement readings, and the same weight assignment function that we used in class will be used to update the weights. In other words, we will use 1/l1 distance between the likelihood field measurement and the actual measurement. 
* Normalizing the particles' importance weights can be done by summing their weights, getting the inverse of this sum, and multiplying this with each particle's importance weight. This will give us a probability distribution on particles for the resampling step. Resampling the particles will be done using Python's `random.choice`, which is able to select elements in a list using probability weights. Testing of `normalize_particles` can be done by verifying the result of the calculation, and testing of `resample_particles` can be done by running multiple trials, and determining that a weighted choice is able to select a particle with the probability that its weight suggests (e.g. if a particle has an importance weigt of 0.1, 100 trils should show that it is selected approximately 10 times).
* The estimated pose of the robot will be updated by averaging the position and angles of all of the particles. `update_estimated_robot_pose` can be tested by simply verifying that the robot's udpated position is correct using different lists of updated particles, before and after a state change.
* Noise: Noise will be added in `update_particles_with_motion_model`, where some random amount of noise is added to each particle's movement and rotation. The noise we will use will be Gaussian noise with mean 0 and a small variance. We will determine the appropriate amount of variance through trial and error. The `np.random.normal()` function is appropriate to generate this noise. 
#### Timeline:
* Fri. April 16th - finish writing/testing `initialize_particle_cloud`
* Sat. April 17th - finish writing/testing `update_particles_with_motion_model`
* Mon. April 19th - finish writing/testing `update_particle_weights_with_measurement_model`
* Wed. April 21st - finish writing/testing `normalize_particles` and `resample_particles`
* Fri. April 23rd - finish writing/testing `update_estimated_robot_pose`
## Writeup:
**Objectives:**
Describe the goal of this project. The goal of this project is to determine the location of Turtlebot3 robot (i.e. performing localization) within a provided house map using map data and the robot's LiDAR. This is done using Monte Carlo localization, where several particles are placed throughout the map, and using the robot's own sensor data, their position and orientation is adjusted to more closely match the robot's own position.

**Design Description:** The algorithm makes several "guesses" of where the robot is located, represented by particles holding information on position and orientation. The particles used to guess the robot's position are initialized by placing a particle on every unnoccupied location on the map, according to the map's coordinates. When the robot moves some distance above a threshold (either translated or rotated), each particle is moved the same amount the robot moved, plus some noise to account for uncertainty. The weight of each particle is computed using `compute_likelihood`, the Likelihood Fields for Range Finders algorithm provided during class. Particles are resampled according to their weights, up to a predetermined number of particles. The robot's estimated position and orientation is the average position and orientation of all particles. This algorithm updates whenever the robot moves more than the predetermined threshold.

**Code Implementation:**
  * Initialization of particle cloud: this is implemented in `initialize_particle_cloud()`, `normalize_particles()`. `initialize_particle_cloud()` gets the dimensions of the map, and iterates over every unoccupied point on the map's surface. If the point is unoccupied, the function adds the point to the `inside_house` list, which is the list of unoccupied points. Then, the function adds `num_particles` amount of particles at the points drawn uniformly from that list. The function adds a particle to the particle array with a random rotation around the z-axis (so that the particle faces away paralell to the ground) and with a weight of 1. `normalize_particles()` normalizes the weights of the particles such that the sum of their weights is equal to one.
  * Movement model: This is implemented in `update_particles_with_motion_model()`. Also helper functions `get_theta()` and `set_theta` were added to the particle class to make it easier to work in angles instead of quaternions. The movement model calculates the current and previous poses of the robot and adds the difference vectors to all of the particles. When adding these vectors a uniform noise is added to both x,y, and yaw. `get_theta()` and `set_theta()` are used to update the angle (yaw) of a particle. 
  * Measurement model: The measurement model is implemented in `update_particle_weights_with_measurement_model()` and `compute_likelihood()`. Our measurement model uses the likelihood field algorithm discussed and implemented in class. The first function iterates through each particle and calculates the likelihood of that particle based on the data by calling the second function. `compute_likelihood()` is used to calculate the likelihood of a given particle based on the algorithm given in class. The cardinal directions that are used were 45 degrees apart to both accomodate for precision and speed. 
  * Resampling: This is done in `resample_particles()` using `draw_random_sample()`. The second function uses `np.random.choice()` to sample with replacement from a given list where the probabilities are provided. Since we are sampling `particle` objects, deep copies are created before sampling. The probabilities and the particles are provided in `resample_particles()` where the probabilities are the weights of the particles calculated by the likelihood field algorithm in the previous step. 
  * Incorporation of noise: The noise is incorporated in the movement model in the function `update_particles_with_motion_model()`. We add uniform noise to each particles movement on x, y coordinate and the yaw of the particle. The uniform noise is generated using `np.random.uniform()`. 
  * Updating estimated robot pose: This is in `update_estimated_robot_pose()`. The helper function `get_theta()` is also used, which was explained above. The robot's pose is calculated by the average poses of all particles in the cloud. The averages are calculated for x,y, and theta, which are then set to be the estimated pose of the robot. 
  * Optimization of parameters: The parameters optimized were the standard deviation of the noise for the movement model `sd` and the number of particles `num_particles`. The noise was optimized based on the behaviour of the particle cloud after a couple of resampling steps. `num_particles` was optimized to balance the tradeoff between runtime speed and estimated pose accuracy.
  
**Challenges:**
The first challenge we faced was publishing the particles seemed difficult at first, because calling `publish_particle_cloud` at the end of initialization caused the particles to not immediately appear, until we figured out that we had to publish the particle poses repeatedly. Another map-related issue that came up was how to generate the particle cloud inside the house. We overcame this problem by using `self.map.data` to check which points were grey. In the likelihood model, using every single angle of the lidar caused the weights to be 0, which messed with the sampling step. Furthermore, it made the model run slow. So, we only used angles 45 degrees apart. In the resampling step, resampling was done with replacement, which caused a change to one particle to reflect on others of the same kind. This was overcome by creating deep copies of the particles when resampling, so that each are treated differently. Another challenge was that we saw that using Gaussian noise for the movement model caused the model to run slowly, so we used a uniform noise model.

**Future Work:**

Optimizing the parameters more could be a good idea. Trying to find the balance of having more cardinal directions can be one method. This will make the model slower, but it may increase the accuracy if the number of directions are not too high. Furthermore, using numpy arrays instead of lists to store certain properties of particles could be a good idea, such as the weights and the positions. This is because numpy methods are much faster compared to plain python loops since they use C++ core functionalities. This would come in handy when normalizing particles, calculating averages, updating positions, etc. Another improvement could be adding a small amount of new particles after each resampling step. This would help if we get unlucky in the first couple of resamples and have no particles close to the robots position. 

(takeaways, 2 bullet points min.)
