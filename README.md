# PENTACODE: Variable-Length Deterministic Stream Cipher
Pentacode is a non-standard, symmetric-key stream cipher implementation utilizing variable-width ciphertext expansion, deterministic alphabet shuffling, and pseudo-random initialization vectors (public keys) generated dynamically per execution

---

## 1. Mathematical Foundation & Mappings

The engine operates over a strict alphabet of 69 supported characters mapped to an integer coordinate space $\mathbb{Z}_{[1, 69]}$.

### 1.1 Character Vector Mapping (MAP1)
All plaintext, private key, and public key characters are mapped using the index assignments below:

| Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val |
|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|
| a    | 1   | b    | 2   | c    | 3   | d    | 4   | e    | 5   | f    | 6   | g    | 7   |
| h    | 8   | i    | 9   | j    | 10  | k    | 11  | l    | 12  | m    | 13  | n    | 14  |
| o    | 15  | p    | 16  | q    | 17  | r    | 18  | s    | 19  | t    | 20  | u    | 21  |
| v    | 22  | w    | 23  | x    | 24  | y    | 25  | z    | 26  | A    | 27  | B    | 28  |
| C    | 29  | D    | 30  | E    | 31  | F    | 32  | G    | 33  | H    | 34  | I    | 35  |

### 1.2 Ciphertext Lookup Array Shuffling (MAP2)
The output space spans an array of 205 specialized symbols representing valid sums from $3$ to $207$. 

Prior to execution, the base symbol array `MAP2_BASE` is deterministically shuffled via a Fisher-Yates variant initialized by a SHA-256 byte stream derived from the `privateKey`. This creates dynamic runtime shifts preventing static frequency analysis.

MAP2 example:

| Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val | Char | Val |
|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|------|-----|
| 3    | π   | 4    | σ   | 5    | Г   | 6    | Э   | 7    | √   | 8    | *   | 9    | "   | 10   | #   | 11   | E   | 12   | O   | 13   | L   | 14   | R   | 15   | f   | 16   | n   | 17   | У   | 18   | w   | 19   | Ю   | 20   | ɒ   | 21   | Y   | 22   | i   | 23   | ∞   | 24   | T   | 25   | ∂   | 26   | ɔ   | 27   | φ   | 28   | §   | 29   | ɛ   | 30   | ɗ   | 31   | ¿   | 32   | 6   | 33   | F   | 34   | ω   | 35   | 7   | 36   | 5   | 37   | ^   | 38   | ο   | 39   | ~   | 40   | }   | 41   | <   | 42   | ;   | 203  | Ձ   |  
| 43   | ≠   | 44   | ∑   | 45   | ɘ   | 46   | Я   | 47   | З   | 48   | z   | 49   | I   | 50   | >   | 51   | %   | 52   | ɞ   | 53   | ¢   | 54   | a   | 55   | χ   | 56   | ÷   | 57   | γ   | 58   | ɑ   | 59   | p   | 60   | b   | 61   | g   | 62   | _   | 63   | e   | 64   | ∫   | 65   | ɢ   | 66   | ø   | 67   | ɕ   | 68   | `|` | 69   | β   | 70   | И   | 71   | æ   | 72   | О   | 73   | 4   | 74   | (   | 75   | 1   | 76   | ɟ   | 77   | υ   | 78   | ®   | 79   | Й   | 80   | ×   | 81   | j   | 82   | !   | 204  | Ղ   |
| 83   | Q   | 84   | Л   | 85   | К   | 86   | ¡   | 87   | Т   | 88   | Н   | 89   | S   | 90   | ¥   | 91   | X   | 92   | c   | 93   | Ц   | 94   | α   | 95   | κ   | 96   | V   | 97   | Ж   | 98   | ψ   | 99   | x   | 100  | €   | 101  | С   | 102  | N   | 103  | ≈   | 104  | :   | 105  | ɠ   | 106  | Þ   | 107  | &   | 108  | t   | 109  | 8   | 110  | s   | 111  | @   | 112  | v   | 113  | ζ   | 114  | k   | 115  | P   | 116  | τ   | 117  | Х   | 118  | ł   | 119  | θ   | 120  | ε   | 121  | M   | 122  | ɓ   | 205  | Ճ   |
| 123  | +   | 124  | .   | 125  | ρ   | 126  | Е   | 127  | G   | 128  | Ч   | 129  | δ   | 130  | ]   | 131  | o   | 132  | ν   | 133  | Ы   | 134  | ±   | 135  | -   | 136  | А   | 137  | 0   | 138  | Ф   | 139  | m   | 140  | y   | 141  | ?   | 142  | µ   | 143  | ¬   | 144  | η   | 145  | `   | 146  | {   | 147  | q   | 148  | Ш   | 149  | H   | 150  | £   | 151  | ξ   | 152  | [   | 153  | 9   | 154  | )   | 155  | ○   | 156  | Z   | 157  | П   | 158  | J   | 159  | '   | 160  | B   | 161  | Ь   | 162  | В   | 206  | Մ   |
| 163  | U   | 164  | C   | 165  | λ   | 166  | r   | 167  | Щ   | 168  | ₽   | 169  | ™   | 170  | 3   | 171  | Б   | 172  | $   | 173  | h   | 174  | ©   | 175  | Р   | 176  | Д   | 177  | °   | 178  | 2   | 179  | l   | 180  | ∆   | 181  | /   | 182  | ɜ   | 183  | ι   | 184  | u   | 185  | μ   | 186  | ɡ   | 187  | Ա   | 188  | Բ   | 189  | Գ   | 190  | Դ   | 191  | Ե   | 192  | Զ   | 193  | Է   | 194  | Ը   | 195  | Թ   | 196  | Ժ   | 197  | Ի   | 198  | Լ   | 199  | Խ   | 200  | Ծ   | 201  | Կ   | 202  | Հ   | 207  | Յ   |

## 2. Dynamic Architecture Specifications

### 2.1 The Cryptographic Stream
To handle shuffling and variable symbol generation without leaking metadata, Pentacode creates infinite deterministic stream bytes:

$$S = \text{SHA-256}(\text{Seed}) \mathbin{\Vert} \text{SHA-256}(\text{SHA-256}(\text{Seed})) \mathbin{\Vert} \dots$$

### 2.2 Variable-Width Ciphertext Expansion
The number of ciphertext symbols representing a single plaintext character is fluid and deterministic based on the private key.

1. A unique length stream is generated by setting the seed to `privateKey + "_lengths"`.
2. For each character index $i$ in the plaintext, a target width ($W_i$) is derived from the stream byte index ($B_i$) using:

$$W_i = (B_i \pmod 2) + 1$$

This scales the character layout precisely to an output block size of either 1, 2, or 3 symbols.

---

## 3. Cryptographic Execution Flow

### 3.1 Encryption Formula
Let $P_i$ be the plaintext value, $K_i$ be the cycled private key value, and $V_i$ be the public key (IV) value at position $i$. The Base Sum is computed as:

$$\text{BaseSum}_i = P_i + K_i + V_i$$

For a given symbol block width $W_i$, the system generates sub-symbols by incrementing the scalar shift $s$ where $s \in [0, W_i - 1]$:

$$\text{subSum}_{i, s} = \text{BaseSum}_i + s$$

Each computed value maps directly to a dynamic token inside `MAP2`.

### 3.2 Walkthrough Matrix
Given the parameters:
* **Plaintext:** `TEST`
* **Private Key:** `katze`
* **Public Key:** `Ab12`

#### Block Breakdown:
* **$P$ Vector (`MAP1`):** `[46, 31, 45, 46]` 
* **$K$ Vector (`MAP1` cycled):** `[11, 1, 20, 26]` 
* **$V$ Vector (`MAP1`):** `[27, 2, 54, 55]` 

$$\text{BaseSum} = P + K + V = [84, 34, 119, 127]$$ 

| Index | Char | BaseSum | Width ($W_i$) | Sub-Sum Operations ($s$) | Ciphertext Output |
| :---: | :--: | :-----: | :-----------: | :----------------------- | :---------------- |
| **0** | `T`  | $84$    | 2             | $84+0=84, \; 84+1=85$    | `ЛК`              |
| **1** | `E`  | $34$    | 2             | $34+0=34, \; 34+1=35$    | `ω7`              |
| **2** | `S`  | $119$   | 1             | $119+0=119$              | `θ`               |
| **3** | `T`  | $127$   | 1             | $127+0=127$              | `G`               |

**Total Result Payload:** `ЛКω7θG` 

---

## 4. Decryption & Integrity Verification

To reverse the process, the client reads the deterministic length stream to safely slice structural blocks back into their primary mathematical coordinates:

1. Compute the character block widths $W_i$ sequentially via the `privateKey + "_lengths"` stream.
2. Isolate $W_i$ elements from the transmission string.
3. Parse the primary block token value $E_i$ via `MAP2`.
4. **Integrity Pass:** Ensure subsequent symbols in the sub-block match the deterministic sequence $\text{Val}_s = E_i + s$. If the sequence is broken, terminate (Data Tampered/Wrong Key).
5. Extract the plaintext character index through modular reverse shift mapping:

$$P_i = E_i - K_i - V_i$$

6. Convert $P_i$ back to an ASCII/UTF-8 equivalent via `MAP1` inverse lookup.
