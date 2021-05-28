JSON Materials
==============

Ogre allows the user to define materials in a json format.
This format is human readable and easily understood, and can be used without needing to recompile the engine.

Ogre contains two hlms implementations out of the box, PBS and Unlit.

PBS
---

Every option for a PBS json material is shown below:

.. code-block:: json

    {
        "pbs" :
        {
            "material_name" :
            {
                "macroblock" : "unique_name" ["unique_name", "unique_name_for_shadows"],
                "blendblock" : "unique_name" ["unique_name", "unique_name_for_shadows"],
                "alpha_test" : ["less" "less_equal" "equal" "not_equal" "greater_equal" "greater" "never" "always" "disabled", 0.5],
                "shadow_const_bias" : 0.01,

                "workflow" : "specular_ogre" "specular_fresnel" "metallic",

                "transparency" :
                {
                    "value" : 1.0,
                    "mode" : "None" "Transparent" "Fade",
                    "use_alpha_from_textures" : true
                },

                "diffuse" :
                {
                    "value" : [1, 1, 1],
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "specular" :
                {
                    "value" : [1, 1, 1],
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "roughness" :
                {
                    "value" : 1,
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "fresnel" :
                {
                    "mode" : "coeff" "ior" "coloured" "coloured_ior",
                    "value" : [1, 1, 1],
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "metallness" :
                {
                    "value" : 1,
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "normal" :
                {
                    "value" : 1,
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "detail_weight" :
                {
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "detail_diffuse0" :
                {
                    "mode" : "NormalNonPremul" "NormalPremul" "Add" "Subtract" "Multiply" "Multiply2x" "Screen" "Overlay" "Lighten" "Darken" "GrainExtract" "GrainMerge" "Difference",
                    "offset" : [0, 0],
                    "scale" : [1, 1],
                    "value" : 1,
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "detail_normal0" :
                {
                    "offset" : [0, 0],
                    "scale" : [1, 1],
                    "value" : 1,
                    "texture" : "texture.png",
                    "sampler" : "unique_name",
                    "uv" : 0
                },

                "reflection" :
                {
                    "texture" : "cubemap.png",
                    "sampler" : "unique_name"
                }
            }
        }
    }


Unlit
-----

Every option for an Unlit json material is shown below:

.. code-block:: json

    {
        "unlit" :
        {
            "material_name" :
            {
                "macroblock" : "unique_name" ["unique_name", "unique_name_for_shadows"],
                "blendblock" : "unique_name" ["unique_name", "unique_name_for_shadows"],
                "alpha_test" : ["disabled", 0.5],
                "shadow_const_bias" : 0.01,

                "diffuse": [1, 1, 1, 1],
                "diffuse_map0":
                {
                    "texture": "texture.png",
                    "sampler" : "unique_name",
                    "blendmode": "NormalNonPremul" "NormalPremul" "Add" "Subtract" "Multiply" "Multiply2x" "Screen" "Overlay" "Lighten" "Darken" "GrainExtract" "GrainMerge" "Difference",
                    "uv": 0,
                    "animate": [
                        [1, 0, 0, 0],
                        [0, 1, 0, 0],
                        [0, 0, 1, 0],
                        [0, 0, 0, 1]
                    ]
                },
                "diffuse_map1": {},
                "diffuse_map2": {},
                "diffuse_map3": {},
                "diffuse_map4": {},
                "diffuse_map5": {},
                "diffuse_map6": {},
                "diffuse_map7": {},
                "diffuse_map8": {},
                "diffuse_map9": {},
                "diffuse_map10": {},
                "diffuse_map11": {},
                "diffuse_map12": {},
                "diffuse_map13": {},
                "diffuse_map14": {},
                "diffuse_map15": {}
            }
        }
    }

Settings for subsequent diffuse maps are omitted here for clarity's sake.
The same settings apply to every diffuse map.

Blendblocks, Samplerblocks, Macroblocks
---------------------------------------
Blendblocks, samplerblocks and macroblocks are used in combination with regular materials to provide extra functionality.
Their settings are separated out from the actual material definition to avoid redundancy and to better improve performance when performing similar tasks.
Once defined, many datablocks can reference these blocks.

.. code-block:: json

    {
        "samplers" :
        {
            "unique_name" :
            {
                "min" : "point" "linear" "anisotropic",
                "mag" : "point" "linear" "anisotropic",
                "mip" : "none" "point" "linear" "anisotropic",
                "u" : "wrap" "mirror" "clamp" "border",
                "v" : "wrap" "mirror" "clamp" "border",
                "w" : "wrap" "mirror" "clamp" "border",
                "miplodbias" : 0,
                "max_anisotropic" : 1,
                "compare_function" : "less" "less_equal" "equal" "not_equal" "greater_equal" "greater" "never" "always" "disabled",
                "border" : [1, 1, 1, 1],
                "min_lod" : -3.40282347E+38,
                "max_lod" : 3.40282347E+38
            }
        },

        "macroblocks" :
        {
            "unique_name" :
            {
                "scissor_test" : false,
                "depth_check" : true,
                "depth_write" : true,
                "depth_function" : "less" "less_equal" "equal" "not_equal" "greater_equal" "greater" "never" "always",
                "depth_bias_constant" : 0,
                "depth_bias_slope_scale" : 0,
                "cull_mode" : "none" "clockwise" "anticlockwise",
                "polygon_mode" : "points" "wireframe" "solid"
            }
        },

        "blendblocks" :
        {
            "unique_name" :
            {
                "alpha_to_coverage" : false,
                "blendmask" : "rgba",
                "separate_blend" : true,
                "src_blend_factor" : "one" "zero" "dst_colour" "src_colour" "one_minus_dst_colour" "one_minus_src_colour" "dst_alpha" "src_alpha" "one_minus_dst_alpha" "one_minus_src_alpha",
                "dst_blend_factor" : "one" "zero" "dst_colour" "src_colour" "one_minus_dst_colour" "one_minus_src_colour" "dst_alpha" "src_alpha" "one_minus_dst_alpha" "one_minus_src_alpha",
                "src_alpha_blend_factor" : "one" "zero" "dst_colour" "src_colour" "one_minus_dst_colour" "one_minus_src_colour" "dst_alpha" "src_alpha" "one_minus_dst_alpha" "one_minus_src_alpha",
                "dst_alpha_blend_factor" : "one" "zero" "dst_colour" "src_colour" "one_minus_dst_colour" "one_minus_src_colour" "dst_alpha" "src_alpha" "one_minus_dst_alpha" "one_minus_src_alpha",
                "blend_operation" : "add" "subtract" "reverse_subtract" "min" "max",
                "blend_operation_alpha" : "add" "subtract" "reverse_subtract" "min" "max"
            }
        }
    }
