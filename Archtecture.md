# Archtecture


Raw Data → Tokenization → Quantum Encoding → Qubit Graph
                                  ↓
                           Compression + Removal
                                  ↓
                   Classical Latent Diffusion Decoder
                                  ↓
                         Quantum Decoder Layer
                                  ↓
                        Error Circuit Estimation
                                  ↓
                        RNN Controls Error Removal
                                  ↓
                          Loop for Refinement




# 1. **Quantum Encoding**

Starting with classical data $x$, you encode it into a quantum state $|\psi_{\text{encoded}}\rangle$:

$$
|\psi_{\text{encoded}}\rangle = U_{\text{encode}}(x) |0\rangle^{\otimes n}
$$

* $U_{\text{encode}}(x)$ is your parameterized encoding circuit.
* $n$ = number of qubits.

---

# 2. **Compression and Subcircuit Removal**

You represent the quantum state as a graph $G = (V, E)$ with qubits as vertices $V$ and entanglements as edges $E$.

Let $S \subseteq V$ be the subcircuit (subset of qubits) to remove.

Removing $S$ results in the compressed graph:

$$
G' = G \setminus S
$$

Where the subcircuit removal is controlled by the RNN, producing a mask $m \in \{0,1\}^n$:

$$
m_i = \begin{cases}
0, & \text{if qubit } i \in S \\
1, & \text{otherwise}
\end{cases}
$$

---

# 3. **Classical Latent Diffusion Decoder**

From the compressed graph $G'$, you extract a latent vector $z$ (e.g., via amplitudes or classical embedding):

$$
z = \text{extract\_latent}(G')
$$

The latent diffusion model $D_{\theta}$ decodes $z$ to produce a classical reconstruction $\hat{x}_c$:

$$
\hat{x}_c = D_{\theta}(z)
$$

---

# 4. **Quantum Decoder Layer**

The quantum decoder applies a variational circuit $U_{\phi}$ on the encoded quantum state to produce the decoded state $|\psi_{\text{decoded}}\rangle$:

$$
|\psi_{\text{decoded}}\rangle = U_{\phi} |\psi_{\text{encoded}}\rangle
$$

Alternatively, if the quantum decoder takes classical latent $z$ and quantum state $|\psi_{\text{encoded}}\rangle$:

$$
|\psi_{\text{decoded}}\rangle = U_{\phi}(z) |\psi_{\text{encoded}}\rangle
$$

---

# 5. **Fidelity and Error Measurement**

Fidelity $F$ between decoded quantum state and ground truth $|\psi_{\text{true}}\rangle$ measures error:

$$
F = |\langle \psi_{\text{true}} | \psi_{\text{decoded}} \rangle|^2
$$

Define **error** $\mathcal{E}$ as:

$$
\mathcal{E} = 1 - F
$$

---

# 6. **RNN Error Controller**

The RNN takes features derived from quantum decoder output and latent diffusion (call it $f$) and outputs a mask $m$ for subcircuit removal:

$$
m = \text{RNN}(f)
$$

Where $m \in \{0,1\}^n$ as before.

---

# 7. **Training Objective**

The total loss $\mathcal{L}$ balances classical reconstruction and quantum fidelity, plus regularization on error mask size:

$$
\mathcal{L} = \lambda_c \cdot \mathcal{L}_{\text{classical}}(\hat{x}_c, x) + \lambda_q \cdot (1 - F) + \lambda_m \cdot \|m\|_1
$$

* $\mathcal{L}_{\text{classical}}$ is classical reconstruction loss (e.g., MSE).
* $\lambda_c, \lambda_q, \lambda_m$ are weighting hyperparameters.
* $\|m\|_1$ encourages sparse pruning (small number of qubits removed).

---

# Summary Diagram of Flow:

$$
x \xrightarrow{U_{\text{encode}}} |\psi_{\text{encoded}}\rangle \xrightarrow{\text{RNN}(f) \to m} G' \xrightarrow{\text{extract\_latent}} z \xrightarrow{D_{\theta}} \hat{x}_c
$$

$$
|\psi_{\text{encoded}}\rangle \xrightarrow{U_{\phi}(z)} |\psi_{\text{decoded}}\rangle \xrightarrow{\text{compare}} F, \mathcal{E}
$$
