## Project: Search and Sample Return

###Perception step
* Created two new functions for color thresholding: `color_thresh_rocks()` to identify rocks and `color_thresh_obstacles()` to identify obstacles
* In both functions used OpenCV for color selecting. To find the rocks used color range from `[15,95,128]` to `[40,255,255]` in HSV, for obstacles used range from `[0,0,0]` to 
`[179,255,178]`

        def color_thresh_rocks(img):
            imgBGR = cv2.cvtColor(img, cv2.COLOR_RGB2BGR)
            imgHSV = cv2.cvtColor(imgBGR, cv2.COLOR_BGR2HSV)

            lower_yellow = np.array([15,95,128])
            upper_yellow = np.array([40,255,255])
    
            mask = cv2.inRange(imgHSV, lower_yellow, upper_yellow)
 
            return mask
* Filled in function `perception_step()` by following steps described in TODO list: applied perspective transform first then color threshold(terrain, obstacles, rocks), then converted image to rover-centric coordinates then to world coordinates, updated worldmap,distance and angels

###Decision step
* Implemented wall crawling(left wall) by setting Rover.steer to mean of all navigable angles + 15 degrees in the 'forward' mode
  `Rover.steer = np.mean(Rover.nav_angles * 180/np.pi) + 15`
* In the 'stop' mode if we don't see any navigable terrain then turns 180 to the right to change it's direction. If we see navigable terrain then turn 10 degrees to the right from the mean of all navigable angles `np.clip(np.mean(Rover.nav_angles * 180/np.pi), -15, 15) - 10` 
* Defined new mode 'stuck'. Rover goes to 'stuck' mode if:
  1. It is in 'forward' mode and its velocity is below 0.1
  2. If rover turns to same direction more than 250 frames, hence value of Rover.steer stays the same for more than 250 frames then it's running in circle.
  

- To detect the first scenario of stuck mode, I checked Rover.vel inside the `If Rover.mode == 'forward'` condition
    
              if Rover.vel < Rover.max_vel:
                    Rover.throttle = Rover.throttle_set
                    if Rover.vel < 0.1 and Rover.start_count > 20 :
                        Rover.mode = 'stuck'
 `Rover.start_cout` variable was added to ensure that rover doesn't go to 'stuck' mode on the very beginning starting point and also doesn't get stuck at 'stuck' mode. Initial value of 'start_count' is 0, and it increases by one every time `decision_step()` is called. And set to 10 when Rover is at 'stuck' mode
* To detect the second scenario of stuck mode I checked if previous steering angle is equal to current steering angle, and both angles must be bigger than zero(since i used left wall crawling Rover mainly runs in circles counter clockwise turning to the left), and if this conditions are met i updated `Rover.steer_frame_count`
               
         if Rover.steer != 0:
                if Rover.steer > 0 and Rover.prev_steer > 0 and Rover.steer == Rover.prev_steer :
                    Rover.steer_frame_count +=1
                elif Rover.steer_frame_count < 250:
                    Rover.steer_frame_count = 0                    
                Rover.prev_steer = Rover.steer
And if 	`Rover.steer_frame_count` doesn't reach 250 it is set back to 0
                                     
          if Rover.steer_frame_count > 250:
                    Rover.mode = 'stuck'
                    if Rover.steer_frame_count > 255:
                        Rover.steer_frame_count = 150
                elif Rover.mode != 'stuck':    
                    Rover.steer = np.mean(Rover.nav_angles * 180/np.pi) + 15
`if Rover.steer_frame_count > 255:`  is done to get Rover out of 'stuck' mode. 

* Inside the 'stuck' mode throttle and brake are set to 0, and steer is set to -30 to turn Rover little to the right. Then if Rover went to 'stuck' mode by running in circles more than 250 frames then `Rover.steer_frame_count` is increased by one. Finally `Rover.mode` is changed to 'forward'
        
        elif Rover.mode == 'stuck':
            print('stuck ------------stuck-----------------stuck')
            Rover.throttle = 0
            Rover.brake = 0
            Rover.steer = -30
            if Rover.steer_frame_count > 250:
                Rover.steer_frame_count +=1
            Rover.start_count = 10
            Rover.mode = 'forward'


---
Test run video (accelerated x3) is included in the zip, run in windows_x64, screen resolution:1280*720, graphic quality: Good, fps ranges between 16-21   