---
layout: post
title: Some LHC Calculations
---

The LHC is currently running at 7 TeV, giving a relativistic gamma factor of 3730:

$$E = \gamma mc^2 \rightarrow \gamma = {E \over mc^2} \rightarrow \gamma(3.5\text{TeV}) = 3730$$

This means the protons are moving at 99.999996% the speed of light:

$$\gamma = {1 \over \sqrt{1 - \beta^2}} \rightarrow \beta = \sqrt{1 - {1 \over \gamma^2}} \approx 0.99999996$$

Alright, now let's look at the collision rate, assuming a recent luminosity of 1e33:

$$R_\text{total} = \mathcal{L} \sigma = (10^{33} {\text{cm}^{-2} \over \text{s}})(110\text{mb})({10^{-27}\text{cm}^2 \over \text{mb}}) \approx 100 \text{MHz of interactions}$$

The cross section for the events I'm looking at is $ \sigma_{WZ\rightarrow 3\ell\nu} \approx 0.5 \text{pb}$, so

$$R(WZ\to 3\ell\nu) = (10^{33} {\text{cm}^{-2} \over \text{s}})(0.5\text{pb})({10^{-36}\text{cm}^2 \over \text{mb}}) = 0.0005 \text{Hz} = \text{40 per day}$$

So how many of these events will we have collected by the end of the 2011 pp run?

$$N_{WZ\rightarrow 3\ell\nu} \approx (5 {\text{fb}^{-1}}) (0.5 \text{pb}) = \text{2500 events}$$

By the time we make all the required cuts to account for the acceptance of the detector and rejection of backgrounds, however, we end up with something on the order of only 100 events in our final sample.
