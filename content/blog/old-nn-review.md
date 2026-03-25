+++
date = '2025-03-23T20:38:46-06:00'
title = 'Old Nn Review'
draft = true
+++

# Context

When I was just learning how to program I tried doing alot of things that were
far outside the reach of my ability. Which resulted in me spending alot of time
in StackOverflow threads and watching tutorial videos on YouTube.

On that note six years ago I tried writing and training a convolutional neural
network in C++ from scratch to learn XOR following along roughly with [some
random video](https://vimeo.com/19569529). I'm now taking a class on
introductory ML and have actually know linear algebra and calculus to a degree
now so I suspect I'll have a few new insights :)

# Original

Ok here are the files `Neuron.cpp`:
```cpp
#include <math.h>
#include <assert.h>
#include <cstdlib> 
#include <ctime> 
#include <iostream>
#include "Neuron.hpp"

using gfd::Layer;

namespace gfd{

  Neuron::Neuron(unsigned numWeights, unsigned index) : _index(index){
    for(unsigned w = 0; w < numWeights; w++){
      _outputWeights.push_back(Weight());
      _outputWeights.back().weight = f_GenerateRandomWeight();
    }
  }

  void Neuron::setActivation(double val){
    _activation = val;
  }

  double Neuron::getActivation() const{
    return _activation;
  }

  void Neuron::feedForward(const Layer& previousLayer){
    double feedForwardVal = 0.0;
    for(int n = 0; n < previousLayer.size(); n++){
      feedForwardVal += previousLayer[n].getActivation() * previousLayer[n]._outputWeights[_index].weight;
    }
    _activation = f_Activation(feedForwardVal);
  }

  void Neuron::backPropagation(Layer &previousLayer, double currentGradientCalc){
    double z;
    for(unsigned pln = 0; pln < previousLayer.size(); pln++){
          z = previousLayer[pln].getActivation();// How much the output changes in relation to the other neuron connected to the weight
          previousLayer[pln]._outputWeights[_index].weight += (currentGradientCalc * z) * Neuron::learningRate;
    }
  }

  double Neuron::f_GenerateRandomWeight(){
    srand(time(0));
    return rand() / double(RAND_MAX);
  }

  double Neuron::f_Activation(double x){
    return tanh(x);
  }

}
```

`Network.cpp`:
```cpp
#include "Network.hpp"
#include <assert.h>
#include <cmath>

namespace gfd {
Network::Network(std::vector<unsigned> topology) {
  for (unsigned l = 0; l < topology.size(); l++) {
    _network.push_back(Layer());
    unsigned numOutputs = l == topology.size() - 1 ? 0 : topology[l + 1];
    std::cout << "\nLayer [" << l << "] created:";
    std::cout << "\n[ Number of neurons recieved for creation: " << topology[l]
              << " ]";
    int created = 0;
    for (unsigned n = 0; n < topology[l]; n++) {
      _network.back().push_back(Neuron(numOutputs, n));
      std::cout << "\n\t{Neuron created}";
      created += 1;
    }
    std::cout << "\n" << created << " Neurons Created\n";
    std::cout << "<------------------------------------------------------>";
    epoch = 0;
  }
}

void Network::feedForward(const std::vector<double> inputs) {
  assert(inputs.size() == _network[0].size());

  for (unsigned n = 0; n < inputs.size(); n++) {
    _network[0][n].setActivation(inputs[n]);
  }

  for (unsigned l = 1; l < _network.size(); l++) {
    for (unsigned n = 0; n < _network[l].size(); n++) {
      _network[l][n].feedForward(_network[l - 1]);
    }
  }

  std::vector<double> networkOutputs;
  for (unsigned n = 0; n < _network[_network.size() - 1].size(); n++) {
    networkOutputs.push_back(_network[_network.size() - 1][n].getActivation());
  }
  output = networkOutputs;
}

void Network::f_backPropagation(const std::vector<double> targets) {
  assert(targets.size() == _network[_network.size() - 1].size());
  double x, y, z;
  for (unsigned l = _network.size() - 1; l > 0; l--) {
    for (unsigned n = 0; n < _network[l].size(); n++) {
      x = -(targets[n] - _network[l][n].getActivation());
      y = _network[l][n].getActivation() *
          (-_network[l][n].getActivation()); // Logistics function derivative
      _network[l][n].backPropagation(_network[l - 1], (double)(x * y));
    }
  }
}

void Network::train(const std::vector<double> inputs,
                    const std::vector<double> targets) {
  feedForward(inputs);
  f_backPropagation(targets);
}

void Network::printNetwork() {
  for (int i = 0; i < _network.size(); i++) {
    std::cout << "\nLayer " << i << ":";
    for (int j = 0; j < _network[i].size(); j++) {
      std::cout << " [" << _network[i][j].getActivation() << "] ";
    }
  }
}

} // namespace gfd
```

# Feed Foward Review
```cpp
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

Ok this is conceptually sound but really quite a terrible way to go about the
implementation. If I had to write this today I would much prefer something
like:

```cpp
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

Then overload multplication between vectors and vectors of vectors so we can
more compactly and compute the output of each layer. Though I have not profiled
I'm quite confident that this is significantly faster than the previous
implementaiton cause we don't copy the inputs and results in and out of the
network AND as long as the matrix multiplication is done properly you won't be
constantly evicting things from the cache. (This may not have mattered for my
case since the network needed to learn XOR is relativly small). Also the extra
`Neuron` class is frankly quite clunky and I don't really like it.

# Backprop Review
**Original:**
```cpp 
void Neuron::backPropagation(Layer &previousLayer, double currentGradientCalc){
  double z;
  for(unsigned pln = 0; pln < previousLayer.size(); pln++){
        z = previousLayer[pln].getActivation();// How much the output changes in relation to the other neuron connected to the weight
        previousLayer[pln]._outputWeights[_index].weight += (currentGradientCalc * z) * Neuron::learningRate;
  }
}

void Network::f_backPropagation(const std::vector<double> targets) {
  assert(targets.size() == _network[_network.size() - 1].size());
  double x, y, z;
  for (unsigned l = _network.size() - 1; l > 0; l--) {
    for (unsigned n = 0; n < _network[l].size(); n++) {
      x = -(targets[n] - _network[l][n].getActivation());
      y = _network[l][n].getActivation() *
          (-_network[l][n].getActivation()); // Logistics function derivative
      _network[l][n].backPropagation(_network[l - 1], (double)(x * y));
    }
  }
}
```

When learning XOR you are training a binary classifier. I didn't really know
the distinction between different types of tasks and how activation and loss
functions should be chosen to support those when I copied/wrote this. Somehow I
ended up with the correct activation function for the output. I also managed to
realize that you needed to go backwards over model layers to propagate errors.
Thats about where the correctness ends.

Now the issues:
- I presume `x` is supposed to be the derivative of the loss function? That
  would make sense because they are chain ruled together to create the
  `currentGradientCalc` passed to the node. We should only be computing the
  erorr using the final targets for the last layers. The error for the earlier
  layers will be computed by the gradient.
- We should be using binary cross-entropy loss for classification. That looks
  like MSE--`1/2 (target - actual)^2`.
- The derivative for the logistic function is wrong. It should be `sigmoid(x) *
  (1 - sigmoid(x))` not `-sigmoid^2(x)`. The next problem is that we are not
  propagating the error from the previous layer.
- In `Neuron::backPropagation` that would be gradient ascent if not for the
  mistaken logistic function derivative.
- This phrasing is very verbose and has a high mental overhead to think about I
  would prefer to read this phrased in terms of vectors and matrices.

**Corrected:**


```cpp

```
