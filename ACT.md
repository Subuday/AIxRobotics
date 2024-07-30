Today we are going to consider the neutral-network Action Chunking with Transformers (ACT). It is imitation learning algoritm that can demonstrate sucess rate  ~90%  in real-world tasks by having around 10mins of demonstratiions.       
##### Imitation Learning
Imitation learning is a technique where robots learn to perform tasks by mimicking human demonstrations.
[insert video images]

##### Imitation Learning Issues
Previous imitation learning algorithms encounter significant challenges by predicting only one action in time.
**Compounding errors:**  imagine a robot makes a small mistake at the start. Because of this, its next decision is based on an already slightly wrong situation, leading to another mistake. As it continues, each new decision is based on an increasingly incorrect scenario, causing these mistakes to add up. Over time, this can result in the robot's behavior being significantly off from what was intended.
**Non-stationary human demonstrations**: describe that a person can go in diff ways.
Human actions can vary greatly when accomplishing the same task

##### Imitation Learning Solutions
Action Chunking
Action chunking helps prevent error accumulation by predicting several future actions at once, instead of one at a time. However, this can lead to jerky movements since the robot doesn't continuously adjust its behavior based on real-time feedback at every timestamp.

Temporal ensemble
To fix jerky movements, we predict actions at each timestamp using action chunking. This gives us multiple action predictions for future timestamps. We can combine these predictions using temporal ensemble, which averages the actions.

Temporal ensemble
But naive implementation of action chunking can be result in jerky robot motion(explain why). To fix we can do action chunking on each timestamp, so it results that we will have number of different actions for the next timestamps. To combine this we can use temporal ensemble. Temporal ensemble  performs a weighted average over the predicted actions with an exponential weighting scheme w_i = exp(-m * i)., where w_0 is the weight of the oldest action.  m is speed of incroporating new observation is governed by m, where smaller m is faster means faster incroporation

[explain math behind weighting average]
[animation]

### Training
The training data is gathered using ALOHA, a robot arm system. This data is observations such as the current joint positions of the follower robot and image feeds from four cameras. ACT is then trained to predict the sequence of future actions based on the observations. Here, an action is defined as the target joint positions for both robot arms.
[picture]

ACT uses a conditional variational autoencoder (CVAE) to handle non-stationary human demonstrations. The CVAE learns various possible ways to perform actions, rather than just one fixed way.

The CVAE has two parts: an encoder (used only during training) and a decoder.
##### Encoder
The CVAE encoder predicts the mean and variance of the style variable z using current joint positions and intended actions. In the context of human demonstrations, z captures variations in speed, trajectory, or technique. For quicker training, we skip the images.

The encoder is implemented with a BERT-like transformer encoder. The inputs are "[CLS]" token, current joint positions and target actions. "[CLS]" is a special token at the start of an input sequence. It collects information about the relationships between all tokens in the sequence (the current joint positions and target actions) and is used to predict the mean and variance of the style variable z. 
[picture]

#### Decoder
The CVAE decoder uses the latent variable z, the images and the joint positions to predict the actions.

The decoder is implemented with ResNet image encoder, a transformer encoder, and a transformer decoder.ResNet processes images into features, which, along with current joint positions and the latent variable z, are input to the transformer encoder. The output is the encoded data containing relationships between these tokens. The transformer decoder uses this data to make predictions of next actions.
[picture]

#### 

The whole model is trained to maximise the log likelyhod of demonstration action chunk... with standart VAE objective which has two terms: a rec loss and term that reguli the encoder to a gaussian prior.??? weight the second term in lesss information transimetter in z.



L1 loss is used fore construction. 
Use target joints positions instead of delta cause it degradates performance.

80M model.
5 hours training on RTX2080Ti GPU.
0.01 seconds inferene time.

### Inference

Only decoder is used.
At inference time, we set z to be the mean of prior distribution e.e zero to deterministically decoder (explain this)
