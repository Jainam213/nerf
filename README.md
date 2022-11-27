# Latent-NeRF for Shape-Guided Generation of 3D Shapes and Textures
<a href="https://arxiv.org/abs/2211.07600"><img src="https://img.shields.io/badge/arXiv-2211.07600-b31b1b.svg" height=22.5></a>
<a href="https://opensource.org/licenses/MIT"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" height=22.5></a>  

> Text-guided image generation has progressed rapidly in recent years, inspiring major breakthroughs in text-guided shape generation. Recently, it has been shown that using score distillation, one can successfully text-guide a NeRF model to generate a 3D object. We adapt the score distillation to the publicly available, and computationally efficient, Latent Diffusion Models, which apply the entire diffusion process in a compact latent space of a pretrained autoencoder. As NeRFs operate in image space, a naïve solution for guiding them with latent score distillation would require encoding to the latent space at each guidance step. Instead, we propose to bring the NeRF to the latent space, resulting in a Latent-NeRF.
Analyzing our Latent-NeRF, we show that while Text-to-3D models can generate impressive results, they are inherently unconstrained and may lack the ability to guide or enforce a specific 3D structure. To assist and direct the 3D generation, we propose to guide our Latent-NeRF using a Sketch-Shape: an abstract geometry that defines the coarse structure of the desired object. Then, we present means to integrate such a constraint directly into a Latent-NeRF. This unique combination of text and shape guidance allows for increased control over the generation process.
We also show that latent score distillation can be successfully applied directly on 3D meshes. This allows for generating high-quality textures on a given geometry. Our experiments validate the power of our different forms of guidance and the efficiency of using latent rendering.

## Description
Official Implementation for "Latent-NeRF for Shape-Guided Generation of 3D Shapes and Textures".

> TL;DR - We explore different ways of introducing shape-guidance for Text-to-3D and present three models: a purely text-guided Latent-NeRF, Latent-NeRF with soft shape guidance for more exact control over the generated shape, and Latent-Paint for texture generation for explicit shapes.



## Recent Updates
* `27.11.2022` - Code release

* `14.11.2022` - Created initial repo

### Latent-Paint

In the `Latent-Paint` application, a texture is generated for an explicit mesh directly on its texture map using stable-diffusion as a prior.

Here the geometry is used as a hard constraint where the generation process is tied to the given mesh and its parameterization.

<img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/car.gif" width="800px"/>

<img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/fish.gif" width="800px"/>

Below we can see the progress of the generation process over the optimization process

<img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/fish_with_texture.gif" width="800px"/>

To create such results, run the `train_latent_paint` script. Parameters are handled using [pyrallis](https://eladrich.github.io/pyrallis/) and can be passed from a config file or the cmd.

```bash
 python -m scripts.train_latent_paint --config_path demo_configs/latent_paint/goldfish.yaml
```

or alternatively 

```bash
python -m scripts.train_latent_paint --log.exp_name 2022_11_22_goldfish --guide.text "A goldfish"  --guide.shape_path /nfs/private/gal/meshes/blub.obj
```

### Sketch-Guided Latent-NeRF

Here we use a simple coarse geometry which we call a `SketchShape` to guide the generation process. 

A `SketchShape` presents a soft constraint which guides the occupancy of a learned NeRF model but isn't constrained to its exact geometry.

<img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/sketch_teddy.gif" width="800px"/>

A `SketchShape` can come in many forms, here are some extruded ones.

<img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/german_shep.gif" width="800px"/>

<img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/robot_hand.gif" width="800px"/>

To create such results, run the `train_latent_nerf` script. Parameters are handled using [pyrallis](https://eladrich.github.io/pyrallis/) and can be passed from a config file or the cmd.

```bash
 python -m scripts.train_latent_nerf --config_path demo_configs/latent_nerf/lego_man.yaml
```

Or alternatively
```bash
python -m scripts.train_latent_nerf --log.exp_name: '2022_11_25_lego_man' --guide.text 'a lego man' --guide.shape_path shapes/teddy.obj --render.nerf_type latent
```



### Unconstrained Latent-NeRF

Here we apply a text-to-3D without any shape constraint similarly to dreamfusion and stable-dreamfusion. 

We directly train the NeRF in latent space, so no encoding into the latent space is required during training.


<img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/latent_nerf_compressed.gif" width="800px"/>

<p align="left">
  <img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/castle.gif" width="200px"/>
  <img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/palmtree.gif" width="200px"/>
  <img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/fruits.gif" width="200px"/>
  <img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/pancake.gif" width="200px"/>
</p>

To create such results, run the `train_latent_nerf` script. Parameters are handled using [pyrallis](https://eladrich.github.io/pyrallis/) and can be passed from a config file or the cmd.

```bash
 python -m scripts.train_latent_nerf --config_path demo_configs/latent_nerf/sand_castle.yaml
```

Or alternatively
```bash
python -m scripts.train_latent_nerf --log.exp_name: 'sand_castle' --guide.text 'a highly detailed sand castle' --render.nerf_type latent
```

#### Textual Inversion

As our Latent-NeRF is supervised by Stable-Diffusion, we can also use `Textual Inversion` tokens as part of the input text prompt. This allows conditioning the object generation on specific objects and styles, defined only by input images.

<img src="https://github.com/eladrich/latent-nerf/raw/docs/docs/textual_inversion.gif" width="800px"/>

For Textual-Inversion results use the `guide.concept_name` with a concept from the [hugging-face concept library](https://huggingface.co/sd-concepts-library). For example `--guide.concept_name=cat-toy` and then simply use the corresponding token in your `--guide.text`



## Getting Started


### Installation
Install the common dependencies from the `requirements.txt` file
```bash
pip install -r requirements.txt
```

For `Latent-NeRF` with shape-guidance, additionally install `igl`
```bash
conda install -c conda-forge igl
```

For `Latent-Paint`, additionally install `kaolin`
```bash
 pip install git+https://github.com/NVIDIAGameWorks/kaolin
```

## Acknowledgments
The `Latent-NeRF` code is heavily based on the [stable-dreamfusion](https://github.com/ashawkey/stable-dreamfusion) project, and the `Latent-Paint` code borrows from [text2mesh](https://github.com/threedle/text2mesh).





