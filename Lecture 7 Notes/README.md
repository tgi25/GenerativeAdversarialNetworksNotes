## Feature Extraction

### Original GAN

* For original GAN, the dimension of input vector *z* has no meaningful relationship on output
* Specific dimension of input vector *z* corresponds to certain meaningful characteristic in the generated image
* We hope the characteristics of image form distribution with pattern. 
* For example, if *z* have two 2 dimensions and the characteristics are colour. It might look like the following image
    
<img src="images/dist1.PNG" width="300"/>

* However, in truth different characteristic does not form distribution with pattern. Looks like following image, without any pattern at all.

<img src="images/dist2.PNG" width="300"/>

### InfoGAN

<img src="images/infogan.PNG" width="600"/>

* Combination of auto-encoder and GAN
* Auto-encoder consists of encoder and decoder
* The input vector **z** is split into two part **c** and **z'**
* The encoder plays the role of Generator 
* The encoder converts **z** to an image **x**
* The decoder (classifier) recovers the code **c** from **x**
* Discriminator is necessary
* Generator wants to help decoder to decode **c** from **x**
* Without discriminator, Generator may learns to copy **c** onto **x**, which is the easiest way for the decoder to recover **c** from **x**
* Having discriminator helps to avoid this behavior by checking whether the generated **x** is real or fake
* **c** must have **clear influence** on **x** so that the decoder can deduce **c** from **x**
* **c** : specific dimension corresponds to meaningful characteristic on output
* **z'** : Randomness
* Question : How do we know that **c**'s dimension represents certain meaning on output ?
* Answer : We do not know **c** has meaningful influence on output. But following InfoGAN training, the generator learns to generate output which is influenced by **c** 

#### InfoGAN result:

<img src="images/infogan_result.PNG" width="600"/>

### VAE-GAN
* Consists of Variational Auto-encoder (VAE) and GAN
* Can be seen as using VAE to strengthen GAN
* Can be seen as using GAN to strengthen VAE

<img src="images/vaegan.PNG" width="600"/>

* Encoder encodes images *x* to a vector *z*
* The distribution of *z* close to normal distribution
* Decoder reconstruct *x* from the vector *z*
* Minimize the reconstruction error
* Decoder also plays the role of Generator, to **fool** the Discriminator
* Discriminator differentiates real, generated and reconstructed images
* From VAE viewpoint :
    * VAE generates images which are blurry
    * Addition of Discriminator provide feedback to generator
    * Generator generates more realistic image

* From GAN viewpoint:
    * Generator is fed a randomly sampled vector
    * Generator has not seen any images
    * Will take a lot of effort to update parameters to learn how real images look like
    * Auto-encoder minimizes reconstruction error
    * Because decoder (generator) needs to reconstruct an image that looks like input (real) image from a vector, it knows how real image looks like
    * Combination of GAN and auto-encoder, the training is more stable

<img src="images/vaegan_algo.PNG" width="600"/>

* Update encoder to reduce reconstruction error
* Update generator to reduce reconstruction error, increase D(generated) and increase D(reconstructed)
* Update discriminator to increase D(real), decrease D(generated) and D(reconstructed)
* Another kind of discriminator, differentiates the input to discriminator is real, generated or reconstructed

## BiGAN

<img src="images/bigan.PNG" width="600"/>

* Consists of encoder and decoder
* Encoder encodes the real images into code *z*
* Decoder decodes the code *z* sampled from normal distribution into image
* This makes both encoder and decoder have their own pair of *z* and *x*
* The output of encoder is not used as input to decoder and vice versa
* Input and output of encoder and decoder are fed to the discriminator
* Discriminator differentiates whether the input-output pair comes from encoder or decoder

<img src="images/bigan_algo.PNG" width="600"/>

* Update discriminator to increase D(encoder_pair) and decrease D(decoder_pair)
* Update encoder and decoder to decrease D(encoder pair) and increase D(decoder pair)
* Can switch to increase or decrease encoder or decoder, the result is the same

<img src="images/bigan2.PNG" width="600"/>

* Discriminator: Evaluate two distributions are close or not (Divergence)
* Encoder's input and output form a joint distribution **P(x,z)**
* Decoder's input and output form a joint distribution **Q(x,z)**
* Wants P and Q to be closer 
* If the P and Q are the same, we would have optimal encoder and decoder with properties as shown in the figure above
* So why not train **two auto-encoders** ?
* The generated images would be blurry 
* For BiGAN, the output is different but realistic / clear
* For example, encoder converts an image of bird to code
* The decoder will not reconstruct the image of same bird, but rather generates an image of a different bird

## Domain Adversarial Training

<img src="images/domain.PNG" width="600"/>

* Training data and testing data are in different domains
* For example:
    * Training data: Grayscale
    * Testing data: Colour
* Train a generator which extracts features from the data from different domains 

<img src="images/domain2.PNG" width="600"/> 

* Domain classifier: Classify the feature is from which domain
* Label predictor: Classify the feature into class
    * For example, Digit classifier : 0, 1, 2, ..., 9 
* Feature extractor: Extract features, fools domain classifier and satisfy label predictor at the same time
* It is a big network
* Can train it part by part
* FGAN paper shows Generator and Discriminator can be trained together but the training is not stable
* Training goal :
    * Feature extractor : Maximize label classification accuracy + minimize domain classification accuracy
    * Label predictor : Maximize classification accuracy
    * Domain classifier : Maximize classification accuracy

### Feature Disentangle

#### Original Seq2Seq Auto-Encoder

<img src="images/s1.PNG" width="600"/> 

* The code consists of phonetic information, speaker information, background noise and more
* If able to isolate the phonetic and speaker information, would have many applications

<img src="images/s2.PNG" width="600"/> 

* Phonetic encoder isolates the phonetic information from the speech signal and removes speaker information
* Phonetic encoder can be plugged into speech recognition system
* Speaker encoder isolates speaker information while removes phonetic information
* Can be used for voiceprint recognition which is the task of recognizing who spoke


#### Training speaker encoder

<img src="images/s3.PNG" width="600"/> 

* Take a speech signal from same speaker
* Split it into two parts
* Fed into speaker encoder which generates vector
* We want both of the vector as close as possible

<img src="images/s4.PNG" width="600"/> 

* Take speech signal from different speakers
* Distance between two encoded vectors should not be smaller than a threshold

#### Training phonetic encoder

<img src="images/s5.PNG" width="600"/> 

* Inspired by domain adversarial training
* The speaker classifier try to differentiate the vectors are from same speaker or different speaker
* Give low score to different speaker
* Give high score to same speaker
* Phonetic encoder tries to confuse / fool the speaker classifier

#### Results

<img src="images/r1.PNG" width="300"/>

* From different speakers, words form clusters when void of speaker information

<img src="images/r2.PNG" width="300"/>

* Same word but different speaker

<img src="images/r3.PNG" width="300"/>

* Different speakers have similar phonetic information. Different speakers' pronunciation may be similar in some part

<img src="images/r4.PNG" width="300"/>

* Different speaker forms different clusters