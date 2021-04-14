# Particle Filter Localization Project
### Team members: Nic Gard, Karhan Kayan
#### Implementation:
* The particle cloud will be initialized using uniformly distrubted approximately 200-500 points across a field with dimensions 50 x 50. The exact number of points, and the size of the field, are subject to change based on testing, and the initial number of particles initialized will be smaller for the sake of testing. When testing `initialize_particle_cloud`, the output will be checked manually to determine that no large groups of particles form.
* The positions of the particles will be updated by taking the movement information of the robot (i.e. how much the robot is expected to have moved in 1 tick, whether that be a change in position or angle) and finding the new position of each particle using that movement information. `update_particles_with_motion_model` will be tested by creating a list of movements, applying them to the list of particles, and verifying that their new positions is correct.
* Importance weights: **???**
* Normalizing the particles' importance weights can be done by summing their weights, getting the inverse of this sum, and multiplying this with each particle's importance weight. Resampling the particles will be done using Python's `random.choice`, which is able to select elements in a list using probability weights. Testing of `normalize_particles` can be done by verifying the result of the calculation, and testing of `resample_particles` can be done by running multiple trials, and determining that a weighted choice is able to select a particle with the probability that its weight suggests (e.g. if a particle has an importance weigt of 0.1, 100 trils should show that it is selected approximately 10 times).
* The estimated pose of the robot will be updated by averaging the position of all of the particles. `update_estimated_robot_pose` can be tested by simply verifying that the robot's udpated position is correct using different lists of updated particles, before and after a state change.
* Noise: **???**
#### Timeline:
