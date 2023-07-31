# AIGC model analysis


In order to better understand different AIGC related models like stable diffusion, we use this repo to analyze them.


 
1. [stable diffusion cross attention analysis](https://github.com/cloudwishper/AIGC_model_analysis/tree/main/stable_diffusion_cross_attention)

    Prompt-to-prompt ICLR 2023 is a good paper for me to understand the cross-attention in the stable diffusion model. The cross attention map represent the layout of the generated structure, and the remaining operations like attention * value, residual block, conv can help the model generated correct object in the location decided by the attention map.

# Paper reading

| paper  | resource |
| ------------- | ------------- |
| High-Resolution Image Synthesis with Latent Diffusion Models (stable diffusion)  | CVPR 2022  |
| DreamBooth: Fine Tuning Text-to-Image Diffusion Models for Subject-Driven Generation | CVPR 2023 |
|Prompt-to-Prompt Image Editing with Cross-Attention Control |ICLR 2023 |
|Expressive Text-to-Image Generation with Rich Text |ICCV 2023 |
