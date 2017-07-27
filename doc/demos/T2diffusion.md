# Fitting the T2diffusion model

## Create the scheme
```python
import numpy as np
import math
import sys


filename_bval = 'file.bval'
filename_bvec = 'file.bvec'
filename_output = 'dwi'

bval = np.loadtxt( filename_bval )
bvec = np.loadtxt( filename_bvec )

# create StejskalTanner
smallDelta = 0.007
bigDelta = [ 0.0173 ]
gyromagneticRatioH = 267513.

file = open( filename_output+'.scheme', 'w')
file.write('VERSION: STEJSKALTANNER\n')
for x in range (0, bval.shape[0]):
file.write(str(bvec[0][x])+' '+str(bvec[1][x])+ ' '+ str(bvec[2][x])+' '+ str(math.sqrt(bval[x]/((bigDelta[0] - smallDelta/3 )*smallDelta**2 * gyromagneticRatioH**2) )) + ' ' + str(bigDelta[0]) + ' ' + str(smallDelta) + ' ' + str(TE[x]) +'\n')
file.close()
```

## Fitting the model to the data
```python
import numpy as np
import amico

amico.core.setup()

ae = amico.Evaluation(".", ".")

ae.set_config('doNormalizeSignal', False)
ae.set_config('doComputeNRMSE', True)
ae.load_data(dwi_filename = "file_DWIs.nii", scheme_filename = "dwi.scheme", b0_thr = 0)

ae.d_par  = np.array([ 2.42E-3, 2.13E-3 ])      # Parallel diffusivity [mm^2/s]
ae.ICVFs  = np.arange(0.3,0.9,0.1)              # Intra-cellular volume fraction(s) [0..1]
ae.d_ISOs = np.array([ 3.0E-3 ])                # Isotropic diffusivitie(s) [mm^2/s]
ae.T2s = np.array([ 104, 62 ])                  # T2 list [s]

ae.set_model("StickZeppelinBallDiffusivityT2")
ae.generate_kernels( regenerate = True )

ae.load_kernels()

ae.fit()

ae.save_results()
```
