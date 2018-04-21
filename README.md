# General Node Tree Description

Python module developed for storing node tree materials into xml-based text file. 

## Example

Suppose we would like to create description for the following node tree material.

![Nodetree example](tree_example.png?raw=true)

This is a simple material for Cycles in Blender. So, to store it we use the following code:

```python
from gntd import NodeTreeDescriptor, Color4, Color3, Normal

doc = NodeTreeDescriptor("Example library")
node_tree = doc.add_material()
# add BSDF node
bsdf_node = node_tree.add_node("BSDF", "bsdf_node")  # set the node type and it name
# add input ports for this node
in_color = bsdf_node.add_input("Color", "color4", Color4(0.8))  # set input name, type and default value
in_raugness = bsdf_node.add_input("Roughness", "float", 0.1)
in_normal = bsdf_node.add_input("Normal", "vector3", Normal(0.0))
# also output port
out_bsdf = bsdf_node.add_output("BSDF", "color4")  # set the name of the port and it type

# next add RGB node
rgb_node = node_tree.add_node("RGB", "rgb_node")
# this node has only color parameter, add it
rgb_node.add_parameter("Color", "color3", Color3(1.0))
# output port
out_rgb = rgb_node.add_output("Color", "color3")

# next Value node
value_node = node_tree.add_node("VALUE", "value_node")
value_node.add_parameter("Value", "float", 0.5)
out_value = value_node.add_output("Value", "float")

# the last node is MaterialOutput node
out_node = node_tree.add_node("OUTPUT", "output_node")
# this node has three input ports
out_node.add_input("Surface", "color4", Color4(0.0))
out_node.add_input("Volume", "color4", Color4(0.0))
out_node.add_input("Displacement", "float", 0.0)

# finally create connections
node_tree.add_connection(bsdf_node.get_name(), "BSDF", out_node.get_name(), "Surface")
node_tree.add_connection(rgb_node.get_name(), "Color", bsdf_node.get_name(), "Color")
node_tree.add_connection(value_node.get_name(), "Value", bsdf_node.get_name(), "Roughness")

# save doc to the file
doc.write_xml("example.gem")
```

As a result we will obtain the following example.xml

```
<library name="Example library">
    <node_tree name="Material0">
        <nodes>
            <BSDF name="bsdf_node">
                <input default="0.8 0.8 0.8 1.0" port="Color" type="color4" />
                <input default="0.1" port="Roughness" type="float" />
                <input default="0.0 0.0 0.0" port="Normal" type="vector3" />
                <output port="BSDF" type="color4" />
            </BSDF>
            <RGB name="rgb_node">
                <output port="Color" type="color3" />
                <parameter name="Color" type="color3" value="1.0 1.0 1.0" />
            </RGB>
            <VALUE name="value_node">
                <output port="Value" type="float" />
                <parameter name="Value" type="float" value="0.5" />
            </VALUE>
            <OUTPUT name="output_node">
                <input default="0.0 0.0 0.0 1.0" port="Surface" type="color4" />
                <input default="0.0 0.0 0.0 1.0" port="Volume" type="color4" />
                <input default="0.0" port="Displacement" type="float" />
            </OUTPUT>
        </nodes>
        <connections>
            <connection external="False" in_node="output_node" in_port="Surface" out_node="bsdf_node" out_port="BSDF" />
            <connection external="False" in_node="bsdf_node" in_port="Color" out_node="rgb_node" out_port="Color" />
            <connection external="False" in_node="bsdf_node" in_port="Roughness" out_node="value_node" out_port="Value" />
        </connections>
    </node_tree>
</library>
```