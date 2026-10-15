
# Quantum-Monte-Carlo-Amplitude-Estimation

A Qiskit-based implementation of Quantum Amplitude Estimation (QAE), benchmarking estimation error and runtime against classical Monte Carlo across sample sizes up to $10^9$.

## Overview

Monte Carlo methods are widely used to estimate probabilities and expected values in computational science, finance, statistical modelling, and physics. Their main limitation is statistical: reducing the typical estimation error requires a substantial increase in the number of samples.

Quantum Amplitude Estimation (QAE) offers a different approach. By encoding a probability into a quantum state's amplitude and exploiting quantum interference and amplitude amplification, QAE can achieve a **quadratic improvement in query complexity** over classical Monte Carlo estimation under the appropriate algorithmic assumptions.

This project implements a simple probability-estimation problem in Qiskit and compares it with a classical Monte Carlo baseline. It includes:

- Classical Monte Carlo probability estimation using NumPy.
- Quantum state preparation to encode a known success probability.
- Iterative Quantum Amplitude Estimation using Qiskit Algorithms.
- Classical-versus-quantum comparisons of absolute estimation error and execution time.
- A logarithmic sample-size sweep extending from $10^2$ to $10^9$.
- Visualizations illustrating the accuracy-runtime trade-off and the behaviour of both methods across different sample-size ranges.

The notebook is intended as an educational and experimental demonstration of quantum amplitude estimation, its underlying principles, and its potential computational advantages.

## Project Contents

| File | Description |
|---|---|
| `Monte_Carlo_vs_QAE.ipynb` | Main notebook containing the classical estimator, Qiskit implementation, numerical comparisons, and plots. |
| `README.md` | Project documentation, theoretical background, implementation details, and usage instructions. |
| `LICENSE` | MIT License. |

## Theoretical Background

### 1. Classical Monte Carlo Estimation

Consider a random variable $X$ with a Bernoulli distribution:

$$
X =
\begin{cases}
1, & \text{with probability } a, \\
0, & \text{with probability } 1-a.
\end{cases}
$$

The objective is to estimate the unknown success probability $a$ from repeated observations.

For $N$ independent samples, the classical Monte Carlo estimator is

$$
\hat{a}_{\mathrm{MC}} =
\frac{1}{N}\sum_{i=1}^{N} X_i.
$$

This estimator is unbiased, and its variance is

$$
\operatorname{Var}(\hat{a}_{\mathrm{MC}})
= \frac{a(1-a)}{N}.
$$

Consequently, its standard error scales as

$$
\boxed{\epsilon_{\mathrm{MC}} = O(N^{-1/2})}
$$

where $N$ is the number of independent samples.

This scaling has an important consequence: reducing the typical statistical error by a factor of ten generally requires approximately one hundred times as many samples.

### 2. Quantum Amplitude Encoding

Quantum Amplitude Estimation encodes the probability of interest into the amplitudes of a quantum state.

For the probability $a$, the state can be written as

$$
|\psi\rangle =
\sqrt{1-a}\,|0\rangle +
\sqrt{a}\,|1\rangle.
$$

The probability of measuring the success state $|1\rangle$ is therefore

$$
|\langle 1|\psi\rangle|^2 = a.
$$

In this notebook, the true success probability is set to

$$
a = 0.2.
$$

The prepared state is

$$
|\psi\rangle =
\sqrt{0.8}\,|0\rangle +
\sqrt{0.2}\,|1\rangle.
$$

The implementation constructs this state using Qiskit's `StatePreparation` circuit.

```python
true_a = 0.2
theta = 2 * np.arcsin(np.sqrt(true_a))

prep = StatePreparation([
    np.cos(theta / 2),
    np.sin(theta / 2)
])
```

The resulting state preparation operation is supplied to Qiskit's amplitude-estimation framework.

### 3. Amplitude Amplification and Quantum Estimation

Amplitude amplification uses quantum reflections to increase or otherwise transform the amplitude associated with a designated success subspace.

Writing

$$
a = \sin^2\theta,
$$

the success probability after $k$ Grover-style amplification iterations takes the form

$$
P_k = \sin^2\big((2k+1)\theta\big).
$$

The resulting dependence on the iteration count provides information about the underlying amplitude. Quantum Amplitude Estimation uses this information to estimate $a$.

Unlike ordinary sampling, QAE does not rely solely on counting independent outcomes from the original state. It exploits the structure of quantum operations to extract information about the amplitude.

### 4. Classical Versus Quantum Query Complexity

For a target additive estimation error $\epsilon$, the standard theoretical comparison is:

| Method | Query complexity |
|---|---|
| Classical Monte Carlo | $O(1/\epsilon^2)$ |
| Quantum Amplitude Estimation | $O(1/\epsilon)$ |

Under the relevant assumptions, QAE provides a **quadratic improvement in query complexity**.

Equivalently, the idealized error scaling with the number of relevant queries $N$ is $O(1/N)$ for QAE, compared with $O(1/\sqrt{N})$ for classical Monte Carlo.

These are theoretical complexity results, not guarantees of a quadratic improvement in wall-clock execution time for every implementation. The cost of state preparation, oracle operations, circuit execution, hardware noise, and post-processing must also be considered.

## Implementation

The notebook uses Python, NumPy, Matplotlib, and Qiskit to implement the estimation problem and evaluate both approaches.

### Classical Monte Carlo

The classical baseline generates uniformly distributed random numbers and estimates the success probability by counting the fraction below the true probability.

```python
def classical_monte_carlo(true_prob, n_samples):
    samples = np.random.rand(n_samples)
    est = np.mean(samples < true_prob)
    return est
```

For each sample size, the notebook records:

- Estimated success probability.
- Absolute estimation error.
- Execution time.

The absolute error is defined as

$$
E_{\mathrm{MC}} =
|\hat{a}_{\mathrm{MC}} - a|.
$$

### Quantum Amplitude Estimation

The quantum implementation uses the following Qiskit components:

- `StatePreparation` to prepare the amplitude-encoded state.
- `EstimationProblem` to define the state preparation and success qubit.
- `IterativeAmplitudeEstimation` to estimate the success amplitude.
- `Sampler` to execute the estimation experiments.

The main estimation configuration is:

```python
qae = IterativeAmplitudeEstimation(
    epsilon_target=0.001,
    alpha=0.05,
    sampler=sampler
)

qae_result = qae.estimate(problem)
```

The target precision is set to `0.001`, and `alpha=0.05` specifies the algorithm's confidence parameter.

The notebook records the processed quantum estimate, its absolute error relative to the known probability, and the measured execution time.

## Benchmarking Methodology

### Experiment 1: Accuracy Versus Runtime

The first benchmark compares the classical Monte Carlo estimator at several sample sizes with a QAE estimate of the same probability.

The resulting plot uses logarithmic axes for runtime and absolute estimation error.

It illustrates the trade-off between computational effort and estimation accuracy. Increasing the classical sample count generally improves statistical precision, while QAE uses amplitude-estimation procedures to estimate the probability.

### Experiment 2: Classical and Quantum Error Comparison

The notebook prints comparison tables containing the estimated probability, absolute error, and runtime for both methods.

Because the true probability is known, the estimation error can be calculated directly:

$$
E = |\hat{a} - a|.
$$

This makes the example useful for evaluating numerical accuracy without requiring an external reference value.

### Experiment 3: Scaling the Classical Sample Size

The notebook performs an additional sweep over different upper exponents:

```python
for end_exponent in range(2, 10):
    sample_sizes = np.logspace(
        2, end_exponent, num=7, dtype=int
    )
```

The sweep covers upper exponents from 2 through 9, extending the classical sample-size range from $10^2$ to $10^9$.

| Upper exponent | Sample-size interval |
|---:|---|
| 2 | $10^2$ to $10^2$ |
| 3 | $10^2$ to $10^3$ |
| 4 | $10^2$ to $10^4$ |
| 5 | $10^2$ to $10^5$ |
| 6 | $10^2$ to $10^6$ |
| 7 | $10^2$ to $10^7$ |
| 8 | $10^2$ to $10^8$ |
| 9 | $10^2$ to $10^9$ |

For each upper exponent, the notebook evaluates classical Monte Carlo at seven logarithmically spaced sample sizes. It also reruns QAE for the same prepared state and records the resulting runtime and absolute error.

The final visualization overlays the classical and quantum measurements for all upper exponents, using different colours to distinguish the sweeps.

**Important interpretation:** The classical workload changes with the upper exponent, but the QAE problem and target precision remain fixed. The final graph therefore illustrates the effect of increasing the classical sample-size range alongside repeated QAE runs; it is not, by itself, a measurement of QAE complexity as a function of an increasing quantum problem size.

For the exponent-2 sweep, all seven sample sizes are identical because both endpoints equal $10^2$.

## Observed Results

In the notebook's saved initial benchmark run, the classical Monte Carlo estimator uses $10^7$ samples, while the QAE routine estimates the same prepared probability.

| Metric | Classical Monte Carlo | Quantum Amplitude Estimation |
|---|---:|---:|
| True probability | 0.2 | 0.2 |
| Estimated probability | 0.1998149 | 0.2000000 |
| Absolute error | $1.851\times10^{-4}$ | Approximately $8.3\times10^{-17}$ |
| Recorded runtime | Approximately 0.121 s | Approximately 0.0118 s |

These are values from the notebook's saved execution, not guaranteed results for every run or machine.

The quantum estimate is extremely close to the prepared probability in this execution. However, the near-machine-precision error should not be interpreted as the general accuracy of QAE. It reflects this particular idealized, simulator-based experiment and configuration.

Similarly, the recorded runtime difference is an observation from one local execution. It is not a universal speedup factor and does not establish that QAE will outperform classical Monte Carlo in wall-clock time for arbitrary estimation tasks.

The classical baseline is stochastic, so its estimate, absolute error, and runtime may change between runs. The notebook fixes the Qiskit algorithm seed, but does not currently fix NumPy's random seed.

## Getting Started

### Option 1: Run in Google Colab

1. Open the notebook in the repository.
2. Select **Open in Colab**, or use the following link after replacing the repository details:

   [Open in Google Colab](https://colab.research.google.com/github/USERNAME/REPOSITORY/blob/main/Monte_Carlo_vs_QAE.ipynb)

3. Execute the notebook cells in order.
4. Review the printed benchmark tables and the generated visualizations.

### Option 2: Run Locally

Clone the repository:

```bash
git clone https://github.com/USERNAME/REPOSITORY.git
cd REPOSITORY
```

Create and activate a virtual environment:

```bash
python -m venv .venv
```

On Windows (PowerShell):

```powershell
.venv\Scripts\Activate.ps1
```

On Linux or macOS:

```bash
source .venv/bin/activate
```

Install the dependencies:

```bash
python -m pip install "numpy==2.0.2" "scipy>=1.14.0" matplotlib
python -m pip install "qiskit==1.2.4" "qiskit-aer==0.15.1" "qiskit-algorithms==0.3.1"
python -m pip install jupyter
```

Launch Jupyter:

```bash
jupyter notebook
```

Open `Monte_Carlo_vs_QAE.ipynb` and run the cells sequentially.

The notebook's recorded environment uses Python 3.12 and the package versions listed above. Compatibility may vary with other Python and Qiskit versions.

## Limitations and Reproducibility

This project is an educational benchmark of classical Monte Carlo and quantum amplitude estimation. The following limitations are important when interpreting the results:

- **Idealized probability model:** The success probability is directly encoded into a prepared quantum state. The example does not implement a complex application-specific probability oracle.
- **Simulator-based execution:** The recorded QAE measurements do not demonstrate performance on a physical quantum processor.
- **Query complexity versus runtime:** Theoretical query-complexity advantages do not automatically translate into wall-clock speedups.
- **Numerical precision:** A very small absolute error in one execution should not be generalized to arbitrary probabilities or problem instances.
- **Stochastic classical results:** Classical estimates vary across runs unless the NumPy random generator is seeded.
- **Repeated quantum estimation:** The exponent sweep reruns the same QAE problem at fixed target precision. It does not increase the quantum problem size with the classical sample count.
- **Environment setup:** The original notebook contains package-uninstallation and installation cells and uses Qiskit's legacy `Sampler` interface. These cells may need adjustment in a fresh environment or with newer Qiskit releases.
- **Benchmark variability:** Execution time depends on the environment, software versions, machine load, and simulator configuration.

The notebook is best understood as a demonstration of the amplitude-estimation principle and its theoretical scaling advantage, supported by a numerical comparison on a simple probability-estimation task.

## Future Improvements

Possible extensions include:

- Seeding NumPy to make classical comparisons reproducible.
- Replacing destructive package-installation cells with a separate dependency file.
- Updating the sampler implementation to a supported Qiskit interface.
- Repeating classical and quantum experiments to report mean errors, variability, and confidence intervals.
- Comparing estimation error against actual query or oracle-call counts, in addition to wall-clock runtime.
- Extending the model to application-specific probability estimation and expected-value estimation.
- Evaluating circuit execution under realistic noise models and, where appropriate, on quantum hardware.

## References

1. Brassard, G., Høyer, P., Mosca, M., and Tapp, A. *Quantum Amplitude Amplification and Estimation*. Contemporary Mathematics, 2002. [arXiv:quant-ph/0005055](https://arxiv.org/abs/quant-ph/0005055).
2. Grinko, D., Gacon, J., Zoufal, C., and Woerner, S. *Iterative Quantum Amplitude Estimation*. npj Quantum Information, 7, 52 (2021). [https://doi.org/10.1038/s41534-021-00379-1](https://doi.org/10.1038/s41534-021-00379-1).
3. [Qiskit Algorithms documentation](https://qiskit-community.github.io/qiskit-algorithms/).
4. [Qiskit documentation](https://docs.quantum.ibm.com/).

## License

This project is licensed under the [MIT License](LICENSE).

Copyright (c) 2026 Om Gururaj Kanade
