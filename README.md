# P1: Algoritmo de covertura BSA

The practic is about creating the sofware of a localized vacuum cleaner to make it clean the higher amount of room posible. The vacuum knew it's possition in the world and it had a map of the room.
The project could be divided in three parts

## World-map correlation
Despite the vacuum cleaner knows where it is, we needed to figure out how to to change for the woorld coordenates to the map ones.
To do so I used the affine transformation to change from world coordenades to the map ones or vice versa.
In order to do so I get the world and map coordenades of three points, that in my case correspond to the upper right, upper left and bottom left corners, and create a matrix with one of the coordenaded, map or world depending on which one I'm trying to convert from, 
and x and y vector with the coordenades in the other reference system and use the necesary echuations to cange from one to another.

## Path plannig
For the robot to be able to clean all the house I mus have a route to follow. To do so I follow some steps
1. The map is in RBG, but the visualizator of the function `WebGUI.showNumpy()` only proceses `Int` matrix, so I adapt the map to the format, changing the black, [0, 0, 0], for 0 and the white, [255, 255, 255], for 127
2. The robot have a size of 35 pixels, so to avoid having that into consideration while plannig, I ingrease the size of the walls. In this case I would mark them as `FAKE_WALL`, to make the resto of the algorithm easier.
3. Ones the map is suitable to navigate, I create a smaller grid map with the positions the robot could go. To do so I create a smaller matrix in which each cell represent an area of the size of the robot, or a bit smaller.
   To do so the program follows this rules:
   - If there is a wall pixel in the region, that cell would be marked as wall.
   - If the cell contains `FAKE_WALL` pixels but not actual walls, the pixel would be marked as `DIRTY` only if the pixel in the middle of the region, or the 4 pixels in case that the side of the square is an even number, is not in a pixel marked as `FAKE_WALL`
  The reason for this is to be able to consider that the robot is smaller than what it accually is, so the area that the borders of the robot would touch are in more than one cell. This is usefull because if I consider the square exacly the robot size, due to inacuracy in the navigation process, i would not clean everything

4. With the grid map created, I create the route the robot would follow. To do so I use the actual position of the robot as the starting point and them I would go throw every `DIRTY` cell until there are non. The way is done is by checking the four directions, North, East, South and West in
that order and if it finds a `DIRTY` cell, it would mark it as `CLEAN` and asume that is in that one. If all the surrounding positions are `CLEAN`, I would use the BSA algorithm to find the closes `DIRTY` cell and the route to it

5. To make the execution of the plan smoothier, I would remove from the plan the cells that are inner poits of a line. For example, if the route is [(0, 0), (0, 1), (0, 2), (0, 3), (1, 3), (2, 3)], I would change it to [(0,0), (0,3), (2,3)]

### Other solutions I've try
- Initially what I did was to create the cells the size of the robot and not to expand the walls. It worked fine but due to inaccuracy in the control it would leave space without cleaning between the lines, so it wasn't ideal.
- I also try to remove the black upper and left borders of the map in order to make the cells fit better to the robot size, but it didn't change anything
 
## Plan execution

After it has finished the plan, it have to execute it. To do so firstly it look at the first goal of the route. It conteins the coordeantes in the smaller map, so it would transform them from those to the actoual map ones and, from them, to the world coordenates it needs.
With that it calculates the distance and the angle between it's current position and the goal. If the robot is not facing the goal, it would turn arround until it face it and then it would move to the goal. Depending of the distance it would go in a higher or lower speed, both 
angular and linear. This is the reason why the route only contains the extreams of the lines, because initially it was always moving slowly trying to nail the inner possition despite the fact it would continue straight after that, so by removing those points i can effectively move
in a proper speed

## Video
https://urjc-my.sharepoint.com/:v:/g/personal/d_lopezm_2022_alumnos_urjc_es/Eao6hYEUC7JGoCqDS3di5tQBdBGUsBMy54-QkckbEuhGIQ?e=QIcw61&nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0%3D
      
#
#
# P2: Rescue Drone
This practice goal was to make a drone able to detect people in the sea. In orther to archive that, we are proporcionate with the altitude and latitude of both the drone landing port and the stimate localization of the survivors.

The code has been implemented as a FSM (finite state machine) whith three states, `START`, `FIND_PEOPLE` and `RETURN`
## Start mode
In the start mode, the drone will firstly take off from the initial platform to a middle hight, then fly to the stimate position of the survivors.

The hight at which it will start going to the survivors position is smaller than the one that it will use to fly, because in the route to there it has plenty of time to reach the desire altitude

Ones the drone has reached the survivors stimate position, it will go to the Search mode

## Search mode
In this mode, the drone goal is to find every single person in the area.

As the drone can't know the size of the area it must search in, it will start at the stimate possition and move tracing a square spiral, so it will look for the people in a surface whose size is constantly increasing. This spiral is divided in small squares, in which the drone will look for the people.

### Preople detection
To be able to detect the people, the drone uses a facial detection function, but it had some problems.
- Oringinaly, the facial detection gave a lot of false positives in the sea surfice, so to solve it I use a HSV blue filter so it avoids looking where ther is only water.
- If the face was not aling with the camera or up-side-down, it will not detect it, so I turn the image in steps of 45º to see it from every posible direction.
- Ones the other solutions were implemented, the algorithm maked the execution realy slow, so it set specific points to use it, the middle of the spiral squares.
Ones it has found a face, it must check if its a new face or not, to avoid counting each person multiple times, so I transform from the coordenades in the image to the ones of the world, so it can compare them with the ones of the survivors previously founded. If it was a new face, the localization will be store.

## Return mode
The drone dosn't know how much people there is, so it will keep searching until it runs out of battery.

To simulate the battery I use a timer. Ones a specific time theshold is surpased, it will detect that the drone battery is low and it will change to return mode
In this mode it will fly to the landing port and, ones it has stop flying, it will send the localization of the survivors. 

## Other ideas

Initialy, the idea was to have the drone flying low so it will only see each person ones. This was to be able to send the current drone localization as the survivor one, what its much easier than transforming the camera coordenades to the map ones. The problem was that some of the survivors where so close to the edges of the squares in which the drone looks that if it's too close it won't see them but if it's further away they will be seen twice. At the end the change was very positive as it let me fly in a higher altitude, so it will spend less time to find everyone.

To send this information, it firstly needs to transform from its map coordenades to UTM so they can be used by the rescatists.

## Video
https://urjc-my.sharepoint.com/:v:/g/personal/d_lopezm_2022_alumnos_urjc_es/EZYS01OeoN1FqbE2qOj_DPYBulyD2DIiBCMlAl5fn8z-JQ?e=fKlhxY&nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0%3D


#
#
# P3: Autoparking

In this exercise the goal was to make a car park by itserve. To do that I used a FSM. The problem consisted in a set of objetives: Aligning with the road, finding the paking space and parking.

## Aligning with the road
In orther to be able to park the car must be facing the same direction as the road. To do so I fistly must know what the direction fo the road is. That was obtained by calculating the main line on the road and changing the angle between the car and the line to world coordenades. With that the car can know in every moment if is facing the right direction.

When the direction is kown, I check if the distance between the car an the paked ones is too big or small, and move the car if it must be changed.

## Finding a parking space
Ones the car is able to move in the road direction, it must find were to park. If the front or back laser does not detect anything, the place has been found, but if not is a bit more complicated. The car moves forward until the middle laser detects nothing. Then it checks the first non infinite distances at both sides if the middle and calculates the distances between them. The laser gives the distance between itselve and the point, but if it is multiply bu the sin of the angle we obtain the distance to the middle, and with that the distance between the front of one car and the back of the other.

## Parking
To park the car must be well position. To check that, because the middle right laser distance is infinite, the car moves forward until is not. Then, it keeps moving until one of the right lasers facing back, in this case the one with 65º to the middle, does also not detect. I fistly check the middle one because if not it may have some misdetections.

Then I turn the weels all to the right and go back until the diference between the car yaw and the road one is bigger than 40º.

Next it turns the weels to the left and keeps going back until is aligned with the road again.

Lastly it checks the distances to both the front and the back car and moves to let some space between it and those cars

## Video
https://urjc-my.sharepoint.com/:v:/g/personal/d_lopezm_2022_alumnos_urjc_es/EaaYYXFVgt9LvWqjP0SoBM8BGgye5NkRmYIGEUhy6bjd3Q?e=zPU22G&nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0%3D

#
#
# P4: Logistic

In this exercise the goal was to make a movile robot capable of moving shelfs inside a warehouse. I was divided in three dificulty levels.

## Level 1

In the first level we had an holonomic robot. Firstly, I increase the walls of the map to be able to consider the robot a point as done in the first exercise and then I made the functions that changed from the world coordenades to the map ones and vice versa.

Then I started to use the ompl library to calculate the route. As the robot could be consider as a point, to check if it was a valid position I only checked if the middle point was inside a wall. TO plan I've used the BITstar as it game me better results and the simple setup to create the problem instace, as it led me simplify the routes.

After it was finishd, to make it able to move I just check if the angle difference between the desire one for the route and the yaw was bigger than a little threshold. If so it turned until if was facing that direction, otherwise i will go straigt to the goal.

In this version, as I wasn't taking into consideration the shelf geometry, I don't make the robot pick it up.

## Level 2

In the second level I had to check the geometry. With the level 1 code as a base, I removed the walls increase and I created a proper function to check if the robot was in a possible position. It was done by creating a area the side of the robot, or the shelf one, turning it to the yaw angle and then chaching if inside it there were walls.

Also, as now I was moving the shelves, I had to modify the map. When the robot picks up the shelf, it draw a white rectangle in the shelf position. It was usefull to avoid having the shelf coliding with itselve, and making the planner unable to find a route.
After, when the robot was in the position it wanted to leave the shelf, I added four little squares in the shelf legs positions, do it will avoid them when goint for the next one.

## Level 3

In the final level, the robot changed to an ackermann one. To make it work firstly I had to change the function to check the position, as this new robot has a diferent geometry.

After that I change the state to `ReedsSheppStateSpace`, as it led me add the turning radius to the planner.

To finish I had to change the was the robot moved. To do so I check the angle as in the fist one but, insted of just turning, I check if is bigger or smaller than 90º, to move in one direction or another and avoid doing big circular trayectories. I have also inccreased the angular speed it has, as it isn't as effective as the holonomic one at turning arround.

## Video
https://urjc-my.sharepoint.com/:v:/g/personal/d_lopezm_2022_alumnos_urjc_es/EZMzczZ-oKlPsrKFmOJ3vZcBhvUjW_8--uFHrvnGJpTOMw?e=uxMmj5&nav=eyJyZWZlcnJhbEluZm8iOnsicmVmZXJyYWxBcHAiOiJTdHJlYW1XZWJBcHAiLCJyZWZlcnJhbFZpZXciOiJTaGFyZURpYWxvZy1MaW5rIiwicmVmZXJyYWxBcHBQbGF0Zm9ybSI6IldlYiIsInJlZmVycmFsTW9kZSI6InZpZXcifX0%3D




