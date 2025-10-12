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
      
