We use the code for [prompt-to-prompt](https://prompt-to-prompt.github.io/) to analyze the attention in stable diffusion.


# Visualize Cross-Attention 

The way to visualize cross-attention between word tokens and image feature map is shown as follows


<img src="pictures/attention_vis_process.jpeg" alt="drawing" style="width:800px;"/>

Visualization exampels:

<img src="pictures/attention_vis_example1.jpg" alt="drawing" style="width:800px;"/>
<img src="pictures/attention_vis_example2.jpg" alt="drawing" style="width:800px;"/>


# no control for vanilla stable diffusion

We repalce 'squirrel' with 'lion' in the input prompot, there is no local control on the generated images even with same random seed. A small modification of the text prompt often leads to a completely different outcome.

<img src="pictures/without_p2p.jpg" alt="drawing" style="width:400px;"/>

My own ayalysis: the early steps in the decoding process control the global structure of the generated image. The input is a sample nosie feature, we do the cross attention map between it (actuall it's feature map after Conv) and tokens. The size of the attention map is N x 77 (77 is the length of the words tokens.). When we replace 'squirrel' with 'lion', if we consider the first cross attention, the vector multiplication between unchanged tokens and features is not changed, actually we replace a column (correspoding to "squirrel") value with new values. Eventhough we only change a single column, but if we consider each row, many pixels most attended words are changed. Then this change in the first step is enlarged by following decoding steps.



# cross attention edit for control

### 1. replacement edit
In order to only change the replaced word region, prompt-to-prompt propsed to use the original attention map to swap with the new attention map (following figure). For previous example, the region attend to "squirrel" will attend to 'lion', and the location of other region keep unchanged. Thats why it can do local replacement edit.

<img src="pictures/word_swap.jpg" alt="drawing" style="width:200px;"/>

The replacement edit results:

<img src="pictures/replace_edit1.jpg" alt="drawing" style="width:400px;"/>

**One issue**: look at burgers, they are also changed when we only want to change squirrel to lion. We use the original attention map for cross attention and only the 'squirrel' region should changed to 'lion'. But we have other operations, like the convlution (also self attention), conv has gredually increasing receptive field. Even we only changed the 'squirrel' region for the feature after cross atteion, it also has small affects for other regions due to large receptive field of following convlution and self-attention operations. 

There is one smart and easy soluiton was proposed in prompt-to-prompt  (Algorithm 1 L11-14 in paper) called ***local blend***. Using the attention map as mask, and then use the origianl feature map values for the regions that we don't want to edit (like burgers). The updated result is showed as follows. Since the mask  used is not 100% accurate, burgers are not 100% unchanged. But much better than before.

<img src="pictures/replace_edit2.jpg" alt="drawing" style="width:400px;"/>


The edit operations can be applied to from 0% layers and 100% layers. I think it's a good parameter to tune and depends on the difference degree between replace words. The following image is from the original paper:


<img src="pictures/replace_edit3.jpg" alt="drawing" style="width:1200px;"/>




### 2. Refinement edit
Use the original attention maps and inject the attention map layers for the new words from the new attention maps.

<img src="pictures/prompt_refinement.jpg" alt="drawing" style="width:200px;"/>

The result:


<img src="pictures/refine_edit1.jpg" alt="drawing" style="width:400px;"/>

But I find the refinement edit fails in many cases, like following example, I think the injection operations affects attention relations between feature map and tokens, that's why this operation is not accurate as others.

<img src="pictures/refine_edit2.jpg" alt="drawing" style="width:400px;"/>

It also perform bad when we want to add a new object, like following example. I think it makes sense because if we don't want to change the structre of the image, there is no place to generate build. Look at the 3rd image generated without p2p control, the 'build' attend large region in the image, when inject that attention layer to cross attention by using p2p control (2nd image), there are many regions should attend to 'building', but some of them are located by 'butterfly', so it can't generate a good 'building' without enough regions in 2nd image. I think that's why it generated similar color in 2nd image without clear bulding structure.

<img src="pictures/refine_edit3.jpg" alt="drawing" style="width:600px;"/>


### 3.  Re-Weighting edit 

We can simply re-weighting the cross attention map layer emphasize or un-emphasize a word.

<img src="pictures/attention_reweight.jpg" alt="drawing" style="width:200px;"/>

We can definitely use the local blend too. The re-weighting edit results w/o local blend is shown as follows:


<img src="pictures/reweight_edit1.jpg" alt="drawing" style="width:1200px;"/>
