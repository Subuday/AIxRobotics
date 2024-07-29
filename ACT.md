Today we are going to consider the neutral-network Action Chunking with Transformers (ACT). It is imitation learning algoritm that can demonstrate sucess rate  ~90%  in real-world tasks by having around 10mins of demonstratiions.       
### Notations

### Training

Imitation Learning
Data which is used for training is high quality user demonstrations created using ALOHA(describe here what is robot arm). Data is observation which is composed from current joint position (picture) of follower robot (describe why follower only robot) + image feed from 4 cameras. ACT trained to predict the seq of future actions given the curr observations. (picture?). Action is target joint positions for both arms. 

Action chunking 
Previous imitation learning algos have notable issues: errors in the policy can compound over time(why?) and human demonstrations can be non stationary(rephrase).
To combat problem of compounding error uses idea of action chunking. It is predicting the next k actions instead of a single action (why it works?).

Temporal ensemble
But naive implementation of action chunking can be result in jerky robot motion(explain why). To fix we can do action chunking on each timestamp, so it results that we will have number of different actions for the next timestamps. To combine this we can use temporal ensemble. Temporal ensemble  performs a weighted average over the predicted actions with an exponential weighting scheme w_i = exp(-m * i)., where w_0 is the weight of the oldest action.  m is speed of incroporating new observation is governed by m, where smaller m is faster means faster incroporation. 

When training a robot, it's crucial that the model learns to be very accurate in parts of the task where precision is critical, just like humans adjust their behavior to be more precise when necessary.
So action chunking policy is trained as conditional variational autoencoder (CVAE)

CVAE has two components: CVAE encoder(used only for training) and CVAE decoder. CVAE encoder predicts the mean and and variance of the style variable z's distribution, which is parametrized as a diagonal Gaussian, given the current observation and action seq as inputs.(explain better). For faster training in practive, we leave out the image observations and only condytion on the proprioceptive observation and the action seq. 

CVAE decoder conditions on both z and the current observations(images + joint positions) to predict the action seq. 

The whole model is trained to maximise the log likelyhod of demonstration action chunk... with standart VAE objective which has two terms: a rec loss and term that reguli the encoder to a gaussian prior.??? weight the second term in lesss information transimetter in z.

CVAE encoder and decoder are implemented with transformers.(why?) The CVAE encoder is implemented with BERT-like trasnformer encoder.

The inputs to the encoder are the current joint position and the target action seq of length k from demonst datasset. prepended by a learned "CLS" token.  After passing through the transofrmer the feature correspoinding to "[CLS]" is used to predict the mean and variance of the "style. variable" z (explain) which is then used as input to the decoder. CVAE decoder takes the current observations and z as the input and predicts the next k actions. 

ResNet is used as ResNet image encoder (transformer encoder) synthesis information from different camera viewpoints joint positions, style variable and transformer decoder to implement the CVAE decoder (why res net?) the transofmer decider generates a coherent action seq. 

The policy first processes the images with ResNet18 backbones with convert 480x640x3 rgb images into 15x20x512 feature maps. to preserve spatial dimension we add 2D sinusoidal position embedding to the feature seq(what does it mean?). Then is appended two more feature: the curr joint positions and "style variable" z. they are projected from their original dimensions to 512 thorugh linear laters respectively. the transformer decoder conditions on the encoder output through cross attention, where input seq is a fixed psotion embedding with dimension kx512, and the keys and values are coming from the encoder. the output of transformer decoder of k x 512 which is then down projected with an MLP into k x 14. corresponding to the predicted target joint positions for the next k steps. 

L1 loss is used fore construction. 
Use target joints positions instead of delta cause it degradates performance.

80M model.
5 hours training on RTX2080Ti GPU.
0.01 seconds inferene time.

### Inference

Only decoder is used.
At inference time, we set z to be the mean of prior distribution e.e zero to deterministically decoder (explain this)
