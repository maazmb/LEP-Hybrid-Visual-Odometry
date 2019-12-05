# LEP-Hybrid-Visual-Odometry-
We propose a novel real time monocular Hybrid visual odometry formulation which combines the high precision of indirect approaches with the fast performance of direct methods. The system initializes inverse depth estimates represented as a Gaussian probability distribution for features (lines, edges and points) extracted in each keyframe which we continuously propagate and update with new measurements in the following frames. The key idea is to incorporate the depth filter distributions into the initial pose tracking via sparse image alignment and also the pose refinement via map localization. We also propose a comprehensive initialization method of these depth filters and classify the map points into different categories based on the uncertainty of these depth estimates which as a result greatly improves the tracking performance. The experimental evaluation on benchmark datasets shows that the proposed approach is significantly faster than the state-of-the-art algorithms while achieving comparable accuracy. We make our implementation publically open source at github to provide as a valuable reference for the SLAM community.  
