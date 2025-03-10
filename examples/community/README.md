# Community Examples

> **For more information about community pipelines, please have a look at [this issue](https://github.com/huggingface/diffusers/issues/841).**

**Community** examples consist of both inference and training examples that have been added by the community.
Please have a look at the following table to get an overview of all community examples. Click on the **Code Example** to get a copy-and-paste ready code example that you can try out.
If a community doesn't work as expected, please open an issue and ping the author on it.

| Example                                | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Code Example                                                      | Colab                                                                                                                                                                                                              |                                                     Author |
|:---------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------:|
| CLIP Guided Stable Diffusion           | Doing CLIP guidance for text to image generation with Stable Diffusion                                                                                                                                                                                                                                                                                                                                                                                                                                   | [CLIP Guided Stable Diffusion](#clip-guided-stable-diffusion)     | [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/huggingface/notebooks/blob/main/diffusers/CLIP_Guided_Stable_diffusion_with_diffusers.ipynb) |             [Suraj Patil](https://github.com/patil-suraj/) | 
| One Step U-Net (Dummy)                 | Example showcasing of how to use Community Pipelines (see https://github.com/huggingface/diffusers/issues/841)                                                                                                                                                                                                                                                                                                                                                                                           | [One Step U-Net](#one-step-unet)                                  | -                                                                                                                                                                                                                  | [Patrick von Platen](https://github.com/patrickvonplaten/) |
| Stable Diffusion Interpolation         | Interpolate the latent space of Stable Diffusion between different prompts/seeds                                                                                                                                                                                                                                                                                                                                                                                                                         | [Stable Diffusion Interpolation](#stable-diffusion-interpolation) | -                                                                                                                                                                                                                  |                    [Nate Raw](https://github.com/nateraw/) |
| Stable Diffusion Mega                  | **One** Stable Diffusion Pipeline with all functionalities of [Text2Image](https://github.com/huggingface/diffusers/blob/main/src/diffusers/pipelines/stable_diffusion/pipeline_stable_diffusion.py), [Image2Image](https://github.com/huggingface/diffusers/blob/main/src/diffusers/pipelines/stable_diffusion/pipeline_stable_diffusion_img2img.py) and [Inpainting](https://github.com/huggingface/diffusers/blob/main/src/diffusers/pipelines/stable_diffusion/pipeline_stable_diffusion_inpaint.py) | [Stable Diffusion Mega](#stable-diffusion-mega)                   | -                                                                                                                                                                                                                  | [Patrick von Platen](https://github.com/patrickvonplaten/) |
| Long Prompt Weighting Stable Diffusion | **One** Stable Diffusion Pipeline without tokens length limit, and support parsing weighting in prompt.                                                                                                                                                                                                                                                                                                                                                                                                  | [Long Prompt Weighting Stable Diffusion](#long-prompt-weighting-stable-diffusion)                                                                 | -                                                                                                                                                                                                                  |                        [SkyTNT](https://github.com/SkyTNT) |
| Speech to Image                        | Using automatic-speech-recognition to transcribe text and Stable Diffusion to generate images                                                                                                                                                                                                                                                                                                                                                                                                            | [Speech to Image](#speech-to-image)                               | -                                                                                                                                                                                                                  | [Mikail Duzenli](https://github.com/MikailINTech)
| Wild Card Stable Diffusion | Stable Diffusion Pipeline that supports prompts that contain wildcard terms (indicated by surrounding double underscores), with values instantiated randomly from a corresponding txt file or a dictionary of possible values                                                                                                                                                                                                                                                                                                     | [Wildcard Stable Diffusion](#wildcard-stable-diffusion)                                                                 | -                                                                                                                                                                                                                  |                        [Shyam Sudhakaran](https://github.com/shyamsn97) |

To load a custom pipeline you just need to pass the `custom_pipeline` argument to `DiffusionPipeline`, as one of the files in `diffusers/examples/community`. Feel free to send a PR with your own pipelines, we will merge them quickly.
```py
pipe = DiffusionPipeline.from_pretrained("CompVis/stable-diffusion-v1-4", custom_pipeline="filename_in_the_community_folder")
```

## Example usages

### CLIP Guided Stable Diffusion

CLIP guided stable diffusion can help to generate more realistic images 
by guiding stable diffusion at every denoising step with an additional CLIP model.

The following code requires roughly 12GB of GPU RAM.

```python
from diffusers import DiffusionPipeline
from transformers import CLIPFeatureExtractor, CLIPModel
import torch


feature_extractor = CLIPFeatureExtractor.from_pretrained("laion/CLIP-ViT-B-32-laion2B-s34B-b79K")
clip_model = CLIPModel.from_pretrained("laion/CLIP-ViT-B-32-laion2B-s34B-b79K", torch_dtype=torch.float16)


guided_pipeline = DiffusionPipeline.from_pretrained(
    "CompVis/stable-diffusion-v1-4",
    custom_pipeline="clip_guided_stable_diffusion",
    clip_model=clip_model,
    feature_extractor=feature_extractor,
    revision="fp16",
    torch_dtype=torch.float16,
)
guided_pipeline.enable_attention_slicing()
guided_pipeline = guided_pipeline.to("cuda")

prompt = "fantasy book cover, full moon, fantasy forest landscape, golden vector elements, fantasy magic, dark light night, intricate, elegant, sharp focus, illustration, highly detailed, digital painting, concept art, matte, art by WLOP and Artgerm and Albert Bierstadt, masterpiece"

generator = torch.Generator(device="cuda").manual_seed(0)
images = []
for i in range(4):
    image = guided_pipeline(
        prompt,
        num_inference_steps=50,
        guidance_scale=7.5,
        clip_guidance_scale=100,
        num_cutouts=4,
        use_cutouts=False,
        generator=generator,
    ).images[0]
    images.append(image)
    
# save images locally
for i, img in enumerate(images):
    img.save(f"./clip_guided_sd/image_{i}.png")
```

The `images` list contains a list of PIL images that can be saved locally or displayed directly in a google colab.
Generated images tend to be of higher qualtiy than natively using stable diffusion. E.g. the above script generates the following images:

![clip_guidance](https://huggingface.co/datasets/patrickvonplaten/images/resolve/main/clip_guidance/merged_clip_guidance.jpg).

### One Step Unet

The dummy "one-step-unet" can be run as follows:

```python
from diffusers import DiffusionPipeline

pipe = DiffusionPipeline.from_pretrained("google/ddpm-cifar10-32", custom_pipeline="one_step_unet")
pipe()
```

**Note**: This community pipeline is not useful as a feature, but rather just serves as an example of how community pipelines can be added (see https://github.com/huggingface/diffusers/issues/841).

### Stable Diffusion Interpolation

The following code can be run on a GPU of at least 8GB VRAM and should take approximately 5 minutes.

```python
from diffusers import DiffusionPipeline
import torch

pipe = DiffusionPipeline.from_pretrained(
    "CompVis/stable-diffusion-v1-4",
    revision='fp16',
    torch_dtype=torch.float16,
    safety_checker=None,  # Very important for videos...lots of false positives while interpolating
    custom_pipeline="interpolate_stable_diffusion",
).to('cuda')
pipe.enable_attention_slicing()

frame_filepaths = pipe.walk(
    prompts=['a dog', 'a cat', 'a horse'],
    seeds=[42, 1337, 1234],
    num_interpolation_steps=16,
    output_dir='./dreams',
    batch_size=4,
    height=512,
    width=512,
    guidance_scale=8.5,
    num_inference_steps=50,
)
```

The output of the `walk(...)` function returns a list of images saved under the folder as defined in `output_dir`. You can use these images to create videos of stable diffusion. 

> **Please have a look at https://github.com/nateraw/stable-diffusion-videos for more in-detail information on how to create videos using stable diffusion as well as more feature-complete functionality.**

### Stable Diffusion Mega

The Stable Diffusion Mega Pipeline lets you use the main use cases of the stable diffusion pipeline in a single class.

```python
#!/usr/bin/env python3
from diffusers import DiffusionPipeline
import PIL
import requests
from io import BytesIO
import torch


def download_image(url):
    response = requests.get(url)
    return PIL.Image.open(BytesIO(response.content)).convert("RGB")

pipe = DiffusionPipeline.from_pretrained("CompVis/stable-diffusion-v1-4", custom_pipeline="stable_diffusion_mega", torch_dtype=torch.float16, revision="fp16")
pipe.to("cuda")
pipe.enable_attention_slicing()


### Text-to-Image

images = pipe.text2img("An astronaut riding a horse").images

### Image-to-Image

init_image = download_image("https://raw.githubusercontent.com/CompVis/stable-diffusion/main/assets/stable-samples/img2img/sketch-mountains-input.jpg")

prompt = "A fantasy landscape, trending on artstation"

images = pipe.img2img(prompt=prompt, init_image=init_image, strength=0.75, guidance_scale=7.5).images

### Inpainting

img_url = "https://raw.githubusercontent.com/CompVis/latent-diffusion/main/data/inpainting_examples/overture-creations-5sI6fQgYIuo.png"
mask_url = "https://raw.githubusercontent.com/CompVis/latent-diffusion/main/data/inpainting_examples/overture-creations-5sI6fQgYIuo_mask.png"
init_image = download_image(img_url).resize((512, 512))
mask_image = download_image(mask_url).resize((512, 512))

prompt = "a cat sitting on a bench"
images = pipe.inpaint(prompt=prompt, init_image=init_image, mask_image=mask_image, strength=0.75).images
```

As shown above this one pipeline can run all both "text-to-image", "image-to-image", and "inpainting" in one pipeline.

### Long Prompt Weighting Stable Diffusion

The Pipeline lets you input prompt without 77 token length limit. And you can increase words weighting by using "()" or decrease words weighting by using "[]"
The Pipeline also lets you use the main use cases of the stable diffusion pipeline in a single class.

#### pytorch

```python
from diffusers import DiffusionPipeline
import torch

pipe = DiffusionPipeline.from_pretrained(
    'hakurei/waifu-diffusion',
    custom_pipeline="lpw_stable_diffusion",
    revision="fp16",
    torch_dtype=torch.float16
)
pipe=pipe.to("cuda")

prompt = "best_quality (1girl:1.3) bow bride brown_hair closed_mouth frilled_bow frilled_hair_tubes frills (full_body:1.3) fox_ear hair_bow hair_tubes happy hood japanese_clothes kimono long_sleeves red_bow smile solo tabi uchikake white_kimono wide_sleeves cherry_blossoms"
neg_prompt = "lowres, bad_anatomy, error_body, error_hair, error_arm, error_hands, bad_hands, error_fingers, bad_fingers, missing_fingers, error_legs, bad_legs, multiple_legs, missing_legs, error_lighting, error_shadow, error_reflection, text, error, extra_digit, fewer_digits, cropped, worst_quality, low_quality, normal_quality, jpeg_artifacts, signature, watermark, username, blurry"

pipe.text2img(prompt, negative_prompt=neg_prompt, width=512,height=512,max_embeddings_multiples=3).images[0]

```

#### onnxruntime

```python
from diffusers import DiffusionPipeline
import torch

pipe = DiffusionPipeline.from_pretrained(
    'CompVis/stable-diffusion-v1-4',
    custom_pipeline="lpw_stable_diffusion_onnx",
    revision="onnx",
    provider="CUDAExecutionProvider"
)

prompt = "a photo of an astronaut riding a horse on mars, best quality"
neg_prompt = "lowres, bad anatomy, error body, error hair, error arm, error hands, bad hands, error fingers, bad fingers, missing fingers, error legs, bad legs, multiple legs, missing legs, error lighting, error shadow, error reflection, text, error, extra digit, fewer digits, cropped, worst quality, low quality, normal quality, jpeg artifacts, signature, watermark, username, blurry"

pipe.text2img(prompt,negative_prompt=neg_prompt, width=512, height=512, max_embeddings_multiples=3).images[0]

```

if you see `Token indices sequence length is longer than the specified maximum sequence length for this model ( *** > 77 ) . Running this sequence through the model will result in indexing errors`. Do not worry, it is normal.

### Speech to Image

The following code can generate an image from an audio sample using pre-trained OpenAI whisper-small and Stable Diffusion.

```Python
import torch

import matplotlib.pyplot as plt
from datasets import load_dataset
from diffusers import DiffusionPipeline
from transformers import (
    WhisperForConditionalGeneration,
    WhisperProcessor,
)


device = "cuda" if torch.cuda.is_available() else "cpu"

ds = load_dataset("hf-internal-testing/librispeech_asr_dummy", "clean", split="validation")

audio_sample = ds[3]

text = audio_sample["text"].lower()
speech_data = audio_sample["audio"]["array"]

model = WhisperForConditionalGeneration.from_pretrained("openai/whisper-small").to(device)
processor = WhisperProcessor.from_pretrained("openai/whisper-small")

diffuser_pipeline = DiffusionPipeline.from_pretrained(
    "CompVis/stable-diffusion-v1-4",
    custom_pipeline="speech_to_image_diffusion",
    speech_model=model,
    speech_processor=processor,
    revision="fp16",
    torch_dtype=torch.float16,
)

diffuser_pipeline.enable_attention_slicing()
diffuser_pipeline = diffuser_pipeline.to(device)

output = diffuser_pipeline(speech_data)
plt.imshow(output.images[0])
```
This example produces the following image:

![image](https://user-images.githubusercontent.com/45072645/196901736-77d9c6fc-63ee-4072-90b0-dc8b903d63e3.png)

### Wildcard Stable Diffusion
Following the great examples from https://github.com/jtkelm2/stable-diffusion-webui-1/blob/master/scripts/wildcards.py and https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Custom-Scripts#wildcards, here's a minimal implementation that allows for users to add "wildcards", denoted by `__wildcard__` to prompts that are used as placeholders for randomly sampled values given by either a dictionary or a `.txt` file. For example:

Say we have a prompt:

```
prompt = "__animal__ sitting on a __object__ wearing a __clothing__"
```

We can then define possible values to be sampled for `animal`, `object`, and `clothing`. These can either be from a `.txt` with the same name as the category.

The possible values can also be defined / combined by using a dictionary like: `{"animal":["dog", "cat", mouse"]}`.

The actual pipeline works just like `StableDiffusionPipeline`, except the `__call__` method takes in:

`wildcard_files`: list of file paths for wild card replacement
`wildcard_option_dict`: dict with key as `wildcard` and values as a list of possible replacements
`num_prompt_samples`: number of prompts to sample, uniformly sampling wildcards

A full example:

create `animal.txt`, with contents like:

```
dog
cat
mouse
```

create `object.txt`, with contents like:

```
chair
sofa
bench
```

```python
from diffusers import DiffusionPipeline
import torch

pipe = DiffusionPipeline.from_pretrained(
    "CompVis/stable-diffusion-v1-4",
    custom_pipeline="wildcard_stable_diffusion",
    revision="fp16",
    torch_dtype=torch.float16,
)
prompt = "__animal__ sitting on a __object__ wearing a __clothing__"
out = pipe(
    prompt,
    wildcard_option_dict={
        "clothing":["hat", "shirt", "scarf", "beret"]
    },
    wildcard_files=["object.txt", "animal.txt"],
    num_prompt_samples=1
)
