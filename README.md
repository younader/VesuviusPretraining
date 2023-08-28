# Vesuvius Pretraining

This repository contains my approach and exploration of pretraining effects in the Vesuvius challenge. The idea is to pretrain the model on the ink scrolls that we want to read, then finetune on the kaggle data in order to bridge the gap between the scrolls and the fragments.


## Approach
1- 3d Resnet 50 depth
2- z-translation of 1-2 slices
3- augmentations similar to kaggle solutions, with a few more holes in the images
4- finetuning on smaller output sizes
5- inference without outer edges
Technically, you can pretrain any models of your choice, I found 3d resnets to be good/easy enough to use. 
## How to pretrain?
My first idea for pretraining is to employ self-supervised learning to learn some good representations from the scrolls before training on ink. SSL typically learns representations by trying to align 2 different views of the same image. One of the major challenges of this approach is information collapse, where instead of outputing a meaningful representation the model just outputs a fixed vector for everything, so that everything is similar. SSL employs various techniques to counter this issue but it's even harder in this problem, mainly because the ink data represents a small portion of the fragment (in the kaggle framents somewhere around 11%).
I tried 2 approaches BYOL and SimSiam and they failed miserabely, but then I stumbled across VICReg. 
VICReg employs explicit regularization to the embeddings to prevent collapse. 
One sanity check that there is learning going on is to compare the similarity of the representations learned by different epochs, using something like CKA. 
![CKA](/images/download%20(31).png)
<span class="caption">CKA Similarity, linear on the right and rbf kernel on the left, both show that the similarity of the representations is changing the more we train</span>

I pretrained a 3dresnet of depth 50 on 38 segments (quality 4/3)+ 4th kaggle fragment,
and present my findings bellow.
Due to computation resources, I only pretrained for 17 epochs (SSL training recommends 100-200 epochs) on image size 128x128.
pretrained weights can be found [placeholder until the upload goes through].
## How to know pretraining works?

The simplest and possibly most effective way of verifying pretraining works is to freeze the encoder and see how good it is to detect ink on the fragments.
I compare the pretraining to a random encoder and a kinetic-700 pretrained encoder
There are 2 questions to explore:
##### 1- how good are the learned representations detecting ink on the fragments?
###### data: train: 1,3,4, 70% of 2        , validation: lower 30% of 2
###### results:  
![Random Baseline](/images/random_frozen.png)
<span class="caption">Random Baseline</span>
random baseline tests what the decoder can learn without a good encoder. In my experiments I noticed that an encoder can overcome a random initialized frozen decoder, but the other way is much harder, so I just wanted to make sure that the model's actually learning and not the decoder doing heavy lifting, hence this baseline :\)
![k700 pretrained](/images/k700frozen.png)
<span class="caption">k700 pretrained</span>
The kinetics-700 pretrained model was shown in the kaggle 4th place solution, and has quite a positive influence on the training the models, so I wanted to compare against my pretrained model.
![segments pretraining](/images/pretraining_frozen.png)
<span class="caption">segments VICREG pretraining</span>

##### 2- how well does pretraining generalize to the pretrained fragments?
###### data: train: 1,2,3       , validation: 4
###### results:
![segments pretraining](/images/masks_1360_45dbc1067b345419759a.png)
prediction on the 4th fragment (I skipped patches where no ink is present in evaluation, although perhaps I should keep them in the future)
The signal is much messier and weaker, but is present in some regions (last and first lines if you compare with the ink labels).
## Notes
I expect the signal of ink to get better after longer pretraining, since it's a finer detail. I observed that while freezing the encoder, showing the decoder parts of a fragment really helps filter out noise/shape up the signal, as seen in the previous experiment. 

## So where to go from here?
From my experiments, the output of pretraining hints at letter locations and few letters, but leaves more to be desired. 
My current priority is to long train one of the models, somewhere around 100 epochs should be good. 
I believe there are a few challenges that need to be tackled before we can start fully seeing letters. 
One of these major challenges is how to handle the distribution shift, I found that normalization helps however there are different ways to do it and they all lead to different results.
There are also other pretraining techniques to try than contrastive learning out there, to be honest I didn't believe contrastive learning would show any hints of life, but here we are ¯\\\_(ツ)_/¯

