<!-- SHIELDS -->
<div align="left">

  [![Release](https://img.shields.io/pypi/v/pauli-propagation.svg?label=Release)](https://github.com/Qiskit/pauli-propagation/releases)
  ![Platform](https://img.shields.io/badge/%F0%9F%92%BB_Platform-Linux%20%7C%20macOS-blue)
  [![Python](https://img.shields.io/pypi/pyversions/pauli-propagation?label=Python&logo=python)](https://www.python.org/)
  [![Qiskit](https://img.shields.io/badge/Qiskit%20-%20%3E%3D1.2%20-%20%236133BD?logo=Qiskit)](https://github.com/Qiskit/qiskit)
<br />
  [![Docs (stable)](https://img.shields.io/badge/%F0%9F%93%84%20Docs-stable-blue.svg)](https://qiskit.github.io/pauli-propagation/)
  <!-- [![DOI](https://zenodo.org/badge/DOI/TODO](https://zenodo.org/doi/TODO -->
  [![License](https://img.shields.io/github/license/Qiskit/pauli-propagation?label=License)](LICENSE.txt)
  [![Tests](https://github.com/Qiskit/pauli-propagation/actions/workflows/test_latest_versions.yml/badge.svg)](https://github.com/Qiskit/pauli-propagation/actions/workflows/test_latest_versions.yml)

# Pauli propagation

### Table of contents

* [About](#about)
* [Documentation](#documentation)
* [Installation](#installation)
* [Computational requirements](#computational-requirements)
* [Deprecation Policy](#deprecation-policy)
* [Contributing](#contributing)
* [License](#license)
* [References](#references)

----------------------------------------------------------------------------------------------------

### About

Pauli propagation, also known as sparse Pauli dynamics (SPD), is a framework for approximating the
evolution of operators in the Pauli basis under the action of other operators, such as quantum
circuit gates and noise channels [1] - [4]. This technique has most commonly been used to classically estimate
expectation values of quantum systems, but it has also been used for reducing the depth of quantum
circuits to be run on a quantum processor [5]. Check out the [tutorial](https://github.com/Qiskit/pauli-propagation/blob/main/docs/tutorials/do_something.ipynb) to learn how to use
this package to classically simulate expectation values of quantum systems.

This package provides a Rust-accelerated Python interface for performing the most common Pauli
propagation routines. Namely:

- ``propagate_through_rotation_gates``: Evolve a Pauli operator, $O$, through a sequence of Pauli rotation
    gates, $P$, creating a transformed operator, $\tilde{O}$. This evolution can be done in either
    the Heisenberg frame ($\tilde{O} = P^{\dagger}OP$) or the Schrödinger frame ($\tilde{O} = POP^{\dagger}$).
- ``propagate_through_operator``: Evolve a Pauli operator, $O$, through a non-unitary Pauli-sum operator, $G$,
    creating a transformed operator, $\tilde{O}$. This evolution can be done in either the
    Heisenberg frame ($\tilde{O} = G^{\dagger}OG$) or the Schrödinger frame ($\tilde{O} = GOG^{\dagger}$).
- ``evolve_through_cliffords``: Separate a quantum circuit, $U$, into its Clifford and non-Clifford
    parts, $C$ and $P$ respectively, such that $U = PC$.

Some features and technical details of the package include:

- Rust-accelerated Python interface
- Ability to truncate terms from $\tilde{O}$ during evolution based on an absolute coefficient
    tolerance, a fixed number of terms in the evolving operator, or a combination of both.
- Ability to perform Pauli propagation in both the Schrödinger and Heisenberg frames.
- Novel technique for approximating the conjugation of two non-unitary Pauli-sum operators, $G O G^{\dagger}$.
    This would normally require calculating a cubic number of Pauli terms, but this implementation
    generates only the terms in the product with the largest coefficients.
- The Rust acceleration module uses bit-packing to reduce the memory requirements and runtime of
    ``evolve_by_circuit``; however, a ``qiskit.quantum_info.SparsePauliOp`` is still instantiated to
    describe the final $\tilde{O}$. If instantiating this Qiskit object is prohibitive, we could
    provide the bit-packed data to the user directly. Please let us know if something like this
    would be useful in your workflows!


----------------------------------------------------------------------------------------------------

### Documentation

All documentation is available at https://qiskit.github.io/pauli-propagation/.

----------------------------------------------------------------------------------------------------

### Installation

We encourage installing this package via `pip`, when possible:

```bash
pip install 'pauli-propagation'
```

For more installation information refer to these [installation instructions](docs/install.rst).

----------------------------------------------------------------------------------------------------

### Computational requirements

Both the memory and time cost for Pauli propagation routines generally scale with the size to which
the evolved operator is allowed to grow.

``propagate_through_rotation_gates``: As the Pauli operator, $\tilde{O}$, is propagated under the
action of a sequence of $N$ Pauli rotation gates, it will grow as $\mathcal{O}(2^{N})$. To control
the memory usage, the operator is truncated after application of each gate, which introduces some
error proportional to the magnitudes of the truncated terms' coefficients. The memory requirements
are generally linear in the size of the evolved operator and runtime scales linearly in both the
operator size and the number of gates.

``propagate_through_operator``: To conjugate an operator in the Pauli basis by another such operator,
($G^{\dagger}OG$) one must generate a cubic number of terms, one term for each combination of Pauli
terms in the product ($G^{\dagger}[i], O[j], G[k]$). This implementation sorts the coefficients in
$G^{\dagger}$, $O$, and $G$ in descending order and performs a search for the terms with the largest
coefficients over the 3D index space, starting with the origin, $(0, 0, 0)$, which is guaranteed
to result in the most significant contribution to the product. In our benchmarks, the runtime is
primarily used to traverse the 3D index space to find the index triplets representing the most
significant terms in the product; however, a non-negligible amount of time is also spent sorting
the operators and performing Pauli multiplication to generate the terms in the new operator.

``evolve_through_cliffords``: This function heavily leverages the Clifford evolution subroutines
from Qiskit. While this is reasonably fast, it may be unnecessarily slow for users wishing to call
it in a tight loop. There is ongoing work in Qiskit to speed up some of these routines which may
be leveraged by this package in the future. Please let us know if this function is a bottleneck in
your workflows.

----------------------------------------------------------------------------------------------------

### Deprecation Policy

We follow [semantic versioning](https://semver.org/) and are guided by the principles in
[Qiskit's deprecation policy](https://github.com/Qiskit/qiskit/blob/main/DEPRECATION.md).
We may occasionally make breaking changes in order to improve the user experience.
When possible, we will keep old interfaces and mark them as deprecated, as long as they can co-exist with the
new ones.
Each substantial improvement, breaking change, or deprecation will be documented in the
[release notes](https://qiskit.github.io/pauli-propagation/release-notes.html).

----------------------------------------------------------------------------------------------------

### Contributing

The source code is available [on GitHub](https://github.com/Qiskit/pauli-propagation).

The developer guide is located at [CONTRIBUTING.md](https://github.com/Qiskit/pauli-propagation/blob/main/CONTRIBUTING.md)
in the root of this project's repository.
By participating, you are expected to uphold Qiskit's [code of conduct](https://github.com/Qiskit/qiskit/blob/main/CODE_OF_CONDUCT.md).

----------------------------------------------------------------------------------------------------

### License

[Apache License 2.0](LICENSE.txt)

----------------------------------------------------------------------------------------------------

### References

[1] Tomislav Begušić, Johnnie Gray, Garnet Kin-Lic Chan, [Chemistry Beyond Exact Solutions on a Quantum-Centric Supercomputer](https://arxiv.org/abs/2308.05077), arXiv:2308.05077 [quant-ph].

[2] Manuel S. Rudolph, et al., [Pauli Propagation: A Computational Framework for Simulating Quantum Systems](https://arxiv.org/abs/2505.21606), arXiv:2505.21606 [quant-ph].

[3] Hrant Gharibyan, et al., [A Practical Guide to using Pauli Path Simulators for Utility-Scale Quantum Experiments](https://arxiv.org/abs/2507.10771), arXiv:2507.10771 [quant-ph].

[4] Lukas Broers, et al., [Scalable Simulation of Quantum Many-Body Dynamics with Or-Represented Quantum Algebra](https://arxiv.org/abs/2506.13241), arXiv:2506.13241 [quant-ph].

[5] Bryce Fuller, et al., [Improved Quantum Computation using Operator Backpropagation](https://arxiv.org/abs/2502.01897), arXiv:2502.01897 [quant-ph].
