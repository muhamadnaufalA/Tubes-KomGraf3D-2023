import bpy, random

#bpy.ops.wm.console_toggle()

#########################README#################################
#The object is split into three elements: 
#                          -tree_trunk
#                          -main_foliage (a single large cube)
#                          -secondary_foliage (smaller cubes placed
#                                              on the corners of the
#                                              main foliage cube
#                                              and on the center of each
#                                              face)
#Param list: 
#COLOR_ELEM - defines the color of each element

#TRUNK_WIDTH - space between the points at the base of the trunk
#TRUNK_HEIGHT - how high the trunk stretches on the z axis
#TRUNK_CEILING_TAPER - the ceiling of the trunk is a smaller triangle
#                      the ceiling_taper is by how much each
#                      point is closer to each other

#INITIAL_OBJECT_LOCATION - the location where the whole tree wil end up rendered

#MAIN_FOLIAGE_SIZE,             |
#SECONDARY_FOLIAGE_CENTER_SIZE, | size of the cubes
#SECONDARY_FOLIAGE_CORNER_SIZE, |          

#PERCENT_DIST_SECONDARY_FOLIAGE_MAIN_FOLIAGE_CENTER -
# - a percentage of how far the secondary_foliage cubes
# should be moved (closer or farther away) 
# 0.0 means moving them to the same coords as the center of main_foliage
# 1.0 means not moving them
# >1.0 means moving them in the opposite direction from the center 
# of main_foliage

#APPLY_PERCENT_DIST_ONLY_TO_SECONDARY_CORNERS -
# boolean flag to decide if the secondary_foliage corner cubes
# should be the only ones who have the percent_dist move opearation
# applied to them or if the secondary_foliage center cubes should
# have it applied as well

#USE_RANDOM_SECONDARY_FOLIAGE_SAMPLING -
# boolean flag to decide if only a random sampling of the secondary
# foliage cubes should be chosen to be rendered 
# (how many of each secondary_foliage cube is given by the following
# 2 params)

#NUMBER_OF_SECONDARY_FOLIAGE_CORNER |
#NUMBER_OF_SECONDARY_FOLIAGE_CENTER | -
# describes how many of each of the secondary_foliage cubes
# should be chosen when USE_RANDOM_SECONDARY_FOLIAGE_SAMPLING
# is enabled

#MAIN_FOLIAGE_ROTATION
# rotation of the main_foliage cube

#SECONDARY_CORNER_ROTATION |
#SECONDARY_CENTER_ROTATION | -
# rotation of the secondary_foliage cubes.
# note that these two values get added to the main_foliage_rotation
# for consistency's sake

def create_tree(

        COLOR_SECONDARY_FOLIAGE_CORNER = (0.3, 1, 0.2),
        COLOR_SECONDARY_FOLIAGE_CENTER = (0.1, 0.75, 0.15),
        COLOR_MAIN_FOLIAGE = (0, 1, 0),
        COLOR_TREE_TRUNK = (0.36, 0.20, 0.09),

        TRUNK_WIDTH = 2,
        TRUNK_HEIGHT = 10,
        TRUNK_CEILING_TAPER = 2,

        INITIAL_OBJECT_LOCATION = (1,1,0),

        MAIN_FOLIAGE_SIZE = 5,
        SECONDARY_FOLIAGE_CENTER_SIZE = 3,
        SECONDARY_FOLIAGE_CORNER_SIZE = 3,

        PERCENT_DIST_SECONDARY_FOLIAGE_MAIN_FOLIAGE_CENTER = 0.8, 
        APPLY_PERCENT_DIST_ONLY_TO_SECONDARY_CORNERS = True, 
        
        USE_RANDOM_SECONDARY_FOLIAGE_SAMPLING = True, 
        NUMBER_OF_SECONDARY_FOLIAGE_CORNER = 5, 
        NUMBER_OF_SECONDARY_FOLIAGE_CENTER = 4, 

        MAIN_FOLIAGE_ROTATION = (45, 35, 0),
        SECONDARY_CORNER_ROTATION = (-30, 15, 10),
        SECONDARY_CENTER_ROTATION = (30, 30, 0),

    ):

    #########################UTILS#################################

    def degrees_to_radians(degrees: float):
        return degrees * 0.01745329

    def move_point_on_axis(movable, reference, percentage):
        p1, p2 = movable, reference
        percentage = 1 - percentage
        return (
            p1[0] + (p2[0] - p1[0])*percentage,
            p1[1] + (p2[1] - p1[1])*percentage,
            p1[2] + (p2[2] - p1[2])*percentage
        )
        
    def add_coords(tuple1, tuple2):
        return (
            tuple1[0] + tuple2[0],
            tuple1[1] + tuple2[1],
            tuple1[2] + tuple2[2]
        )
        
    def create_color_material(r, g, b, a=0):
    # https://blender.stackexchange.com/questions/201874/how-to-add-a-color-to-a-generated-cube-within-a-python-script
        mat = bpy.data.materials.new("Custom color")
        # Activate its nodes
        mat.use_nodes = True
        # Get the principled BSDF (created by default)
        principled = mat.node_tree.nodes['Principled BSDF']
        # Assign the color
        principled.inputs['Base Color'].default_value = (r, g, b, a)
        return mat

    ALL_VERTS = [] #list of all vertices to be used for mesh creation
    ALL_FACES = [] #list of all faces to be used for mesh creation

    #########################PARAMS#################################

    COLOR_SECONDARY_FOLIAGE_CORNER = create_color_material(
                                        COLOR_SECONDARY_FOLIAGE_CORNER[0],
                                        COLOR_SECONDARY_FOLIAGE_CORNER[1],
                                        COLOR_SECONDARY_FOLIAGE_CORNER[2]
                                        )
    COLOR_SECONDARY_FOLIAGE_CENTER = create_color_material(
                                        COLOR_SECONDARY_FOLIAGE_CENTER[0],
                                        COLOR_SECONDARY_FOLIAGE_CENTER[1],
                                        COLOR_SECONDARY_FOLIAGE_CENTER[2]
                                        )
    COLOR_MAIN_FOLIAGE = create_color_material(
                            COLOR_MAIN_FOLIAGE[0],
                            COLOR_MAIN_FOLIAGE[1],
                            COLOR_MAIN_FOLIAGE[2]
                            )
    COLOR_TREE_TRUNK = create_color_material(
                            COLOR_TREE_TRUNK[0],
                            COLOR_TREE_TRUNK[1],
                            COLOR_TREE_TRUNK[2]
                            )

    MAIN_FOLIAGE_ROTATION = (degrees_to_radians(MAIN_FOLIAGE_ROTATION[0]),
                             degrees_to_radians(MAIN_FOLIAGE_ROTATION[1]),
                             degrees_to_radians(MAIN_FOLIAGE_ROTATION[2]))
    SECONDARY_CORNER_ROTATION = (degrees_to_radians(SECONDARY_CORNER_ROTATION[0]),
                                 degrees_to_radians(SECONDARY_CORNER_ROTATION[1]),
                                 degrees_to_radians(SECONDARY_CORNER_ROTATION[2]))
    SECONDARY_CENTER_ROTATION = (degrees_to_radians(SECONDARY_CENTER_ROTATION[0]),
                                 degrees_to_radians(SECONDARY_CENTER_ROTATION[1]),
                                 degrees_to_radians(SECONDARY_CENTER_ROTATION[2]))                         
            

    #########################VERTS#################################

    verts_trunk = [
    (0,0,0), #floor
    (TRUNK_WIDTH,0,0), #floor
    (0,TRUNK_WIDTH,0), #floor
    (0,0,TRUNK_HEIGHT), #ceiling 
    (TRUNK_WIDTH/TRUNK_CEILING_TAPER, 0, TRUNK_HEIGHT), #ceiling
    (0, TRUNK_WIDTH/TRUNK_CEILING_TAPER, TRUNK_HEIGHT) #ceiling
    ] 

    ALL_VERTS.extend(verts_trunk)



    #########################FACES#################################

    faces_trunk = [
    (0,1,2), #floor
    (3,4,5), #ceiling
    (0,1,4,3), #first trunk face
    (1,2,5,4), #second trunk face
    (2,0,3,5) #third trunk face
    ]

    ALL_FACES.extend(faces_trunk)

    #########################PRIMITIVES CREATION#################################

    #main foliage is a cube rotated by 45 degrees on x and y
    #with its diagonal going straight into 
    #the centroid of the trunk ceiling triangle

    trunk_ceiling_centroid = (
    (verts_trunk[3][0]+verts_trunk[4][0]+verts_trunk[5][0])/3
    + INITIAL_OBJECT_LOCATION[0],
    (verts_trunk[3][1]+verts_trunk[4][1]+verts_trunk[5][1])/3
    +INITIAL_OBJECT_LOCATION[1],
    TRUNK_HEIGHT + INITIAL_OBJECT_LOCATION[2])

    main_foliage_location = (
    trunk_ceiling_centroid[0] ,
    trunk_ceiling_centroid[1] ,
    trunk_ceiling_centroid[2] + MAIN_FOLIAGE_SIZE /1.5
    )

    bpy.ops.mesh.primitive_cube_add(size=MAIN_FOLIAGE_SIZE, 
                                    location=main_foliage_location,
                                    rotation=MAIN_FOLIAGE_ROTATION)
    obj = bpy.context.object
    obj.name = "main_foliage"
    obj.data.materials.append(COLOR_MAIN_FOLIAGE)

    #have to apply the objects transform matrix to get the correct coords
    verts_foliage_main = [(obj.matrix_world @ v.co) for v in obj.data.vertices]
    face_centers_foliage_main = [(obj.matrix_world @ f.center) for f in obj.data.polygons]

    ###############DISPLAYING THE CORNER SECONDARY FOLIAGE###################
    verts_secondary_foliage = verts_foliage_main

    #remove vertex with lowest z coord as it is inside the tree trunk
    bottom_vertex = min(verts_foliage_main, key=lambda vertex: vertex[2])
    verts_secondary_foliage.remove(bottom_vertex)

    if USE_RANDOM_SECONDARY_FOLIAGE_SAMPLING:
        verts_secondary_foliage = random.sample(verts_secondary_foliage,
                                    k=NUMBER_OF_SECONDARY_FOLIAGE_CORNER)

    #bring these coordinates closer to the center of the main cube
    verts_secondary_foliage = [
    move_point_on_axis(v, 
                       main_foliage_location, 
                       PERCENT_DIST_SECONDARY_FOLIAGE_MAIN_FOLIAGE_CENTER) 
    for v in verts_secondary_foliage
    ]

    #add them to the scene and add color
    for vertex in verts_secondary_foliage:
        bpy.ops.mesh.primitive_cube_add(size=SECONDARY_FOLIAGE_CORNER_SIZE, 
                                        location=vertex, 
                                        rotation=add_coords(MAIN_FOLIAGE_ROTATION, 
                                                            SECONDARY_CORNER_ROTATION))
        obj = bpy.context.object
        obj.name = "secondary_foliage_corner"
        obj.data.materials.append(COLOR_SECONDARY_FOLIAGE_CORNER)

    ###############DISPLAYING THE CENTER SECONDARY FOLIAGE###################
    verts_secondary_foliage = []
    for f in face_centers_foliage_main:
        verts_secondary_foliage.append(f)

    if USE_RANDOM_SECONDARY_FOLIAGE_SAMPLING:
        verts_secondary_foliage = random.sample(verts_secondary_foliage, 
                                    k=NUMBER_OF_SECONDARY_FOLIAGE_CENTER)

    if not APPLY_PERCENT_DIST_ONLY_TO_SECONDARY_CORNERS:
        verts_secondary_foliage = [
    move_point_on_axis(v, 
                       main_foliage_location, 
                       PERCENT_DIST_SECONDARY_FOLIAGE_MAIN_FOLIAGE_CENTER) 
    for v in verts_secondary_foliage
    ]

    #add them to the scene and add color
    for vertex in verts_secondary_foliage:
        bpy.ops.mesh.primitive_cube_add(size=SECONDARY_FOLIAGE_CENTER_SIZE, 
                                        location=vertex, 
                                        rotation=add_coords(MAIN_FOLIAGE_ROTATION, 
                                                            SECONDARY_CENTER_ROTATION))
        obj = bpy.context.object
        obj.name = "secondary_foliage_center"
        obj.data.materials.append(COLOR_SECONDARY_FOLIAGE_CENTER)
            
                                        
    #########################MESH CREATION#################################
    MESH = bpy.data.meshes.new("tree_trunk")

    TREE_OBJ = bpy.data.objects.new("tree_trunk", MESH)
    TREE_OBJ.location = INITIAL_OBJECT_LOCATION
    bpy.context.collection.objects.link(TREE_OBJ)
    TREE_OBJ.data.materials.append(COLOR_TREE_TRUNK)

    MESH.from_pydata(ALL_VERTS, [], ALL_FACES)
    MESH.update(calc_edges=True)




#delete all objects
bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

#render some trees
create_tree(INITIAL_OBJECT_LOCATION=(1,1,0))
create_tree(INITIAL_OBJECT_LOCATION=(10,7,0))
create_tree(INITIAL_OBJECT_LOCATION=(2,18,0))