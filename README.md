# muGAN

Implementation of the SHiP muon background Generative Adversarial Network (GAN). The networks are based on those in the SHiP paper [Fast simulation of muons produced at the SHiP
experiment using Generative Adversarial Networks](https://arxiv.org/abs/1909.04451).

The Z values are calibrated to these lines being present in run_simScript.py
```
if fileType == 'tree':
 # 2018 background production 
  primGen.SetTarget(ship_geo.target.z0+70.1225*u.m,0.)
 else:
  primGen.SetTarget(ship_geo.target.z0+50*u.m,0.)
  -7084.5
```


Unfortunately the mast MuonBackGenerator.cxx is struggles if -n is not specified or if -n is larger than the number of muons in the file. A buffer event is added to the end of the ROOT file generated. 
```
python $FAIRSHIP/macro/run_simScript.py --muShieldDesign 8 -f example.root --stepMuonShield --MuonBack --FastMuon -n 10 -g /eos/experiment/ship/data/magnet_geofiles/magnet_geo_to_test_oliver.root
```

## How to use

Clone:
```
git clone https://github.com/alexmarshallbristol/muGAN.git
```

*To run this code an installation of both [Keras](https://keras.io/) and [uproot](https://github.com/scikit-hep/uproot) are required.* (installed automatically with the package)

`cd muGAN` and the run `pip install -e . `. Depending on the version you are using, you might need to run `pip3`. 


First step is to import the library and initialise the muGAN class:
```
from SHiP_GAN_module import muGAN

muGAN = muGAN()
```


### Simple generation...

To generate physical distributions of muon kinemtics the following command can be used:
```
muon_kinematic_vectors = muGAN.generate(size=10000, tuned_aux=True)
```

The library also has a plotting function:
```
muGAN.plot_kinematics(data=muon_kinematic_vectors)
```

An example of the results obtained with this procedure are presented in [Generated_kinematics.png](Example_plots/Generated_kinematics.png).


### Boosted generation...

The GANs in this library have conditional style architectures. The generator network has auxiliary inputs which correspond to physical properties of muons. To generate physical muon kinematic distributions these auxiliary inputs should be single tailed normal distributions with a scale of 1. However, to boost the generated output in a certain direction, these auxilary distributions can be widened. Here is and example where we ask for muons with large P_t values:
```
size = 10000
boosted_auxiliary_distribution = muGAN.generate_aux_tuned(size) # Start with tuned auxiliary distributions
boosted_auxiliary_distribution[:,2] *= 2
boosted_muon_kinematic_vectors = muGAN.generate_custom_aux(boosted_auxiliary_distribution)
```
Then to plot the results:
```
muGAN.plot_kinematics(data=boosted_muon_kinematic_vectors, filename='Generated_kinematics_BOOSTED_PT.png')
```

An example of the boosted distributions obtained with this procedure are presented in [Generated_kinematics_BOOSTED_PT.png](Example_plots/Generated_kinematics_BOOSTED_PT.png).



### Generating a specific enhanced distribution...

Enhanced distributions can also be generated based on a seed distribution. This seed distribution must be a distribution created from training data. For this to work the discriminator of the GAN predicts the auxiliary inputs of the seed distribution. These values are then subsequently fed into the generator network to produce samples. To obtain useful distributions the number of points fed in as the seed distribution must not be too small. Here is an example where we have just reused the generated sample from above, throwing away all the points with low P_t.
```
muon_kinematic_vectors_enhanced_example = muon_kinematic_vectors[np.where(np.add(muon_kinematic_vectors[:,4]**2,muon_kinematic_vectors[:,5]**2)>1.5)]
muon_kinematic_vectors_enchanced = muGAN.generate_enhanced(size=1000, seed_vectors=muon_kinematic_vectors_enhanced_example)
```
Then to plot the results, showing off some additional features of the plotting function:
```
muGAN.plot_kinematics(data=muon_kinematic_vectors_enchanced, bins=25, log=False, filename='Generated_kinematics_ENHANCED.png', normalize_colormaps=False)
```

An example of the results obtained with this procedure are presented in [Generated_kinematics_ENHANCED.png](Example_plots/Generated_kinematics_ENHANCED.png).


