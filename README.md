# rs_fec_conv

[![PyPi](https://img.shields.io/pypi/v/rs_fec_conv.svg?style=flat-square)](https://pypi.org/project/rs_fec_conv/)
[![Docs](https://readthedocs.org/projects/rs_fec_conv/badge/?version=latest)](http://rs_fec_conv.readthedocs.io/en/latest/?badge=latest)

## Getting Started
The package rs_fec_conv is a rust binding built with [pyo3](https://github.com/PyO3/pyo3).
rs_fec_conv is intended to be used in parallel with the 
[scikit-dsp-comm](https://github.com/mwickert/scikit-dsp-comm) package.
The rust binding improve the processing time of the conv_encoder and viterbi_decoder algorithms.

### Rust Install
Rust is not needed on the system to execute the binaries since the functions are already pre-compiled.
Although, [Rust](https://www.rust-lang.org/tools/install) can be downloaded online or 
installed on Windows Subsystem for Linux.

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```
### Package Requirements
This package requires Python 3.7.x.

### rs_fec_conv Install
You can download the package rs_fec_conv from PyPi [PyPi](https://pypi.org/project/rs_fec_conv/),
or by the command
```bash
pip install rs_fec_conv
``` 

Note: The preferred method is to download from PyPi. 
If downloading directly from GitHub you will need to install Rust prior.

### Results
| BEP Simulation (EbN0=4,100000 bits)  G, depth | Python Time (sec) | Rust Time (sec) | Rust Speed Factor Increase |
| --------------------------------------------- | ----------------- | --------------- | -------------------------- |
| ('111', '101'), 10            | 39.88       | 0.79      | 50.24      |
| ('11111','11011','10101'), 25 | 675.00      | 21.32     | 31.66      |
| ('1111001','1011011'), 25     | 217.02      | 9.27      | 23.41      |


## Tutorial

### Convolutional Encoder
The function conv_encoder_rs can be implemented

```bash
import numpy as np
import matplotlib.pyplot as plt
from sk_dsp_comm import fec_conv

# Generate random data
N = 20
x = np.random.randint(0, 2, N)

# Initialize fec_conv object with either G length 2 or 3
G = ('111', '101')
# G = ('11110111','11011001','10010101')
cc1 = fec_conv.fec_conv(G, 10)
state = '00'

# Convolutionally Encode Signal
y, state = cc1.conv_encoder(x, state)

# Plot input signal
plt.subplot(211)
plt.stem(x)
plt.xlabel('Number of Samples')
plt.ylabel('x')
plt.title('Input Signal')

# Plot convolutionally encoded signal
plt.subplot(212)
plt.stem(y)
plt.xlabel('Number of Samples')
plt.ylabel('y')
plt.title('Convolutionally Encoded Signal')
plt.tight_layout()
plt.savefig('conv_enc.png')
```

![Convolutionally Encoded Signal](https://github.com/grayfox57/rs_fec_conv/blob/master/conv_enc.png)

### Viterbi Decoder
The function viterbi_decoder_rs can be implemented by
```bash
import numpy as np
import matplotlib.pyplot as plt
from sk_dsp_comm import fec_conv

# Generate random data
N = 20
x = np.random.randint(0, 2, N)

# Initialize fec_conv object with either G length 2 or 3
G = ('111', '101')
# G = ('11110111','11011001','10010101')
cc1 = fec_conv.fec_conv(G, 10)
state = '00'

y, state = cc1.conv_encoder(x, state)

# Viterbi decode
z = cc1.viterbi_decoder(y.astype(int), 'hard', 3)

# Plot input signal
plt.subplot(211)
plt.stem(x[:11])
plt.xlabel('Number of Samples')
plt.ylabel('x')
plt.title('Input Signal')
plt.xlim([0, 10])

# Plot viterbi decoded signal
plt.subplot(212)
plt.stem(z)
plt.xlabel('Number of Samples')
plt.ylabel('z')
plt.title('Viterbi decoded Signal')
plt.xlim([0, 10])
plt.tight_layout()
plt.savefig('viterbi_dec.png')
```

![Viterbi Decoded Signal](https://github.com/grayfox57/rs_fec_conv/blob/master/viterbi_dec.png)

Since there is no channel noise added to the signal the Viterbi decoded signal results
in no bit errors from the original signal.   

### Channel Simulation
A simulation using AWGN can be done using by integrating with other functions provided 
in the scikit-dsp-comm toolbox
```bash
# Soft decision rate 1/2 simulation
N_bits_per_frame = 100000
EbN0 = 4
total_bit_errors = 0
total_bit_count = 0
cc1 = rs_fec.fec_conv(('11101','10011'),25)

# Encode with shift register starting state of '0000'
state = '0000'
while total_bit_errors < 100:
	# Create 100000 random 0/1 bits
	x = randint(0,2,N_bits_per_frame)
	y,state = cc1.conv_encoder(x,state)

	# Add channel noise to bits, include antipodal level shift to [-1,1]
	# Channel SNR is 3 dB less for rate 1/2
	yn_soft = dc.cpx_AWGN(2*y-1,EbN0-3,1) 
	yn_hard = ((np.sign(yn_soft.real)+1)/2).astype(int)
	z = cc1.viterbi_decoder(yn_hard,'hard')

	# Count bit errors
	bit_count, bit_errors = dc.bit_errors(x,z)
	total_bit_errors += bit_errors
	total_bit_count += bit_count
	print('Bits Received = %d, Bit errors = %d, BEP = %1.2e' %\
		  (total_bit_count, total_bit_errors,\
		   total_bit_errors/total_bit_count))

print('*****************************************************')
print('Bits Received = %d, Bit errors = %d, BEP = %1.2e' %\
	  (total_bit_count, total_bit_errors,\
	   total_bit_errors/total_bit_count))
```
   
Rate 1/2 Object

kmax =  0, taumax = 0

Bits Received = 99976, Bit errors = 845, BEP = 8.45e-03
