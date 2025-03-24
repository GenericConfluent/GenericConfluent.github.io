+++
date = '2025-03-23T20:38:46-06:00'
title = 'Old Nn Review'
+++

# Context
When I was just learning how to program I tried doing alot of things that were far outside the reach of my ability. Which resulted in me spending alot of time in StackOverflow threads and watching tutorial videos on YouTube. 

On that note six years ago I tried writing and training a convolutional neural network in C++ from scratch to learn XOR following along roughly with [some random video](https://vimeo.com/19569529). I'm now taking a class on introductory ML and have actually know linear algebra and calculus to a degree now so I suspect I'll have a few new insights :)

# Old Code New Cringe
## Feed Foward

```C++
void Neuron::feedForward(const Layer& previousLayer){
    double feedForwardVal = 0.0;
    for(int n = 0; n < previousLayer.size(); n++){
        feedForwardVal += previousLayer[n].getActivation() * previousLayer[n]._outputWeights[_index].weight;
    }
    _activation = f_Activation(feedForwardVal);
} 

void Network::feedForward(const std::vector<double> inputs){
    assert(inputs.size() == _network[0].size());

    for(unsigned n = 0; n < inputs.size(); n++){
        _network[0][n].setActivation(inputs[n]);
    }

    for(unsigned l = 1; l < _network.size(); l++){
        for(unsigned n = 0; n < _network[l].size(); n++){
        _network[l][n].feedForward(_network[l-1]);
        }
    }

    std::vector<double> networkOutputs;
    for(unsigned n = 0; n < _network[_network.size() - 1].size(); n++){
        networkOutputs.push_back(_network[_network.size() - 1][n].getActivation());
    }
    output = networkOutputs;
}
```

Ok this is conceptually sound but really quite a terrible way to go about the implementation. If I had to write this today I would much prefer something like:

```C++
typedef std::vector<double> vd;

const vd& Network::feed_forward(const vd &inputs){
    for (unsigned layer = 0; layer < network.size(); ++layer) {
        const vd &previous = layer == 0 ? inputs : network[layer - 1];

        network[layer] = weights[layer - 1] * previous;

        for (auto &value : network[layer]) {
            value = Network::activation(value);
        }
    }

    return network.back();
}
```

Then overload multplication between vectors and vectors of vectors so we can more compactly and compute the output of each layer. Though I have not profiled I'm quite confident that this is significantly faster than the previous implementaiton cause we don't copy the inputs and results in and out of the network AND as long as the matrix multiplication is done properly you won't be constantly evicting things from the cache. (This may not have mattered for my case since the network needed to learn XOR is relativly small). Also the extra `Neuron` class is frankly quite clunky and I don't really like it?
