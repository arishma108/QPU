```
!pip install --upgrade cudaq
```

```
import cudaq
from cudaq import operators, spin, Schedule, ScipyZvodeIntegrator
from cudaq.operator import coherent_state
import numpy as np
import cupy as cp
 
cudaq.set_target("dynamics")
```

```
# Number of cavity photons
N = 20
 
# System dimensions: transmon + cavity
dimensions = {0: 2, 1: N}
 
# System parameters
# Unit: GHz
omega_01 = 3.0 * 2 * np.pi  # transmon qubit frequency
omega_r = 2.0 * 2 * np.pi   # resonator frequency
 
# Dispersive shift
chi_01 = 0.025 * 2 * np.pi
chi_12 = 0.0
```

```
# Alias for commonly used operators
# Cavity operators
a = operators.annihilate(1)
a_dag = operators.create(1)
nc = operators.number(1)
xc = operators.annihilate(1) + operators.create(1)
 
# Transmon operators
sz = spin.z(0)
sx = spin.x(0)
nq = operators.number(0)
xq = operators.annihilate(0) + operators.create(0)
```

```
omega_01_prime = omega_01 + chi_01
omega_r_prime = omega_r - chi_12 / 2.0
chi = chi_01 - chi_12 / 2.0
hamiltonian = 0.5 * omega_01_prime * sz + (omega_r_prime + chi * sz) * a_dag * a

```

```
# Transmon in a superposition state
transmon_state = cp.array([1. / np.sqrt(2.), 1. / np.sqrt(2.)],
                          dtype=cp.complex128)
 
# Cavity in a superposition state
cavity_state = coherent_state(N, 2.0)
psi0 = cudaq.State.from_data(cp.kron(transmon_state, cavity_state))
```

```
steps = np.linspace(0, 250, 1000)
schedule = Schedule(steps, ["time"])

```

```
evolution_result = cudaq.evolve(hamiltonian,
                          dimensions,
                          schedule,
                          psi0,
                          observables=[nc, nq, xc, xq],
                          collapse_operators=[],
                          store_intermediate_results=True,
                          integrator=ScipyZvodeIntegrator())

```

```
get_result = lambda idx, res: [
    exp_vals[idx].expectation() for exp_vals in res.expectation_values()
]
count_results = [
    get_result(0, evolution_result),
    get_result(1, evolution_result)
]
 
quadrature_results = [
    get_result(2, evolution_result),
    get_result(3, evolution_result)
]
```
