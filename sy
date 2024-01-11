bl_info = {
    "name": "Li快导",
    "author": "B站id：快绘",
    "version": (1, 0),
    "blender": (2, 80, 0),
    "location": "View3D > Add > My Plugin",
    "description": "快捷导入pbr",
    "category" : "L" 
}

import bpy
import os

def import_texture(material, texture_path, node_type='ShaderNodeTexImage', location=(0, 0)):
    texture_node = material.node_tree.nodes.new(type=node_type)
    texture_node.location = location
    texture = bpy.data.images.get(texture_path)
    if not texture:
        texture = bpy.data.images.load(texture_path)
    texture_node.image = texture
    return texture_node.outputs['Color']

class ImportTexturesOperator(bpy.types.Operator):
    bl_idname = "spio.import_textures"
    bl_label = "导入贴图"

    def execute(self, context):
        main_folder_path = bpy.path.abspath(context.scene.pbr_folder_path)

        if not os.path.exists(main_folder_path):
            self.report({'ERROR'}, "文件夹路径无效")
            return {'CANCELLED'}

        bpy.ops.object.select_all(action='DESELECT')
        bpy.ops.object.select_by_type(type='MESH')
        bpy.ops.object.delete()

        file_list = [f for f in os.listdir(main_folder_path) if f.lower().endswith(('.png', '.tif','.jpg', '.jpeg'))]

        row_count = 0
        col_count = 0

        for filename in file_list:
            texture_path = os.path.join(main_folder_path, filename)

            # 计算平面的位置
            x = col_count * 1.5
            y = row_count * -1.5

            # 创建平面
            bpy.ops.mesh.primitive_plane_add(size=1.0, location=(x, y, 0))
            plane = bpy.context.active_object
            plane.name = os.path.splitext(filename)[0]

            material = bpy.data.materials.new(name=plane.name)
            plane.data.materials.append(material)

            material.use_nodes = True
            tree = material.node_tree
            nodes = tree.nodes

            for node in nodes:
                nodes.remove(node)

            principled_node = nodes.new(type='ShaderNodeBsdfPrincipled')
            principled_node.location = (0, 300)

            # 连接纹理节点到BSDF的Base Color输入
            texture_output = import_texture(material, texture_path)
            links = tree.links
            links.new(texture_output, principled_node.inputs['Base Color'])

            # 添加凹凸节点并连接高度和法线
            bump_node = nodes.new(type='ShaderNodeBump')
            bump_node.location = (-200, 0)
            links.new(texture_output, bump_node.inputs['Height'])
            links.new(bump_node.outputs['Normal'], principled_node.inputs['Normal'])
            
            # 设置凹凸节点的强度为0.5
            bump_node.inputs['Strength'].default_value = 0.2

            output_node = nodes.new(type='ShaderNodeOutputMaterial')
            output_node.location = (400, 300)
            links.new(principled_node.outputs['BSDF'], output_node.inputs['Surface'])

            material.use_nodes = True
            material.asset_mark()
            material.asset_generate_preview()

            col_count += 1
            if col_count >= 30:  # 每行放置5个平面
                col_count = 0
                row_count += 1

        return {'FINISHED'}

class PBRMaterialPanel(bpy.types.Panel):
    bl_label = "PBR材质导入"
    bl_idname = "PBR_MATERIAL_PANEL"
    bl_space_type = 'VIEW_3D'
    bl_region_type = 'UI'
    bl_category = "PBR"

    def draw(self, context):
        layout = self.layout

        scene = context.scene
        layout.prop(scene, "pbr_folder_path")
        layout.operator("spio.import_textures", text="导入贴图")
        # 添加一个新的按钮
        layout.operator("spio.import_pbr_textures", text="导入PBR贴图")
        
# 添加一个自定义操作
class ImportPBRTexturesOperator(bpy.types.Operator):
    bl_idname = "spio.import_pbr_textures"
    bl_label = "导入PBR贴图"
    
    def execute(self, context):
        main_folder_path = bpy.path.abspath(context.scene.pbr_folder_path)
        
        if not os.path.exists(main_folder_path):
            self.report({'ERROR'}, "文件夹路径无效")
            return {'CANCELLED'}
        
        bpy.ops.object.select_all(action='DESELECT')
        bpy.ops.object.select_by_type(type='MESH')
        bpy.ops.object.delete()

        texture_type_mapping = {
            "base": "BaseColor",
            "col": "BaseColor",
            "color": "BaseColor",
            "diffuse": "BaseColor",
            "albedo": "BaseColor",
            "metallic": "Metallic",
            "metalness": "Metallic",
            "metal": "Metallic",
            "roughness": "Roughness",
            "rough": "Roughness",
            "rgh": "Roughness",
            "gloss": "Gloss",
            "glossy": "Gloss",
            "glossiness": "Gloss",
            "normal": "Normal",
            "nor": "Normal",
            "nrm": "Normal",
            "nrml": "Normal",
            "norm": "Normal",
            "bump": "Bump",
            "bmp": "Bump",
            "displacement": "Displacement",
            "displace": "Displacement",
            "disp": "Displacement",
            "dsp": "Displacement",
            "height": "Displacement",
            "heightmap": "Displacement",
            "alpha": "Alpha",
            "opacity": "Alpha",
            "ao": "AO",
            "ambient occlusion": "AO",
            "transmission": "Transmission",
            "transparency": "Transmission"
        }

        plane_size = 1.0  # 平面大小
        plane_spacing = 1.2  # 平面之间的间隔

        row_count = 0
        col_count = 0

        for subfolder_name in os.listdir(main_folder_path):
            subfolder_path = os.path.join(main_folder_path, subfolder_name)

            if os.path.isdir(subfolder_path):
                # 计算平面的位置
                x = col_count * (plane_size + plane_spacing)
                y = row_count * -(plane_size + plane_spacing)

                # 创建平面
                bpy.ops.mesh.primitive_plane_add(size=plane_size, location=(x, y, 0))
                plane = bpy.context.active_object
                plane.name = subfolder_name

                material = bpy.data.materials.new(name=subfolder_name)
                plane.data.materials.append(material)

                material.use_nodes = True
                tree = material.node_tree
                nodes = tree.nodes

                for node in nodes:
                    nodes.remove(node)

                principled_node = nodes.new(type='ShaderNodeBsdfPrincipled')
                principled_node.location = (0, 300)
                output_node = nodes.new(type='ShaderNodeOutputMaterial')
                output_node.location = (400, 300)
                links = tree.links
                links.new(principled_node.outputs['BSDF'], output_node.inputs['Surface'])

                file_list = [f for f in os.listdir(subfolder_path) if f.lower().endswith(('.png','.tif', '.jpg', '.jpeg'))]

                for filename in file_list:
                    base_name, extension = os.path.splitext(filename)

                    texture_type = "None"
                    for keyword, texture_type_mapped in texture_type_mapping.items():
                        if keyword in base_name.lower():
                            texture_type = texture_type_mapped
                            break

                    if not texture_type == "None":
                        try:
                            texture_path = os.path.join(subfolder_path, filename)

                            if texture_type == "Normal":
                                normal_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (-400, 0))
                                normal_map_node = nodes.new(type='ShaderNodeNormalMap')
                                normal_map_node.location = (-200, 0)
                                links.new(normal_output, normal_map_node.inputs['Color'])
                                links.new(normal_map_node.outputs['Normal'], principled_node.inputs['Normal'])
                            elif texture_type == "Roughness":
                                roughness_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (-400, -300))
                                links.new(roughness_output, principled_node.inputs['Roughness'])
                            elif texture_type == "Gloss":
                                gloss_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (-400, -600))
                                invert_node = nodes.new(type='ShaderNodeInvert')
                                invert_node.location = (-200, -600)
                                links.new(gloss_output, invert_node.inputs['Color'])
                                links.new(invert_node.outputs['Color'], principled_node.inputs['Roughness'])
                            elif texture_type == "Metallic":
                                metallic_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (-400, -900))
                                links.new(metallic_output, principled_node.inputs['Metallic'])
                            elif texture_type == "BaseColor":
                                color_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (-400, 300))
                                links.new(color_output, principled_node.inputs['Base Color'])
                            elif texture_type == "Bump":
                                bump_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (-400, -1200))
                                bump_node = nodes.new(type='ShaderNodeBump')
                                bump_node.location = (-200, -1200)
                                links.new(bump_output, bump_node.inputs['Height'])
                                links.new(bump_node.outputs['Normal'], principled_node.inputs['Normal'])
                            elif texture_type == "Displacement":
                                displacement_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (0, -1500))
                                displacement_node = nodes.new(type='ShaderNodeDisplacement')
                                displacement_node.location = (200, -1500)
                                links.new(displacement_output, displacement_node.inputs['Height'])
                                links.new(displacement_node.outputs['Displacement'], output_node.inputs['Displacement'])
                            elif texture_type == "Alpha":
                                alpha_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (0, 1500))
                                links.new(alpha_output, output_node.inputs['Alpha'])
                            elif texture_type == "AO":
                                ao_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (0, 1200))
                                links.new(ao_output, principled_node.inputs['Clearcoat'])
                            elif texture_type == "Transmission":
                                transmission_output = import_texture(material, texture_path, 'ShaderNodeTexImage', (0, 900))
                                links.new(transmission_output, principled_node.inputs['Transmission'])

                            # 释放不再需要的图像资源
                            texture_name = os.path.splitext(filename)[0]  # 获取图像的名称（不包含扩展名）
                            texture = bpy.data.images.get(texture_name)  # 根据名称获取图像对象
                            if texture:
                                bpy.data.images.remove(texture)  # 从内存中移除图像

                        except Exception as e:
                            self.report({'ERROR'}, "无法导入纹理: " + str(e))

                col_count += 1
                if col_count >= 3:  # 每行放置3个平面
                    col_count = 0
                    row_count += 1

                material.use_nodes = True
                material.asset_mark()
                material.asset_generate_preview()

        return {'FINISHED'}
        
        

def register():
    bpy.utils.register_class(ImportPBRTexturesOperator)
    bpy.utils.register_class(PBRMaterialPanel)
    bpy.utils.register_class(ImportTexturesOperator) 
    bpy.types.Scene.pbr_folder_path = bpy.props.StringProperty(
        name="PBR贴图文件夹路径",
        description="选择包含PBR贴图的文件夹路径",
        subtype='DIR_PATH'
    )
    # 批量处理PBR材质的函数
    def create_materials_delayed(scene, depsgraph):
        if scene.pbr_folder_path:
            main_folder_path = bpy.path.abspath(scene.pbr_folder_path)

            if not os.path.exists(main_folder_path):
                return

            bpy.app.handlers.depsgraph_update_post.remove(create_materials_delayed)

            for subfolder_name in os.listdir(main_folder_path):
                subfolder_path = os.path.join(main_folder_path, subfolder_name)

                if os.path.isdir(subfolder_path):
                    # 调用导入PBR贴图操作来创建材质
                    bpy.ops.spio.import_pbr_textures(pbr_folder_path=subfolder_path)

    bpy.app.handlers.depsgraph_update_post.append(create_materials_delayed)





def unregister():
    bpy.utils.unregister_class(ImportPBRTexturesOperator)
    bpy.utils.unregister_class(PBRMaterialPanel)
    bpy.utils.unregister_class(ImportTexturesOperator)
    del bpy.types.Scene.pbr_folder_path

if __name__ == "__main__":
    register()

