# Temporal-Ensembling-for-Semi-Supervised-Learning (UNDER WORK)

This repository includes a implementation of Temportal Ensembling for Semi-Supervised Learning by Laine et al. with Tensorflow eager execution.

When I was reading "Realistic Evaluation of Deep Semi-Supervised Learning Algorithms" by Avital Oliver (2018), I realized I had never played enough with Semi-Supervised Learning, so I came across this paper and thought it was interesting for me to play with. (I highly recommend reading the paper by Avital et al., one of my favorite recent papers).

## Semi-Supervised Learning

Semi-Supervised Learning algorithms try improving traditional supervised learning ones by using unlabeled samples. This is very interesting because in real-world there are a big amount of problems where we have a lot of data that is unlabeled. There is no valid reason why this data cannot be used to learn the general structure of the dataset by supporting the learning process of the supervised training. 

## Self-Ensembling

The paper propose two implementations of self-ensembling, i.e. forming different ensemble predictions in the training process under different conditions of regularization (dropout) and augmentations.

The two different methods are $\Pi$-model and temporal ensembling. Let's dive a little bit into each one of them. 

<p align="center">
  <img src="https://user-images.githubusercontent.com/10371630/41501166-caf8a93c-7196-11e8-9072-19dd6968c8e6.png" alt="temporal-ensembling"/>
</p>

### Pi Model

In the ![Pi]-Model the training inputs ![xi] are evaluated twice, under different conditions of dropout regularization and augmentations, resulting in tow outputs ![zi] and ![tilde_zi]. 

The loss function here includes two components:
- **Supervised Component**: standard cross-entropy between ground-truth ![yi] and predicted values ![zi] (only applied to labeled inputs).
- **Unsupervised Component**: penalization of different outputs for the same input under different augmentations and dropout conditions by mean square difference minimization between ![zi] and ![tilde_zi]. This component is applied to all inputs (labeled and unlabeled).

This two components are combined by summing both components and scaling the unsupervised one using a time-dependent weighting function ![w_t]. According to the paper, asking ![zi] and ![tilde_zi] to be close is a much more string requirement than traditional supervised cross-entropy loss. 

The augmentations and dropouts are always randomly different for each input resulting in different output predictions. Additionally to the augmentations, the inputs are also combined with random gaussian noise to increase the variability. 

The weighting function ![w_t] will be described later since is also used by temporal ensembling, but it ramps up from zero and increases the contribution from the unsupervised component reaching its maximum by 80 epochs. This means that initially the loss and the gradients are mainly dominated by the supervised component (the authors found that this slow ramp-up is important to keep the gradients stable). 

One big difficulty of the ![Pi]-Model is that it relies on the output predictions that can be quite unstable during the training process. To combat this instability the authors propose the temporal ensembling.

### Temporal Ensembling

Trying to resolve the problem of the noisy predictions during train, temporal ensembling aggregates the predictions of past predictions into an ensemble prediction. 

Instead of evaluating each input twice, the predictions ![zi] are forced to be close the a ensemble prediction ![Big_z_tilde], that is based on previous predictions of the network. This algorithm stores each prediction vector ![zi] and in the end of each epoch, these are accumulated in the ensemble vector ![tilde_zi] by using the formula:

![ensemble_form]

where $\alpha$ is a term that controls how far past predictions influence the temporal ensemble. This vector contains a weighted average of previous predictions for all instances, with recent ones having a higher weight. The ensemble training targets ![tilde_zi], to be comparable to ![zi] need to be scaled by dividing them by ![correction_form].

In this algorithm, ![Big_z] and ![tilde_zi] are zero on the first epoch, since no past predictions exist. 

The advantages of this algorithm when compared to the ![Pi]-Model is:
- Training is approximately 2x faster (only one evaluation per input each epoch)
- The ![tilde_zi] is less noisy than in ![Pi]-Model.

The disadvantages are the following:
- The predictions need to be stored across epochs.
- A new hyperparameter ![alpha] is introduced.

(UNDER WORK)

## References
- Laine, Samuli, and Timo Aila. "Temporal ensembling for semi-supervised learning." arXiv preprint arXiv:1610.02242 (2016).
- Oliver, Avital, et al. "Realistic Evaluation of Deep Semi-Supervised Learning Algorithms." arXiv preprint arXiv:1804.09170 (2018).

## Credits

I would like to give credit to some repositories that I found while reading the paper that helped me in my implementation.

- [Original Implementation in Lasagne](https://github.com/smlaine2/tempens)
- [NVIDIA’s Π Model from “Temporal Ensembling for Semi-Supervised Learning” (ICLR 2017) with TensorFlow.](https://github.com/geosada/PI)
- [John Farret blog post with Pytorch Code on MNIST dataset](https://ferretj.github.io/ml/2018/01/22/temporal-ensembling.html)

[Pi]: http://chart.apis.google.com/chart?cht=tx&chl=\Pi
[zi]: http://chart.apis.google.com/chart?cht=tx&chl=z_i
[xi]: http://chart.apis.google.com/chart?cht=tx&chl=x_i
[tilde_zi]: http://chart.apis.google.com/chart?cht=tx&chl=\tilde{z}_i
[yi]: http://chart.apis.google.com/chart?cht=tx&chl=y_i
[Big_z]: http://chart.apis.google.com/chart?cht=tx&chl=Z
[Big_z_tilde]: http://chart.apis.google.com/chart?cht=tx&chl=\tilde{Z}_i
[ensemble_form]: http://chart.apis.google.com/chart?cht=tx&chl={Z}_i=\alpha\tilde{Z}_i+(1-\alpha)z_i
[w_t]: http://chart.apis.google.com/chart?cht=tx&chl=w(t)
[alpha]: http://chart.apis.google.com/chart?cht=tx&chl=\alpha
[correction_form]: http://chart.apis.google.com/chart?cht=tx&chl=(1-\alpha^{t})
