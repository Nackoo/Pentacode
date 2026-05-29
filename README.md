```
    ____             __                       __   
   / __ \___  ____  / /_____ __________  ____/ /__ 
  / /_/ / _ \/ __ \/ __/ __ `/ ___/ __ \/ __  / _ \
 / ____/  __/ / / / /_/ /_/ / /__/ /_/ / /_/ /  __/
/_/    \___/_/ /_/\__/\__,_/\___/\____/\__,_/\___/ 
```    

## OVERVIEW: THE TWO CORE ALPHABETS

Pentacode uses two distinct character sets to hide your message:

* MAP1 (Input Characters): A list of 99 standard characters you type 
  (a-z, A-Z, 0-9, spaces, and punctuation).
* MAP2 (Output Symbols): A list of 297 strange international or 
  mathematical symbols (like π, σ, Г, Э, √) used to create the final scrambled text.

---

## PHASE 1: THE INITIAL SETUP (BUILDING MAP1 & MAP2)

Before any text is scrambled, Pentacode uses your Private Key to build completely 
randomized, custom translation alphabets. 

### STEP 1.1: CREATING THE CRYPTO-STREAM
The system takes your Private Key (e.g., "katze") and runs it through the 
secure SHA-256 algorithm. It chains the output hashes together to form a long, 
unpredictable, but reproducible stream of numbers.
  Private Key ---> [ SHA-256 ] ---> [ Endless Stream of Numbers ]

### STEP 1.2: SHUFFLING MAP1
1. It takes the standard 99 input characters.
2. It uses the stream of numbers to perform a "Fisher-Yates Shuffle" 
   (shuffling them like a deck of cards so their order changes completely).
3. It grabs another number from the stream to calculate an "Offset" (between 1 and 7).
4. Each character is assigned a numeric value: (New Position + Offset).

```
  Visual Example (If Offset is 3):
  Original: [ 'a', 'b', 'c', 'd', ... ]
  Shuffled: [ 'x', 'm', 'a', 'p', ... ]
  Values:     x=3,   m=4,   a=5,   p=6
```

### STEP 1.3: SHUFFLING MAP2
1. It takes the 297 output symbols.
2. It runs a completely separate Fisher-Yates Shuffle using a different part of the key stream.
3. Every shuffled symbol is assigned a static numerical value starting from 3: (Position + 3).

```
  Visual Example:
  Original Symbols: [ 'π', 'σ', 'Г', 'Э', ... ]
  Shuffled Symbols: [ 'Э', 'Г', 'π', 'σ', ... ]
  Values:             Э=3,  Г=4,  π=5,  σ=6
```

---

## PHASE 2: THE GRID CIPHER (SCRAMBLING THE ORDER)

Now, your plain text message is broken apart geometrically so it's no longer linear.
Let's use the Plain Text: "SECRET MESSAGE" and Private Key: "BACON"

### STEP 2.1: DETERMINING GRID SIZE
The length of your Private Key decides how wide the grid is. 
"BACON" has 5 letters, so our grid width (Columns) is 5.

### STEP 2.2: FILLING THE GRID
The message is written straight across the grid, row by row. Empty slots at the 
end are padded with spaces.
```
    Col:  0  1  2  3  4
  Row 0: [S][E][C][R][E]
  Row 1: [T][ ][M][E][S]
  Row 2: [S][A][G][E][ ]
  Row 3: [ ][ ][ ][ ][ ]  (Padded rows/cells)
```

### STEP 2.3: MULTIPLICATIVE ROW SHIFTING
Each row is slid to the left. The distance it slides depends on the row index 
multiplied by the Private Key length.
* Row 0: Shifts by (5 * 1) % 5 = 0 slots  -> [S][E][C][R][E]
* Row 1: Shifts by (5 * 2) % 5 = 0 slots  -> [T][ ][M][E][S]

Let's look at a generic visual representation of a row shifting left:
```
  Before Shift: [ A ][ B ][ C ][ D ][ E ]
  After Shift:  [ C ][ D ][ E ][ A ][ B ]  (Slid 2 spaces left)
```

### STEP 2.4: ROW & COLUMN SHUFFLING (PERMUTATIONS)
The system looks at the alphabetical order of the letters in your Private Key ("BACON").
Alphabetical Rank of "BACON" -> B=1, A=0, C=2, O=4, N=3  --> Order: [1, 0, 2, 4, 3]

The system completely swaps the physical columns and rows matching this exact rank sequence.
```
  Before Columns:  0  1  2  3  4
  After Shuffled:  1  0  2  4  3
```
```
  Before (Col Order):  [S][E][C][R][E]
  After (Col Swapped): [E][S][C][E][R]
```

### STEP 2.5: THE ANTI-DIAGONAL READOUT
Instead of reading your grid left-to-right, Pentacode reads the letters in a 
zig-zag diagonal path starting from the top-right corner down to the bottom-left.

```
    [1][2][4][7]
    [3][5][8]      <--- Follows the numbers 1, 2, 3, 4, 5, 6, 7... 
    [6][9]              slicing downward diagonally!
```

*Result of Phase 2:* Your message characters are completely out of order, e.g., "ESCERS..."

---

## PHASE 3: MATHEMATICAL SYMBOL ENCODING

Now that your characters are completely shuffled, the system turns them into Map 2 symbols.

### STEP 3.1: CHARACTER TO VALUE (MAP1)
The engine looks at each shuffled letter and pulls its number from the custom MAP1 we built.
```
  Letter 'E' ---> Map1 Value = 14
```

### STEP 3.2: DYNAMIC LENGTH SELECTION
The system pulls a number from a special key stream (`_lengths`). 
If the number is Even, this letter will turn into **1 Symbol**.
If the number is Odd, this letter will turn into **2 Symbols**.
This means the exact same letter will have completely different lengths throughout the message.

### STEP 3.3: THE SHIFT MATH
To find the final symbol, the system adds three things together:
```
  (MAP1 Character Value) + (Private Key Character Shift) + (Public Key Character Shift)

  Example Calculation:
  * MAP1 Value for 'E' = 14
  * Private Key Shift  = 8
  * Public Key Shift   = 5
  * Base Target Sum    = 14 + 8 + 5 = 27
```

### STEP 3.4: GENERATING THE SYMBOL
* If 1 Symbol was decided: The system looks up the symbol at value 27 in MAP2 (e.g., '√').
* If 2 Symbols were decided: It grabs value 27 AND value 28 from MAP2 (e.g., '√' and 'æ').

---

## PHASE 4: THE FINAL OUTPUT PACKET

1. The user's system generates a completely random, unique 20-character string 
   called the **Public Key** (e.g., `aB3kL9pQWz2xCvBnM1tY`).
2. This Public Key is pasted directly onto the very front of the text.
3. All the encoded Map 2 symbols are pasted immediately right behind it.
```
  [ 20-Character Public Key ] + [ Variable-Length Cryptographic Symbols ]
```
