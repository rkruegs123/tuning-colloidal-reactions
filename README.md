# jax-catalyst-design

Code for designing **spider catalysts**

## Jan. 17, 2023

For next time:
- test on GPU
  - note that we'll need a local version of the jax-md changes that we did. Will likely want to make a conda environment that has its wn install of jax-md
  - changes are `union_to_points` and in `RigidPointUnion.__getitem__`
- setup optimization
  - note that an initial thing we can do to avoid dummy local minima (i.e. having a huge spider catalyst to splay apart the shell vertices via excluded volume) is to make the initialized height of the catalyst a function of its two radii (well, really the max of something like (head_radius - height, leg_radius), as well as a function of base radius
  - note: could setup two optimizations -- one that only optimizes over energy, another that optimizes over both shape and energy


## Jan. 18, 2023

From today
- Tried to setup a training loop but didn't actually run it
  - need to add a batch size (at the least)
- were getting nans for some spider shape stuff. added some noise to msases and got non-nan grads. but then re-ran on cluster and got 0 gradients for some more things. Maybe we didn't originally save the flie or osmetihng? Maybe we should run again locally? Maybe we didn't run for long enough? Some stocahsticity there? Have to experiment
  - maybe should install the github rpeo on a colab and visualize with the new masses. unequal masses may be messing pu the initialization
- once we resolve the above, just setup a training loop and hit go
  -


## Jan. 29, 2023

today we banged our heads against the wall and reminimized the icosahedron and wokred out that we got both really luckky by choosing the wrong vertex to bind that was the problem one, and also unlucky in the sense that our minimization hadn't worked

so, we reminimized. and changed some +'s to -'s, which we are still confused about. This had to be done when doing the dispalcmenets for (i) the vertex shape (so that the patches weren't facing outwards) *and* for the spider head (so that the spider head wasn't facing inwards). This suggests that our optimization from before had the spider head inside and was just splaying everything evyerwhere. It also suggests that our minimized icosahedron from beforee was wrong (i.e. the one that we generated on the cluster)

For next time:
- going forward, esp. as we set up these optimizations, we want to be careful that tarjectories on the cluster correspond with trajectores in colabs/when we visualize them. So, we propose doing the following:
  - go on the colab. Crank up interaction parameters between the spider and the icosahedron. If nothing really happens, that could explain the 0 gradients
  - once we get some interactions, do the same thing on the cluster, pickle th etrajcetory, upload it to colab, and visualize it
  - total pain, but has to be done

## Feb 2, 2023

dear diary, today we tested the gradients some more. we found out that it makes sense for some gradients to be zero depending on the initial parameters. for example, say the legs and head of the spider aren't touchin anything  --  then ofcourse the grad w.r.t. their diameters will be 0. 

given this, we don't understand why we got 0s for some of the optmization yesterday. to debug this, we propose the following steps:
- continue the test in __main__ in `simulation.py` to take the grad of th esimulatoin rather than of a single energy evaluatoin. use the parameters we used for ht energy evaluation. if this is fine (i.e. all grads that we expect to be non-zero are non-zero), gthen redo optimization. If optimization doesn't work, probably juts soething wrong there. 

Once we are ready (ie.e. aoptimization grads work), should fiddle around in the colab to find a reasonable set of starting parameters.

## Feb 3, 2032

We got some stuff working today. First, we confirmed that gradients (w.r.t. the energy function) propagate through the optimization. However, we realized that the grads w.r.t. the *loss* function are 0 -- we suspect this is due to the loss function being in a local minimum. One cause for beign ina  local minimum could be that the nearest local minimum is this the "explosion", for which the loss is similar/identical (maybe) as in thefully bound state

Some things to address are (i) constraints on the spider/UFO/unidenfied shape, and (ii) constraints on the loss function. These include the following:
- the leg/bond of the spider should have excluded volume. This would prevent the explosion thing because the leg couldn't overlap with the rest of the shell
- we should include additional constraints in the loss function to descibe waht we want. te most immediate is that the rest of the shell should stay together (e.g. the sum of the pairwise distances for the remainiing 11 things). this will also miitigate the explosoin thing, potentially in a better way
constraints of this form could also smooth the landscape

### Part 2

An interesting night -- we found that we had hardcoded all of the values in run_dynamics, but not in run_dynamics_helper. This explained our 0.0 gradients.

We ran our working parameter set in the colab with several keys and initial separation coefficients to midlly evaluate its robustness.

We just launched a bunch of jobs.

Next time, we want to:
- look at what the loss function is telling us about what we're asking for
  - change the loss function accordingly
- consider making the legs repulsive/having some excluded volume


papers to read for next week: diffussion model paper from Baker lab, ICLR paper


## Feb 5, 2032

we want to see random parmeters converge

we have been running some with fixed parameters to see if it could solv eht eproblem of recombining. it oculdn't, so far.

if this doesn't work, a couple things we could try:
- random parameters w/o the extra loss term. confirm that we can converge there
- try differetn optimizers (note: make a flag for this)