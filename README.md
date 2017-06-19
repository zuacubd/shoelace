# [![Shoelace](https://github.com/rjagerman/shoelace/wiki/img/logo.png)](https://github.com/rjagerman/shoelace)

[![Build Status](https://travis-ci.org/rjagerman/shoelace.svg?branch=master)](https://travis-ci.org/rjagerman/shoelace)
[![Coverage Status](https://coveralls.io/repos/github/rjagerman/shoelace/badge.svg?branch=master)](https://coveralls.io/github/rjagerman/shoelace?branch=master)

Shoelace is a neural Learning to Rank library using [Chainer](https://github.com/chainer/chainer). The goal is to make it easy to do offline learning to rank experiments on annotated learning to rank data.

## Features

### Dataset loading facilities

We currently provide ability to load learning to rank datasets (SVMRank format) into chainer.

    from shoelace.dataset.dataset import LtrDataset
    
    with open('./dataset.txt', 'r') as file:
        dataset = LtrDataset.load_txt(file)
        
Additionally, we provide minibatch iterators for Learning to Rank datasets. These generate variable-sized minibatches, where each minibatch represents one query and all associated query-document instances. You can additionally specify whether the iterator should repeat infinitely and/or shuffle the data on every epoch.

    from shoelace.dataset.iterator import LtrIterator
    
    iterator = LtrIterator(dataset, repeat=True, shuffle=True)

### Loss functions

Currently we provide implementations for the following loss functions

 * Top-1 ListNet: `shoelace.loss.listwise.ListNetLoss`
 * ListMLE: `shoelace.loss.listwise.ListMLELoss`
 * ListPL: `shoelace.loss.listwise.ListPLLoss`

## Example

Here is an example script that will train up a single-layer linear neural network with a ListNet loss function:

    from shoelace.dataset import LtrDataset
    from shoelace.iterator import LtrIterator
    from shoelace.loss.listwise import ListNetLoss
    from chainer import training, optimizers, links
    from chainer.training import extensions
    
    # Load data and set up iterator
    with open('./path/to/svmrank.txt', 'r') as f:
        training_set = LtrDataset.load_txt(f)
    training_iterator = LtrIterator(training_set, repeat=True, shuffle=True)
    
    # Create neural network with chainer and apply our loss function
    predictor = links.Linear(None, 1)
    loss = ListNetLoss(predictor)
    
    # Build optimizer, updater and trainer
    optimizer = optimizers.Adam()
    optimizer.setup(loss)
    updater = training.StandardUpdater(training_iterator, optimizer)
    trainer = training.Trainer(updater, (40, 'epoch'))
    trainer.extend(extensions.ProgressBar())
    
    # Train neural network
    trainer.run()

