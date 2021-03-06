# Keras Mixture Density Network Layer

[![DOI](https://zenodo.org/badge/137585470.svg)](https://zenodo.org/badge/latestdoi/137585470)

A mixture density network (MDN) Layer for Keras using TensorFlow's distributions module. This makes it a bit more simple to experiment with neural networks that predict multiple real-valued variables that can take on multiple equally likely values.

This layer can help build MDN-RNNs similar to those used in [RoboJam](https://github.com/cpmpercussion/robojam), [Sketch-RNN](https://experiments.withgoogle.com/sketch-rnn-demo), [handwriting generation](https://distill.pub/2016/handwriting/), and maybe even [world models](https://worldmodels.github.io). You can do a lot of cool stuff with MDNs!

One benefit of this implementation is that you can predict any number of real-values. TensorFlow's `Mixture`, `Categorical`, and `MultivariateNormalDiag` distribution functions are used to generate the loss function (the probability density function of a mixture of multivariate normal distributions with a diagonal covariance matrix). In previous work, the loss function has often been specified by hand which is fine for 1D or 2D prediction, but becomes a bit more annoying after that.

Two important functions are provided for training and prediction:

- `get_mixture_loss_func(output_dim, num_mixtures)`: This function generates a loss function with the correct output dimensiona and number of mixtures.
- `sample_from_output(params, output_dim, num_mixtures, temp=1.0)`: This functions samples from the mixture distribution output by the model.

## Examples

Some examples are provided in the notebooks directory.

There's scripts for fitting multivalued functions, a standard MDN toy problem:

<img src="https://preview.ibb.co/mZzkpd/Keras_MDN_Demo.jpg" alt="Keras MDN Demo" border="0">

There's also a script for generating fake kanji characters:

<img src="https://image.ibb.co/hVTOw8/kanji_test_1.png" alt="kanji test 1" border="0" width="200"/>

And finally, for learning how to generate musical touch-screen performances with a temporal component:

![Robojam Model Examples](https://preview.ibb.co/mpfa9T/robojam_examples.jpg)

## How to use

The MDN layer should be the last in your network and you should use `get_mixture_loss_func` to generate a loss function. Here's an example of a simple network with one Dense layer followed by the MDN.

    import keras
    import mdn

    N_HIDDEN = 15  # number of hidden units in the Dense layer
    N_MIXES = 10  # number of mixture components
    OUTPUT_DIMS = 2  # number of real-values predicted by each mixture component

    model = keras.Sequential()
    model.add(keras.layers.Dense(N_HIDDEN, batch_input_shape=(None, 1), activation='relu'))
    model.add(mdn.MDN(OUTPUT_DIMS, N_MIXES))
    model.compile(loss=mdn.get_mixture_loss_func(OUTPUT_DIMS,N_MIXES), optimizer=keras.optimizers.Adam())
    model.summary()

Fit as normal:

    history = model.fit(x=x_train, y=y_train)

The predictions from the network are parameters of the mixture models, so you have to apply the `sample_from_output` function to generate samples.

    y_test = model.predict(x_test)
    y_samples = np.apply_along_axis(sample_from_output, 1, y_test, OUTPUT_DIMS, N_MIXES, temp=1.0)

See the notebooks directory for examples in jupyter notebooks!

## Acknowledgements

- Hat tip to [Omimo's Keras MDN layer](https://github.com/omimo/Keras-MDN) for a starting point for this code.
- Super hat tip to [hardmaru's MDN explanation, projects, and good ideas for sampling functions](http://blog.otoro.net/2015/11/24/mixture-density-networks-with-tensorflow/) etc.
- Many good ideas from [Axel Brando's Master's Thesis](https://github.com/axelbrando/Mixture-Density-Networks-for-distribution-and-uncertainty-estimation)
- Mixture Density Networks in Edward [tutorial](http://edwardlib.org/tutorials/mixture-density-network).

## References

1. Christopher M. Bishop. 1994. Mixture Density Networks. [Technical Report NCRG/94/004](http://publications.aston.ac.uk/373/). Neural Computing Research Group, Aston University. http://publications.aston.ac.uk/373/
2. Axel Brando. 2017. Mixture Density Networks (MDN) for distribution and uncertainty estimation. Master’s thesis. Universitat Politècnica de Catalunya.
3. A. Graves. 2013. Generating Sequences With Recurrent Neural Networks. ArXiv e-prints (Aug. 2013). https://arxiv.org/abs/1308.0850
4. David Ha and Douglas Eck. 2017. A Neural Representation of Sketch Drawings. ArXiv e-prints (April 2017). https://arxiv.org/abs/1704.03477
5. Charles P. Martin and Jim Torresen. 2018. RoboJam: A Musical Mixture Density Network for Collaborative Touchscreen Interaction. In Evolutionary and Biologically Inspired Music, Sound, Art and Design: EvoMUSART ’18, A. Liapis et al. (Ed.). Lecture Notes in Computer Science, Vol. 10783. Springer International Publishing. DOI:[10.1007/9778-3-319-77583-8_11](http://dx.doi.org/10.1007/9778-3-319-77583-8_11)
